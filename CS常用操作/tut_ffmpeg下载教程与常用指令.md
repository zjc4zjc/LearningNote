# ffmpeg下载教程与常用指令

本文记录 `ffmpeg` 的下载教程与常用指令。

## 进入官网下载安装包

进入[https://www.gyan.dev/ffmpeg/builds/](https://www.gyan.dev/ffmpeg/builds/)，下载ffmpeg-git-full.7z，并配置环境变量

## 指令

以下命令可以暴力压缩 `input.mp4` 文件，压缩后输出文件为 `output.mp4`
```bash
ffmpeg -i "input.mp4" -c:v libx265 -crf 28 -vf "scale=iw/2:ih/2" -c:a aac -b:a 128k output.mp4
```



