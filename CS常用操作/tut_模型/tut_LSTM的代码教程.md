# LSTM的代码教程

本文记录 `LSTM` 的代码教程。

前置条件：
- `python` 或者 `conda` 已经安装可用
在终端按行输入：

```bash
conda --version
(window请输入)where conda
(linux请输入)which conda
```

如果有类似的输出：

```text
conda 25.5.1

D:\Software\anaconda3\condabin\conda.bat
D:\Software\anaconda3\Scripts\conda.exe
```

说明conda能正常调用，可以进入某个虚拟环境唤起python。否则请查看教程[tut_conda的下载安装](tut_conda的下载安装.md)

---

## 导入必要库

```python
import os
import sentencepiece as spm
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader

# os.environ["CUDA_VISIBLE_DEVICES"] = "2"
```

## BPE分词

先训练 `BPE` 分词模型。  
`text = "今天天气非常好。"`  
`encode_result = sp_cn.encode(text, out_type=int)`  
`>>> print("编码:", encode_result)`  
`>>> 编码: [387, 3205, 5241, 11821]`

`decode_result = sp_cn.decode(encode_result)`  
`>>> print("解码:", decode_result)`  
`>>> 解码: 今天天气非常好。`

```python
# ---------------------#
# 1. BPE Tokenization (SentencePiece)
# ---------------------#

# 先运行以下命令训练 BPE 模型（只需要运行一次）
# spm.SentencePieceTrainer.Train(
#     '--input="data\\en2cn\\train_en.txt" --model_prefix=en_bpe '
#     '--vocab_size=16000 --model_type=bpe --character_coverage=1.0 '
#     '--unk_id=0 --pad_id=1 --bos_id=2 --eos_id=3'
# )
# spm.SentencePieceTrainer.Train(
#     '--input="data\\en2cn\\train_zh.txt" --model_prefix=zh_bpe '
#     '--vocab_size=16000 --model_type=bpe --character_coverage=0.9995 '
#     '--unk_id=0 --pad_id=1 --bos_id=2 --eos_id=3'
# )

sp_en = spm.SentencePieceProcessor()
sp_en.load('en_bpe.model')

sp_cn = spm.SentencePieceProcessor()
sp_cn.load('zh_bpe.model')


def tokenize_en(text):
    return sp_en.encode(text, out_type=int)


def tokenize_cn(text):
    return sp_cn.encode(text, out_type=int)


# 中文和英文一致，取英文
PAD_ID = sp_en.pad_id()  # 1
UNK_ID = sp_en.unk_id()  # 0
BOS_ID = sp_en.bos_id()  # 2
EOS_ID = sp_en.eos_id()  # 3
```

---

## 符号说明

后文里常见的维度缩写含义如下：

- `B`: batch size，表示一个 batch 里有多少条样本
- `S`: source sequence length，源序列长度
- `T`: target sequence length，目标序列长度
- `t`: current time step，当前时间步下标
- `H`: hidden size，隐藏状态维度
- `L`: number of layers，网络层数
- `V`: vocabulary size，词表大小
- `emb_dim`: embedding dimension，词向量维度
- `src`: source sequence，源语言输入序列
- `trg`: target sequence，目标语言输出序列

## DataLoader

首先根据路径读取 txt 的 line，示例值（2 代表开头，3 代表结束）：
`src_ids = [2] + [120, 456, 789, 42] + [3]`  
`trg_ids = [2] + [234, 9] + [3]`  
`pairs = [([2, 120, 456, 789, 42, 3], [2, 234, 9, 3]), ...]`
这里一个 `pair` 存储 `([翻译前的编码], [翻译后的编码])`。

以 `B=3` 为例，代表有一个 batch 里包含 3 个翻译前后的 pair 样本：
`batch = [(torch.LongTensor([2, 10, 11, 3]), torch.LongTensor([2, 100, 101, 3])), (torch.LongTensor([2, 12, 3]), torch.LongTensor([2, 102, 103, 104, 3])), (torch.LongTensor([2, 13, 14, 15, 3]), torch.LongTensor([2, 105, 3]))]`
由于 `src` 里最长的长度是 5，`trg` 里最长的长度也是 5，因此需要分别对 `src` 和 `trg` 里非最长的样本进行 padding 补零（用 1 来补），方便后续计算。

