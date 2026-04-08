# Transformer的代码教程

本文记录 `Transformer` 的代码教程。

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

说明 conda 能正常调用，可以进入某个虚拟环境唤起 python。否则请查看教程 [tut_conda的下载安装](tut_conda的下载安装.md)

---

## 导入相关库

```python
import math

import torch
import torch.nn as nn
```

---

## 层归一化 LayerNorm

输入输出尺寸相同。  
例如 `x.shape = (B, S, H)`，其中：

- `B`: batch size
- `S`: sequence length
- `H`: hidden size
- `V`: vocabulary size

如果是bn则是对B进行归一化，如果是ln则是对H进行归一化

将H维向量归一化后乘以α加上ε得到尺寸不变的值（权重随机初始化后经训练确定），所有token使用同样的α和ε

```python
class LayerNormalization(nn.Module):
    def __init__(self, features: int, eps: float = 10 ** -6) -> None:
        super().__init__()
        self.eps = eps
        self.alpha = nn.Parameter(torch.ones(features))  # 可学习缩放参数
        self.bias = nn.Parameter(torch.zeros(features))  # 可学习偏置参数

    def forward(self, x):
        # x: (B, S, H)
        mean = x.mean(dim=-1, keepdim=True)  # (B, S, 1)
        std = x.std(dim=-1, keepdim=True)    # (B, S, 1)
        return self.alpha * (x - mean) / (std + self.eps) + self.bias
```

---

## 前馈网络 FeedForward

把 `H=512` 扩展到 `d_ff=2048`，经过 `ReLU + Dropout` 后再投影回 `H=512`。

```python
class FeedForwardBlock(nn.Module):
    def __init__(self, d_model: int, d_ff: int, dropout: float) -> None:
        super().__init__()
        self.linear_1 = nn.Linear(d_model, d_ff)
        self.dropout = nn.Dropout(dropout)
        self.linear_2 = nn.Linear(d_ff, d_model)

    def forward(self, x):
        # (B, S, d_model) -> (B, S, d_ff) -> (B, S, d_model)
        return self.linear_2(self.dropout(torch.relu(self.linear_1(x))))
```

---

## 词嵌入 Embedding

把输入 `(B, S)` 变成 `(B, S, H)`，并乘以 `sqrt(H)`。

```python
class InputEmbeddings(nn.Module):
    def __init__(self, d_model: int, vocab_size: int) -> None:
        super().__init__()
        self.d_model = d_model
        self.vocab_size = vocab_size
        self.embedding = nn.Embedding(vocab_size, d_model)

    def forward(self, x):
        # (B, S) -> (B, S, d_model)
        return self.embedding(x) * math.sqrt(self.d_model)
```

---

## PE 位置编码

```python
class PositionalEncoding(nn.Module):
    def __init__(self, d_model: int, seq_len: int, dropout: float) -> None:
        super().__init__()
        self.d_model = d_model
        self.seq_len = seq_len
        self.dropout = nn.Dropout(dropout)

        pe = torch.zeros(seq_len, d_model)  # (S, d_model)
        position = torch.arange(0, seq_len, dtype=torch.float).unsqueeze(1)  # (S, 1)
        div_term = torch.pow(
            10000.0,
            -torch.arange(0, d_model, 2, dtype=torch.float) / d_model,
        )

        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)

        pe = pe.unsqueeze(0)  # (1, S, d_model)
        self.register_buffer('pe', pe)

    def forward(self, x):
        x = x + self.pe[:, :x.shape[1], :].requires_grad_(False)
        return self.dropout(x)
```

---

## 残差连接

先做 `LayerNorm`，再走子层，接 `Dropout`，最后和原输入做残差相加。

```python
class ResidualConnection(nn.Module):
    def __init__(self, features: int, dropout: float) -> None:
        super().__init__()
        self.dropout = nn.Dropout(dropout)
        self.norm = LayerNormalization(features)

    def forward(self, x, sublayer):
        return x + self.dropout(sublayer(self.norm(x)))
```

---

## 多头自注意力模块

