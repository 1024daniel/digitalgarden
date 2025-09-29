---
{"dg-publish":true,"permalink":"/media/ffmpeg命令汇总/","noteIcon":"3"}
---

#ffmpeg
1.将音频文件和视频文件合成影片
```sh
ffmpeg -c:v copy -c:a aac -strict experimental output.mp4 -i 1997-10-audio.mp4  -i 1997-10-video.mp4 

```

• -c:v copy: 复制视频流，不重新编码视频。

• -c:a aac: 将音频编码为 AAC 格式。

• -strict experimental: 允许使用某些实验性的编码选项（例如 AAC）。


2.将webm音频文件转为mov文件，final cut pro不支持webm文件，需要进行转换


```sh
ffmpeg -i input.webm -c:v prores -c:a aac output.mov

```