补零后的 `src_pad` 形状为 `[S_max, B] = [5, 3]`
`src_pad = tensor([`
`    [ 2,  2,  2],`
`    [10, 12, 13],`
`    [11,  3, 14],`
`    [ 3,  1, 15],`
`    [ 1,  1,  3],`
`], dtype=torch.int64)`

补零后的 `trg_pad` 形状为 `[T_max, B] = [5, 3]`
`trg_pad = tensor([`
`    [  2,   2,   2],`
`    [100, 102, 105],`
`    [101, 103,   3],`
`    [  3, 104,   1],`
`    [  1,   3,   1],`
`], dtype=torch.int64)`

`src_lens` 记录的是补零前的真实长度，`src` 的3个样本分别长度为4, 3, 5；而 `trg` 的3个样本分别长4, 5, 3。
`src_lens = [4, 3, 5]`  
`trg_lens = [4, 5, 3]`

```python
# ---------------------#
# 2. Dataset & DataLoader
# ---------------------#


class TranslationDataset(Dataset):
    # 初始化方法，读取英文和中文训练文本。然后给每个句子前后增加 <bos> 和 <eos>。
    # 为了防止训练时显存不足，对于长度超过限制的句子进行过滤。
    def __init__(self, src_file, trg_file, src_tokenizer, trg_tokenizer, max_len=100):
        with open(src_file, encoding='utf-8') as f:
            src_lines = f.read().splitlines()

        with open(trg_file, encoding='utf-8') as f:
            trg_lines = f.read().splitlines()

        assert len(src_lines) == len(trg_lines)

        self.pairs = []
        self.src_tokenizer = src_tokenizer
        self.trg_tokenizer = trg_tokenizer

        for src, trg in zip(src_lines, trg_lines):
            # 每个句子前边增加 <bos>，后边增加 <eos>
            src_ids = [BOS_ID] + self.src_tokenizer(src) + [EOS_ID]
            trg_ids = [BOS_ID] + self.trg_tokenizer(trg) + [EOS_ID]

            # 只保留输入和输出序列 token 数同时小于 max_len 的训练样本
            if len(src_ids) <= max_len and len(trg_ids) <= max_len:
                self.pairs.append((src_ids, trg_ids))

    def __len__(self):
        return len(self.pairs)

    def __getitem__(self, idx):
        src_ids, trg_ids = self.pairs[idx]
        return torch.LongTensor(src_ids), torch.LongTensor(trg_ids)

    # 对一个 batch 的输入和输出 token 序列，依照最长序列长度，用 <pad> token 进行填充
    @staticmethod
    def collate_fn(batch):
        src_batch, trg_batch = zip(*batch)

        src_lens = [len(x) for x in src_batch]
        trg_lens = [len(x) for x in trg_batch]

        src_pad = nn.utils.rnn.pad_sequence(src_batch, padding_value=PAD_ID)
        trg_pad = nn.utils.rnn.pad_sequence(trg_batch, padding_value=PAD_ID)

        return src_pad, trg_pad, src_lens, trg_lens
```

---

## Model Definitions with Attention

把 `hidden: [1, B, H] -> [B, 1, H]`，`encoder_outputs: [S, B, 2H] -> [B, S, 2H]`。  
把 `hidden` 沿时间维复制到 `[B, S, H]`，与 `encoder_outputs` 拼接成 `[B, S, 3H]`。  
线性层 `attn: Linear(3H -> H)` + `tanh` 得到 `[B, S, H]`。  
投影 `v: Linear(H -> 1)` 并挤掉最后一维，得到 `[B, S]`。  
用 `mask=0` 处填充大负值，沿 `dim=1` softmax，得到 `[B, S]`。