1. `Q/K/V` 计算：输入 `(B, S, H)`，通过同尺寸线性变换得到三个新的 `(B, S, H)`
2. `Q/K/V` 分头：将 `(B, S, H)` 变为 `(B, h, S, H / h)`，其中 `h` 为头数
3. 计算注意力权重矩阵 `attention_scores`：尺寸为 `(B, h, S, S)`
4. 得到更新后的值向量：尺寸为 `(B, h, S, H / h)`
5. 多头合并：将 `(B, h, S, H / h)` 合并为 `(B, S, H)`
6. 最后再做一次线性变换，整合头之间的信息，避免简单拼接

```python
class MultiHeadAttentionBlock(nn.Module):
    def __init__(self, d_model: int, h: int, dropout: float) -> None:
        super().__init__()
        self.d_model = d_model
        self.h = h

        assert d_model % h == 0, "d_model 不能被 h 整除"

        self.d_k = d_model // h
        self.w_q = nn.Linear(d_model, d_model, bias=False)
        self.w_k = nn.Linear(d_model, d_model, bias=False)
        self.w_v = nn.Linear(d_model, d_model, bias=False)
        self.w_o = nn.Linear(d_model, d_model, bias=False)
        self.dropout = nn.Dropout(dropout)

    @staticmethod
    def attention(query, key, value, mask, dropout: nn.Dropout):
        d_k = query.shape[-1]

        # (B, h, S, d_k) @ (B, h, d_k, S) -> (B, h, S, S)
        attention_scores = (query @ key.transpose(-2, -1)) / math.sqrt(d_k)

        if mask is not None:
            attention_scores.masked_fill_(mask == 0, -1e9)

        attention_scores = attention_scores.softmax(dim=-1)

        if dropout is not None:
            attention_scores = dropout(attention_scores)

        return (attention_scores @ value), attention_scores

    def forward(self, q, k, v, mask):
        query = self.w_q(q)
        key = self.w_k(k)
        value = self.w_v(v)

        # (B, S, d_model) -> (B, h, S, d_k)
        query = query.view(query.shape[0], query.shape[1], self.h, self.d_k).transpose(1, 2)
        key = key.view(key.shape[0], key.shape[1], self.h, self.d_k).transpose(1, 2)
        value = value.view(value.shape[0], value.shape[1], self.h, self.d_k).transpose(1, 2)

        x, self.attention_scores = MultiHeadAttentionBlock.attention(
            query, key, value, mask, self.dropout
        )

        # (B, h, S, d_k) -> (B, S, d_model)
        x = x.transpose(1, 2).contiguous().view(x.shape[0], -1, self.h * self.d_k)

        return self.w_o(x)
```

---

## 编码器(输入输出前后尺寸不变)

编码器中每个 `EncoderBlock` 包含：

- 一个自注意力子层
- 一个前馈子层
- 两个残差连接模块

编码器的作用是根据上下文，不断更新输入序列中每个token的Embedding。让最终输出序列token的embedding有最佳的语义。因为自注意力机制，可以一次性输入整个序列，序列内多个token可以并行处理，大大加快了处理速度。其中使用了位置编码增加了token的位置信息。通过多头注意力让token可以根据上下文信息修改自身embedding。通过全连接模块让每个token更新自身embedding。通过残差连接和LayerNorm让深度网络更容易训练。

```python
class EncoderBlock(nn.Module):
    def __init__(
        self,
        features: int,
        self_attention_block: MultiHeadAttentionBlock,
        feed_forward_block: FeedForwardBlock,
        dropout: float,
    ) -> None:
        super().__init__()
        self.self_attention_block = self_attention_block
        self.feed_forward_block = feed_forward_block
        self.residual_connections = nn.ModuleList(
            [ResidualConnection(features, dropout) for _ in range(2)]
        )

    def forward(self, x, src_mask):
        x = self.residual_connections[0](
            x, lambda x: self.self_attention_block(x, x, x, src_mask)
        )
        x = self.residual_connections[1](x, self.feed_forward_block)
        return x


class Encoder(nn.Module):
    def __init__(self, features: int, layers: nn.ModuleList) -> None:
        super().__init__()
        self.layers = layers
        self.norm = LayerNormalization(features)

    def forward(self, x, mask):
        for layer in self.layers:
            x = layer(x, mask)
        return self.norm(x)
```

---

## 解码器

解码器中的每个 `DecoderBlock` 包含：

- 一个带未来遮掩的自注意力子层
- 一个交叉注意力子层
- 一个前馈子层
- 三个残差连接模块

