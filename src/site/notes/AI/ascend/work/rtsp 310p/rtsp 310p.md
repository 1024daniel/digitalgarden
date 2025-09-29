---
{"dg-publish":true,"permalink":"/AI/ascend/work/rtsp 310p/rtsp 310p/","noteIcon":"3"}
---

### 待适配代码
ffmpeg rtsp拉流，Jetson解码，编码示例
[[rtsp_vidocapture.cpp]]
其中示例中

YUV格式介绍:
[[media/YUV\|YUV]]

核心的视频解码代码:
```c++
while (av_read_frame(fmt_ctx, &pkt) >= 0) {
    // 把压缩包送进解码器
    avcodec_send_packet(codec_ctx, &pkt);

    // 解码器可能立即输出，也可能缓存等待
    while (avcodec_receive_frame(codec_ctx, frame) >= 0) {
        // ✅ 拿到原始帧，可以处理
    }

    av_packet_unref(&pkt);
}

// 送 NULL packet，强制解码器吐出剩余帧 (flush)
avcodec_send_packet(codec_ctx, NULL);
while (avcodec_receive_frame(codec_ctx, frame) >= 0) {
    // ✅ flush 阶段取剩余帧
}

```

注意这里的代码逻辑，从rtsp中读取一帧，将这一帧送到硬件进行解码，针对这一个送入的原始数据帧(压缩数据包，I帧/B帧/P帧)，需要while循环持续输出解码帧，这里主要是考虑到GOP(Groups of Pictures)中，一个I帧压缩数据包输入，解码器会输出一个完整的视频帧，但是输入一个B帧的话，当前不会输出一帧，解码器会进行缓存数据，等待下一帧，如果接受到的是P帧的话，会将前面的B帧等全部和自己当前帧进行解码输出

- **I**‑frames are the least compressible but don't require other video frames to decode.
- **P**‑frames can use data from previous frames to decompress and are more compressible than I‑frames.
- **B**‑frames can use both previous and forward frames for data reference to get the highest amount of data compression.
![Pasted image 20250928201937.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020250928201937.png)

vision sdk rtsp拉流解码编码实例:
```cardlink
url: https://www.hiascend.com/zh/marketplace/mindx-sdk/case-studies/c34603bb-0d0c-475d-911e-d07cfffe2c7b
title: "视频转码-AI应用案例-昇腾社区"
description: "本应用以昇腾Atlas 300I Pro、Atlas 300V Pro为主要的硬件平台，视频转码（MediaCodec）常用于智慧城市交通行业，涉及视频格式转换和分辨率缩减等过程，对实时性与性能有极大的挑战。本案例基于 Vision SDK 实现多路视频实时高性能转码能力。"
host: www.hiascend.com
favicon: https://www.hiascend.com/_static3/favicon.ico
image: https://www.hiascend.com/_static3/logo.BmJjMz5z.png
```




```cardlink
url: https://gitee.com/ascend/mindsdk-referenceapps/blob/master/VisionSDK/MediaCodec/main.cpp
title: "VisionSDK/MediaCodec/main.cpp · Ascend/mindsdk-referenceapps - Gitee.com"
description: "MindSDK Reference Apps"
host: gitee.com
image: https://gitee.com/static/images/logo_themecolor_circle.png
```

上述示例
解码编码样例一些配置都是写死的，且操作都是AddPlugin之类的，和给定ffmpeg的示例代码相差比较大



```cardlink
url: https://gitee.com/ascend/mindsdk-referenceapps/blob/master/tutorials/VideoEncoder&VideoDecoder/C++/main.cpp
title: "tutorials/VideoEncoder&VideoDecoder/C++/main.cpp · Ascend/mindsdk-referenceapps - Gitee.com"
description: "MindSDK Reference Apps"
host: gitee.com
image: https://gitee.com/static/images/logo_themecolor_circle.png
```
这个示例接近给出的示例，但是没有类似找解码器的ffmpeg接口的操作，且输入是本地的视频文件而非rtsp流，待验证




```cardlink
url: https://gitee.com/ascend/mindsdk-referenceapps/blob/master/VisionSDK/AscendFFmpegPlugin/README.md#ffmpeg-ascend-plugin
title: "VisionSDK/AscendFFmpegPlugin/README.md · Ascend/mindsdk-referenceapps - Gitee.com"
description: "MindSDK Reference Apps"
host: gitee.com
image: https://gitee.com/static/images/logo_themecolor_circle.png
```
AscendFFmpegPlugin插件，插件如果功能正常的话针对原始脚本修改会更小