```python
# ---------------------#
# 3. Model Definitions with Attention
# ---------------------#


class Attention(nn.Module):
    def __init__(self, hid_dim):
        super().__init__()
        # 第一层输入维度为 Encoder 的输出隐状态（双向，因此是 hid_dim * 2）
        # 和 Decoder 的输入隐状态（单向，因此是 hid_dim）的拼接
        self.attn = nn.Linear(hid_dim * 2 + hid_dim, hid_dim)

        # 输出一个代表注意力的 logit 值
        self.v = nn.Linear(hid_dim, 1, bias=False)

    def forward(self, hidden, encoder_outputs, mask):
        # [1, B, hid_dim] -> [B, 1, hid_dim]
        hidden = hidden.permute(1, 0, 2)

        # [src_len, B, hid_dim * 2] -> [B, src_len, hid_dim * 2]
        encoder_outputs = encoder_outputs.permute(1, 0, 2)
        src_len = encoder_outputs.shape[1]

        # 将当前 decoder 状态复制到每个源时间步
        hidden = hidden.repeat(1, src_len, 1)  # [B, src_len, hid_dim]

        # 计算注意力能量
        energy = torch.tanh(self.attn(torch.cat((hidden, encoder_outputs), dim=2)))
        attention = self.v(energy).squeeze(2)  # [B, src_len]

        # 对 pad 位置进行 mask
        attention = attention.masked_fill(mask == 0, -1e10)

        return torch.softmax(attention, dim=1)  # [B, src_len]
```

---

## Encoder

`forward` 里 `embedded = self.embedding(src)`，形状是 `[S, B, emb_dim]`，例如 `[5, 3, 512]`。  
作用是为每个源时间步生成双向上下文表示，并把每层最终状态压成单向大小交给 `Decoder`。

输入：
- `src: [S, B]`
- `src_len: List[int]`

输出：
- `encoder_outputs: [S, B, 2H]`
- `hidden: [L, B, H]`
- `cell: [L, B, H]`

```python
class Encoder(nn.Module):
    def __init__(self, vocab_size, emb_dim, hid_dim, n_layers=3):
        super().__init__()

        self.n_layers = n_layers

        # 定义 Embedding，将 token id 转为 emb_dim 维向量
        self.embedding = nn.Embedding(vocab_size, emb_dim, padding_idx=PAD_ID)

        # 双向 LSTM
        self.bi_lstm = nn.LSTM(emb_dim, hid_dim, n_layers, bidirectional=True)

        # 将双向状态降维给单向 Decoder
        self.fc_hidden = nn.ModuleList([
            nn.Linear(hid_dim * 2, hid_dim) for _ in range(n_layers)
        ])
        self.fc_cell = nn.ModuleList([
            nn.Linear(hid_dim * 2, hid_dim) for _ in range(n_layers)
        ])

    def forward(self, src, src_len):
        embedded = self.embedding(src)

        # 跳过 padding 部分的计算
        packed = nn.utils.rnn.pack_padded_sequence(
            embedded, src_len, enforce_sorted=False
        )

        outputs, (hidden, cell) = self.bi_lstm(packed)
        outputs, _ = nn.utils.rnn.pad_packed_sequence(outputs)  # [src_len, B, hid_dim * 2]

        # [n_layers * 2, B, hid_dim] -> [n_layers, 2, B, hid_dim]
        hidden = hidden.view(self.n_layers, 2, -1, hidden.size(2))
        cell = cell.view(self.n_layers, 2, -1, cell.size(2))

        final_hidden = []
        final_cell = []

        for layer in range(self.n_layers):
            h_cat = torch.cat((hidden[layer][0], hidden[layer][1]), dim=1)
            c_cat = torch.cat((cell[layer][0], cell[layer][1]), dim=1)

            h_layer = torch.tanh(self.fc_hidden[layer](h_cat)).unsqueeze(0)
            c_layer = torch.tanh(self.fc_cell[layer](c_cat)).unsqueeze(0)

            final_hidden.append(h_layer)
            final_cell.append(c_layer)

        hidden_concat = torch.cat(final_hidden, dim=0)
        cell_concat = torch.cat(final_cell, dim=0)

        return outputs, hidden_concat, cell_concat
```

---

## Decoder

功能：
- 单步生成下一个目标词的分布。
- 每一步都利用 `Encoder` 的时序输出做注意力加权。

输入：
- `input_token: LongTensor [B]`
- `hidden: FloatTensor [L, B, H]`
- `cell: FloatTensor [L, B, H]`
- `encoder_outputs: FloatTensor [S, B, 2H]`
- `mask: BoolTensor [B, S]`