以`<bos> 我 爱 自然` 为例，这里一共4个token，一次训练能得到4个logits和4个loss，计算总梯度并回传实现并行训练
1）输入 `<bos>`，期望输出 `我`
2）输入 `<bos> 我`，期望输出 `爱`
3）输入 `<bos> 我 爱`，期望输出 `自然`
4）输入 `<bos> 我 爱 自然`，期望输出 `<eos>`

多头注意力，ln+多头注意力+dropout+残差链接
交叉注意力，ln+交叉注意力+dropout+残差链接
	交叉注意力模块的Q矩阵来自Decoder，K,V矩阵来自Encoder的输出
全连接ffb，ln+全连接变换+dropout+残差链接

```python
class DecoderBlock(nn.Module):
    def __init__(
        self,
        features: int,
        self_attention_block: MultiHeadAttentionBlock,
        cross_attention_block: MultiHeadAttentionBlock,
        feed_forward_block: FeedForwardBlock,
        dropout: float,
    ) -> None:
        super().__init__()
        self.self_attention_block = self_attention_block
        self.cross_attention_block = cross_attention_block
        self.feed_forward_block = feed_forward_block
        self.residual_connections = nn.ModuleList(
            [ResidualConnection(features, dropout) for _ in range(3)]
        )

    def forward(self, x, encoder_output, src_mask, tgt_mask):
        x = self.residual_connections[0](
            x, lambda x: self.self_attention_block(x, x, x, tgt_mask)
        )
        x = self.residual_connections[1](
            x,
            lambda x: self.cross_attention_block(
                x, encoder_output, encoder_output, src_mask
            ),
        )
        x = self.residual_connections[2](x, self.feed_forward_block)
        return x


class Decoder(nn.Module):
    def __init__(self, features: int, layers: nn.ModuleList) -> None:
        super().__init__()
        self.layers = layers
        self.norm = LayerNormalization(features)

    def forward(self, x, encoder_output, src_mask, tgt_mask):
        for layer in self.layers:
            x = layer(x, encoder_output, src_mask, tgt_mask)
        return self.norm(x)
```

---

## 输出映射

把 `(B, S, H)` 映射成 `(B, S, V)`。
得到的logit后续再经过softmax即可得到每个词的概率

```python
class ProjectionLayer(nn.Module):
    def __init__(self, d_model, vocab_size) -> None:
        super().__init__()
        self.proj = nn.Linear(d_model, vocab_size)

    def forward(self, x):
        return self.proj(x)
```

---

## Transformer

Encode
- 输入 `src` 的形状是 `(B, S)`，`src_mask` 的形状是 `(B, 1, 1, S)`
- `src` 先经过 embedding，变成 `(B, S, H)`
- 加上位置编码后，形状仍然是 `(B, S, H)`
- 经过 `Encoder` 和 `src_mask` 后，输出仍然是 `(B, S, H)`

Decode
- 输入 `encoder_output` 的形状是 `(B, S, H)`
- 输入 `tgt` 的形状是 `(B, T)`
- 输入 `tgt_mask` 的形状是 `(B, 1, T, T)`
- `tgt` 先经过 embedding 和位置编码，变成 `(B, T, H)`
- 经过 `Decoder` 和 `tgt_mask` 后，输出是 `(B, T, H)`

Project
- 将 `(B, T, H)` 投影为 `(B, T, V)`

```python
class Transformer(nn.Module):
    def __init__(
        self,
        encoder: Encoder,
        decoder: Decoder,
        src_embed: InputEmbeddings,
        tgt_embed: InputEmbeddings,
        src_pos: PositionalEncoding,
        tgt_pos: PositionalEncoding,
        projection_layer: ProjectionLayer,
    ) -> None:
        super().__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.src_embed = src_embed
        self.tgt_embed = tgt_embed
        self.src_pos = src_pos
        self.tgt_pos = tgt_pos
        self.projection_layer = projection_layer

    def encode(self, src, src_mask):
        src = self.src_embed(src)
        src = self.src_pos(src)
        return self.encoder(src, src_mask)

    def decode(
        self,
        encoder_output: torch.Tensor,
        src_mask: torch.Tensor,
        tgt: torch.Tensor,
        tgt_mask: torch.Tensor,
    ):
        tgt = self.tgt_embed(tgt)
        tgt = self.tgt_pos(tgt)
        return self.decoder(tgt, encoder_output, src_mask, tgt_mask)

    def project(self, x):
        return self.projection_layer(x)
```

---

## 组装

```python
def build_transformer(
    src_vocab_size: int,
    tgt_vocab_size: int,
    src_seq_len: int,
    tgt_seq_len: int,
    d_model: int = 512,
    N: int = 6,
    h: int = 8,
    dropout: float = 0.1,
    d_ff: int = 2048,
) -> Transformer:
    src_embed = InputEmbeddings(d_model, src_vocab_size)
    tgt_embed = InputEmbeddings(d_model, tgt_vocab_size)

    src_pos = PositionalEncoding(d_model, src_seq_len, dropout)
    tgt_pos = PositionalEncoding(d_model, tgt_seq_len, dropout)

    encoder_blocks = []
    for _ in range(N):
        encoder_self_attention_block = MultiHeadAttentionBlock(d_model, h, dropout)
        feed_forward_block = FeedForwardBlock(d_model, d_ff, dropout)
        encoder_block = EncoderBlock(
            d_model, encoder_self_attention_block, feed_forward_block, dropout
        )
        encoder_blocks.append(encoder_block)

    decoder_blocks = []
    for _ in range(N):
        decoder_self_attention_block = MultiHeadAttentionBlock(d_model, h, dropout)
        decoder_cross_attention_block = MultiHeadAttentionBlock(d_model, h, dropout)
        feed_forward_block = FeedForwardBlock(d_model, d_ff, dropout)
        decoder_block = DecoderBlock(
            d_model,
            decoder_self_attention_block,
            decoder_cross_attention_block,
            feed_forward_block,
            dropout,
        )
        decoder_blocks.append(decoder_block)

    encoder = Encoder(d_model, nn.ModuleList(encoder_blocks))
    decoder = Decoder(d_model, nn.ModuleList(decoder_blocks))
    projection_layer = ProjectionLayer(d_model, tgt_vocab_size)

    transformer = Transformer(
        encoder, decoder, src_embed, tgt_embed, src_pos, tgt_pos, projection_layer
    )

    for p in transformer.parameters():
        if p.dim() > 1:
            nn.init.xavier_uniform_(p)

    return transformer
```

---

# 翻译任务

## 导入基础库

```python
import sentencepiece as spm
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, Dataset

from transformer import build_transformer

DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")

sp_en = spm.SentencePieceProcessor()
sp_en.load('en_bpe.model')

sp_cn = spm.SentencePieceProcessor()
sp_cn.load('zh_bpe.model')


def tokenize_en(text):
    return sp_en.encode(text, out_type=int)


def tokenize_cn(text):
    return sp_cn.encode(text, out_type=int)


PAD_ID = sp_en.pad_id()  # 1
UNK_ID = sp_en.unk_id()  # 0
BOS_ID = sp_en.bos_id()  # 2
EOS_ID = sp_en.eos_id()  # 3
```

---

## 符号说明

- `B`: batch size
- `S`: source sequence length
- `T`: target sequence length
- `H`: hidden size
- `V`: vocabulary size
- `src`: 源语言输入序列
- `tgt`: 目标语言输出序列

---

## DataLoader

流程如下：

1. 读取英文和中文训练文本
2. 每行先分词，再转为 token id
3. 每个句子前后补上 `<bos>` 和 `<eos>`
4. 过滤掉长度超过限制的样本
5. 在 `collate_fn` 里统一 padding

注意：Transformer 这里用的是 `batch_first=True`，意思是batch_size放在第一维，所以 `src_pad` / `tgt_pad` 形状是 `[B, S]` 或 `[B, T]`。