输出：
- `prediction: FloatTensor [B, V]`
- `hidden: FloatTensor [L, B, H]`
- `cell: FloatTensor [L, B, H]`
- `attn_weights: FloatTensor [B, S]`

```python
class Decoder(nn.Module):
    def __init__(self, output_dim, emb_dim, hid_dim, attention, n_layers=3):
        super().__init__()

        self.output_dim = output_dim
        self.attention = attention
        self.n_layers = n_layers

        self.embedding = nn.Embedding(output_dim, emb_dim, padding_idx=PAD_ID)

        # 输入为 embedding 和注意力上下文向量的拼接
        self.rnn = nn.LSTM(hid_dim * 2 + emb_dim, hid_dim, num_layers=n_layers)

        # 输出分类头
        self.fc_out = nn.Linear(hid_dim * 3, output_dim)

    def forward(self, input_token, hidden, cell, encoder_outputs, mask):
        input_token = input_token.unsqueeze(0)  # [1, B]
        embedded = self.embedding(input_token)  # [1, B, emb_dim]

        last_hidden = hidden[-1].unsqueeze(0)

        a = self.attention(last_hidden, encoder_outputs, mask)  # [B, src_len]
        a = a.unsqueeze(1)  # [B, 1, src_len]

        encoder_outputs = encoder_outputs.permute(1, 0, 2)  # [B, src_len, 2 * hid_dim]

        # 注意力加权得到上下文向量
        weighted = torch.bmm(a, encoder_outputs)  # [B, 1, 2 * hid_dim]
        weighted = weighted.permute(1, 0, 2)  # [1, B, 2 * hid_dim]

        lstm_input = torch.cat((embedded, weighted), dim=2)
        output, (hidden, cell) = self.rnn(lstm_input, (hidden, cell))

        output = output.squeeze(0)      # [B, hid_dim]
        weighted = weighted.squeeze(0)  # [B, 2 * hid_dim]

        prediction = self.fc_out(torch.cat((output, weighted), dim=1))

        return prediction, hidden, cell, a.squeeze(1)
```

---

## Seq2Seq

功能：
- 调用 `Encoder` 得到整句表示和初始状态。
- 用目标序列的 `<bos>` 作为第一步输入，逐步调用 `Decoder`。
- 用源句 `mask` 屏蔽 `<pad>`。
- 收集每一步输出供训练时计算损失。

```python
class Seq2Seq(nn.Module):
    def __init__(self, encoder, decoder, device):
        super().__init__()

        self.encoder = encoder
        self.decoder = decoder
        self.device = device

    def forward(self, src, src_len, trg):
        batch_size = trg.shape[1]
        max_len = trg.shape[0]
        vocab_size = self.decoder.output_dim

        outputs = torch.zeros(max_len, batch_size, vocab_size).to(self.device)

        encoder_outputs, hidden, cell = self.encoder(src, src_len)
        input_token = trg[0]
        mask = (src != PAD_ID).permute(1, 0)

        for t in range(1, max_len):
            output, hidden, cell, _ = self.decoder(
                input_token, hidden, cell, encoder_outputs, mask
            )
            outputs[t] = output
            input_token = trg[t]

        return outputs
```

---

## Training

```python
# ---------------------#
# 4. Training
# ---------------------#


def train(model, iterator, optimizer, criterion):
    model.train()

    epoch_loss = 0
    step_loss = 0  # 用于累计每个 step 的 loss
    step_count = 0

    for i, (src, trg, src_len, _) in enumerate(iterator):
        src, trg = src.to(model.device), trg.to(model.device)

        optimizer.zero_grad()

        output = model(src, src_len, trg)
        output_dim = output.shape[-1]

        output = output[1:].reshape(-1, output_dim)
        trg = trg[1:].reshape(-1)

        loss = criterion(output, trg)
        loss.backward()
        optimizer.step()

        step_loss += loss.item()
        epoch_loss += loss.item()
        step_count += 1

        if (i + 1) % 100 == 0:
            avg_step_loss = step_loss / step_count
            print(f'Step [{i + 1}/{len(iterator)}] | Loss: {avg_step_loss:.4f}')
            step_loss = 0
            step_count = 0

    return epoch_loss / len(iterator)
```

---

## Testing / Inference

测试阶段和训练阶段最大的区别是：训练时 `Decoder` 每一步喂入的通常是真实目标词；而测试/推理时没有真实 `trg`，只能把上一步预测出来的词继续喂给下一步。

下面这个例子用的是最简单的 `greedy decoding`：

- 先把输入英文句子编码成 `src_ids`
- 用 `Encoder` 得到 `encoder_outputs / hidden / cell`
- 用 `<bos>` 作为解码起点
- 每一步取当前输出里概率最大的 token
- 直到遇到 `<eos>` 或达到最大长度

```python
# ---------------------#
# 5. Testing / Inference
# ---------------------#


def translate_sentence(sentence, model, src_tokenizer, trg_processor, device, max_len=50):
    model.eval()

    src_ids = [BOS_ID] + src_tokenizer(sentence) + [EOS_ID]
    src_tensor = torch.LongTensor(src_ids).unsqueeze(1).to(device)  # [S, 1]
    src_len = [len(src_ids)]

    with torch.no_grad():
        encoder_outputs, hidden, cell = model.encoder(src_tensor, src_len)
        mask = (src_tensor != PAD_ID).permute(1, 0)  # [1, S]

        input_token = torch.LongTensor([BOS_ID]).to(device)
        generated_ids = []

        for t in range(max_len):
            output, hidden, cell, _ = model.decoder(
                input_token, hidden, cell, encoder_outputs, mask
            )

            pred_token = output.argmax(1).item()

            if pred_token == EOS_ID:
                break

            generated_ids.append(pred_token)
            input_token = torch.LongTensor([pred_token]).to(device)

    translated_text = trg_processor.decode(generated_ids)
    return translated_text, generated_ids
```

---

## Main，调用训练与测试示例

调用Main会先进行训练，随后使用一个翻译case("I love natural language processing.")来测试模型的效果

```python
# ---------------------#
# 6. Main
# ---------------------#


if __name__ == '__main__':
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

    dataset = TranslationDataset(
        'data/en2cn/train_en.txt',
        'data/en2cn/train_zh.txt',
        tokenize_en,
        tokenize_cn,
    )

    loader = DataLoader(
        dataset,
        batch_size=32,
        shuffle=True,
        collate_fn=TranslationDataset.collate_fn,
    )

    INPUT_DIM = sp_en.get_piece_size()
    OUTPUT_DIM = sp_cn.get_piece_size()
    ENC_EMB_DIM = 512
    DEC_EMB_DIM = 512
    HID_DIM = 512

    attention = Attention(HID_DIM).to(device)
    encoder = Encoder(INPUT_DIM, ENC_EMB_DIM, HID_DIM).to(device)
    decoder = Decoder(OUTPUT_DIM, DEC_EMB_DIM, HID_DIM, attention).to(device)
    model = Seq2Seq(encoder, decoder, device).to(device)

    model.load_state_dict(torch.load('seq2seq_bpe_attention.pt', map_location=device))
    # 如果有原 pt 文件则加载参数权重继续训练；如果没有原 pt 文件，可以删去上面这行，
    # 代表从随机初始化的参数权重开始从零训练。

    optimizer = optim.Adam(model.parameters())
    criterion = nn.CrossEntropyLoss(ignore_index=PAD_ID)

    N_EPOCHS = 1

    for epoch in range(N_EPOCHS):
        loss = train(model, loader, optimizer, criterion)
        print(f'Epoch {epoch + 1}/{N_EPOCHS} | Loss: {loss:.4f}')
        torch.save(model.state_dict(), 'seq2seq_bpe_attention.pt')

    # 测试阶段：给一句英文，输出对应的中文翻译
    test_sentence = "I love natural language processing."
    translated_text, generated_ids = translate_sentence(
        test_sentence,
        model,
        tokenize_en,
        sp_cn,
        device,
    )

    print(f'Input: {test_sentence}')
    print(f'Output token ids: {generated_ids}')
    print(f'Translated text: {translated_text}')
```