```python
class TranslationDataset(Dataset):
    # 读取双语文本，并在句子两端加上 <bos> / <eos>
    def __init__(self, src_file, trg_file, src_tokenizer, trg_tokenizer, max_len=100):
        with open(src_file, encoding='utf-8') as f:
            src_lines = f.read().splitlines()

        with open(trg_file, encoding='utf-8') as f:
            trg_lines = f.read().splitlines()

        assert len(src_lines) == len(trg_lines)

        self.pairs = []
        self.src_tokenizer = src_tokenizer
        self.trg_tokenizer = trg_tokenizer

        index = 0
        for src, trg in zip(src_lines, trg_lines):
            index += 1
            if index % 100000 == 0:
                print(index)

            src_ids = [BOS_ID] + self.src_tokenizer(src) + [EOS_ID]
            trg_ids = [BOS_ID] + self.trg_tokenizer(trg) + [EOS_ID]

            if len(src_ids) <= max_len and len(trg_ids) <= max_len:
                self.pairs.append((src_ids, trg_ids))

    def __len__(self):
        return len(self.pairs)

    def __getitem__(self, idx):
        src_ids, trg_ids = self.pairs[idx]
        return torch.LongTensor(src_ids), torch.LongTensor(trg_ids)

    @staticmethod
    def collate_fn(batch):
        src_batch, trg_batch = zip(*batch)

        src_lens = [len(x) for x in src_batch]
        trg_lens = [len(x) for x in trg_batch]

        src_pad = nn.utils.rnn.pad_sequence(
            src_batch, batch_first=True, padding_value=PAD_ID
        )
        trg_pad = nn.utils.rnn.pad_sequence(
            trg_batch, batch_first=True, padding_value=PAD_ID
        )

        return src_pad, trg_pad, src_lens, trg_lens
```

---

## Mask

需要两个 mask：

- `src_mask`：屏蔽源序列中的 `<pad>`
- `tgt_mask`：既要屏蔽 `<pad>`，也要屏蔽未来 token

```python
def create_mask(src, tgt, pad_idx):
    # encoder mask: (B, 1, 1, S)
    src_mask = (src != pad_idx).unsqueeze(1).unsqueeze(2)

    # decoder pad mask: (B, 1, 1, T)
    tgt_pad_mask = (tgt != pad_idx).unsqueeze(1).unsqueeze(2)

    tgt_len = tgt.size(1)

    # decoder causal mask: (T, T)
    tgt_sub_mask = torch.tril(
        torch.ones((tgt_len, tgt_len), device=tgt.device)
    ).bool()

    # final decoder mask: (B, 1, T, T)
    tgt_mask = tgt_pad_mask & tgt_sub_mask

    return src_mask, tgt_mask
```

---

## Train

训练时，Decoder 的输入和输出要错开一位：

- `tgt_input = tgt[:, :-1] # decoder 输入：<bos>id 我id 爱id 自然id`
- `tgt_output = tgt[:, 1:] # decoder 输出：我id 爱id 自然id <eos>id`

这样模型在每个位置都在预测“下一个词”。

以下输入的都是词id，方便区分所以写的词
举例decoder输入`<bos> 我 爱 自然`，期望decoder输出`我 爱 自然 <eos>`
这里一共4个token，一次训练能得到4个logits和4个loss，计算总梯度并回传实现并行训练
1）输入 `<bos>`，期望输出 `我`
2）输入 `<bos> 我`，期望输出 `爱`
3）输入 `<bos> 我 爱`，期望输出 `自然`
4）输入 `<bos> 我 爱 自然`，期望输出 `<eos>`
根据四次的总梯度来并行更新所有模块的权重

output = model.project(decoder_output)
这里尺寸为 `(B, T, V)`
`reshape` 后变为 `(B * T, V)`，也就是“每个 token 对应一个词表 logits”
tgt_output本来就是`每个token*每个样本的分词id`
类似于类别数为 `V` 的分类问题

```python
def train(model, dataloader, optimizer, criterion, pad_idx):
    model.train()

    total_loss = 0
    step = 0
    log_loss = 0

    for src, tgt, src_lens, tgt_lens in dataloader:
        step += 1

        src = src.to(DEVICE)
        tgt = tgt.to(DEVICE)

        tgt_input = tgt[:, :-1]
        tgt_output = tgt[:, 1:]

        src_mask, tgt_mask = create_mask(src, tgt_input, pad_idx)

        optimizer.zero_grad()

        encoder_output = model.encode(src, src_mask)
        decoder_output = model.decode(encoder_output, src_mask, tgt_input, tgt_mask)
        output = model.project(decoder_output)

        output = output.reshape(-1, output.shape[-1])
        tgt_output = tgt_output.reshape(-1)

        loss = criterion(output, tgt_output)
        loss.backward()
        optimizer.step()

        total_loss += loss.item()
        log_loss += loss.item()

        if step % 100 == 0:
            avg_log_loss = log_loss / 100
            print(f"Step {step}: Avg Loss = {avg_log_loss:.4f}")
            log_loss = 0

    return total_loss / len(dataloader)
```

---

## Testing / Inference

测试阶段和训练阶段的区别在于：训练时 `Decoder` 会看到右移后的目标序列 `tgt_input`，而推理时没有真实目标句，只能从 `<bos>` 开始一步一步自己生成。

下面这个例子使用最简单的 `greedy decoding`：

- 先把英文句子编码成 `src_ids`
- 调用 `Encoder` 得到 `encoder_output`
- 用 `<bos>` 作为解码起点
- 每一步取当前时刻概率最大的 token
- 遇到 `<eos>` 就停止

```python
def greedy_decode(model, src, src_mask, max_len, start_symbol, end_symbol):
    encoder_output = model.encode(src, src_mask)

    ys = torch.tensor([[start_symbol]], dtype=torch.long, device=src.device)  # [1, 1]

    for _ in range(max_len - 1):
        _, tgt_mask = create_mask(src, ys, PAD_ID)

        decoder_output = model.decode(encoder_output, src_mask, ys, tgt_mask)
        prob = model.project(decoder_output[:, -1:])  # [1, 1, V]
        next_word = prob[:, -1, :].argmax(dim=-1).item()

        ys = torch.cat(
            [ys, torch.tensor([[next_word]], dtype=torch.long, device=src.device)],
            dim=1,
        )

        if next_word == end_symbol:
            break

    return ys.squeeze(0).tolist()


def translate_sentence(sentence, model, src_tokenizer, trg_processor, device, max_len=50):
    model.eval()

    src_ids = [BOS_ID] + src_tokenizer(sentence) + [EOS_ID]
    src_tensor = torch.LongTensor(src_ids).unsqueeze(0).to(device)  # [1, S]
    src_mask = (src_tensor != PAD_ID).unsqueeze(1).unsqueeze(2)  # [1, 1, 1, S]

    with torch.no_grad():
        output_ids = greedy_decode(
            model,
            src_tensor,
            src_mask,
            max_len=max_len,
            start_symbol=BOS_ID,
            end_symbol=EOS_ID,
        )

    # 去掉开头的 <bos>，并在遇到 <eos> 后停止
    if output_ids and output_ids[0] == BOS_ID:
        output_ids = output_ids[1:]
    if EOS_ID in output_ids:
        output_ids = output_ids[:output_ids.index(EOS_ID)]

    translated_text = trg_processor.decode(output_ids)
    return translated_text, output_ids
```

---

## Main

调用Main会先进行训练，随后使用一个翻译case("I love natural language processing.")来测试模型的效果

```python
def main():
    SRC_VOCAB_SIZE = 16000
    TGT_VOCAB_SIZE = 16000
    SRC_SEQ_LEN = 128
    TGT_SEQ_LEN = 128
    BATCH_SIZE = 2
    NUM_EPOCHS = 10
    LR = 1e-4

    train_dataset = TranslationDataset(
        'data/en2cn/train_en.txt',
        'data/en2cn/train_zh.txt',
        tokenize_en,
        tokenize_cn,
    )
    train_dataloader = DataLoader(
        train_dataset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        collate_fn=train_dataset.collate_fn,
    )

    model = build_transformer(
        SRC_VOCAB_SIZE, TGT_VOCAB_SIZE, SRC_SEQ_LEN, TGT_SEQ_LEN
    ).to(DEVICE)

    optimizer = optim.Adam(model.parameters(), lr=LR)
    criterion = nn.CrossEntropyLoss(ignore_index=PAD_ID)

    for epoch in range(NUM_EPOCHS):
        loss = train(model, train_dataloader, optimizer, criterion, PAD_ID)
        print(f"Epoch {epoch + 1}/{NUM_EPOCHS} - Loss: {loss:.4f}")
        torch.save(model.state_dict(), "transformer.pt")

    # 测试阶段：输入一句英文，输出对应中文翻译
    test_sentence = "I love natural language processing."
    translated_text, output_ids = translate_sentence(
        test_sentence,
        model,
        tokenize_en,
        sp_cn,
        DEVICE,
    )

    print(f"Input: {test_sentence}")
    print(f"Output token ids: {output_ids}")
    print(f"Translated text: {translated_text}")


if __name__ == "__main__":
    main()
```
