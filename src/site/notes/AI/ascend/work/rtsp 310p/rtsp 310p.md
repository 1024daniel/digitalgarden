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

测试数据集:

```cardlink
url: https://telecommunication-telemedia-assessment.github.io/AVT-VQDB-UHD-1/test_1.html
title: "Test 1 Videos"
description: "This site contains the Test 1 videos"
host: telecommunication-telemedia-assessment.github.io
```


### rtsp流创建


```cardlink
url: https://gitee.com/ascend/mindsdk-referenceapps/blob/master/docs/%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99/Live555%E7%A6%BB%E7%BA%BF%E8%A7%86%E9%A2%91%E8%BD%ACRTSP%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3.md
title: "docs/参考资料/Live555离线视频转RTSP说明文档.md · Ascend/mindsdk-referenceapps - Gitee.com"
description: "MindSDK Reference Apps"
host: gitee.com
image: https://gitee.com/static/images/logo_themecolor_circle.png
```
注意live555mediaserver支持的文件格式有限，需要使用ffmpeg针对视频文件进行转换成支持的格式放在需要对外推流的文件夹

```sh
# 将输入的视频文件转换成live555mediaserver可以成功推流的格式
# - 去掉 MP4 封装 + （必要时）AVCC→Annex-B 格式转换。
ffmpeg -i input.mp4 -c:v h264_ascend -an -bsf:v h264_mp4toannexb output.264

```

目标cc文件需要g++编译，但是ffmpeg中的hwconfig.json存在结构体有public变量这个字段，在C中属于正常，但是这个属于cpp的修饰词
[[ProgrammingLanguages/Cpp/build-set/宏\|宏]]

[[media/ffmpeg命令汇总\|ffmpeg命令汇总]]
```sh
ffprobe -rtsp_transport tcp rtsp://127.0.0.1/test.264

```

### 编译走通

蓝区机器只有800I，先在800I走通编译，之后代码同步绿区310p机器进行验证，按理都能成功跑起来


![Pasted image 20251009152022.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009152022.png)
![Pasted image 20251009152245.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009152245.png)
当前patch的版本为4.4.1，没有opaque_ref字段，该字段为用户提供从packet->frame传递自定义数据的buff
从5.0+版本之后才有
![Pasted image 20251009155543.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009155543.png)

基于n4.4.1的patch文件在n5.0的版本进行修改
编译第一个错误:

![Pasted image 20251009165637.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009165637.png)
新版本pkt_pts字段已经废弃，直接删除413行的赋值进行下一步

```cardlink
url: https://lists.ffmpeg.org/pipermail/ffmpeg-cvslog/2021-April/127382.html?utm_source=chatgpt.com
title: "Making sure you're not a bot!"
host: lists.ffmpeg.org
```

![Pasted image 20251009170603.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009170603.png)
首先在n4.4.1的版本中找到这个字段定义，之后从这个定义的值在n5.0进行反查当前对应的变量名，可以发现对应的变量和编译报错提示替代词一致，直接进行替换
之后出现新报错
![Pasted image 20251009171924.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009171924.png)
可以说明代码依赖的ffmpeg版本为比n5.0更高版本，ffmpeg master分支查看该字段的含义
![Pasted image 20251009172157.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009172157.png)
该字段为标识编码器具备可重排opaque值的能力

```cardlink
url: https://ffmpeg.org/doxygen/7.0/group__lavc__core.html?utm_source=chatgpt.com#gaf13081b482792279ac9d243c547baa76
title: "FFmpeg: Core functions/structures."
host: ffmpeg.org
```
![Pasted image 20251009172642.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251009172642.png)

查看代码发现该字段从n6.0开始才有，注意ffmpeg仓库tags中以n开头的是release版本，当前我们对齐最新n8.0版本进行适配

注意fftools/ffmpeg_hw.c下函数hw_device_setup_for_decode在n8.0版本迁移到fftools/ffmpeg_dec.c下


同理按照patch更改之后出现字段缺失
![Pasted image 20251010113517.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251010113517.png)

查看code tree，n5.0版本还保留了internal字段，n6.0之后删除了internal字段
![Pasted image 20251010113842.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251010113842.png)
搜索关键字ctx->internal可以看到关键字段替换之后对应调用处的更改，如上图`CUDAFramesContext* priv`从`ctx->internal->priv`获取换成了从`ctx->hwctx`获取

同时可以看到原本patch中需要获取`ctx->internal->pool`，但是
```c
typedef struct AVHWFramesContext {
    ...
    AVBufferPool *pool;        // 用户可自定义的 buffer 池
    ...
    struct AVHWFramesInternal *internal;
} AVHWFramesContext;

typedef struct AVHWFramesInternal {
    void *priv;                // 硬件上下文私有数据（hwctx）
    AVBufferPool *pool_internal; // 由 FFmpeg 内部维护的 pool
    ...
} AVHWFramesInternal;

```

- ctx->pool 是 **用户可提供** 的池；
    
- ctx->internal->pool_internal 是 **FFmpeg 内部自动创建的默认池**；
    
- 如果用户没有设置 ctx->pool，av_hwframe_ctx_init() 会自动用内部的 pool_internal 替代它；
    
- 从外部逻辑看，这两者最后表现 **几乎一致**（同指向底层的 buffer 池）。

新版本
```c
typedef struct FFHWFramesContext {
    AVHWFramesContext p;// 公共结构体（对外暴露）
    const HWContextType *hw_type;// 指向硬件类型定义（如 cuda, vaapi, ascend）
    AVBufferPool *pool_internal; // ✅ 内部 pool 指针
    AVBufferRef *source_frames;
    int source_allocation_map_flags;
} FFHWFramesContext;

typedef struct AVHWFramesContext {
    ...
    AVBufferPool *pool;        // 用户可自定义的 buffer 池，硬件驱动和外部api可访问
} AVHWFramesContext;

```
同时需要关注函数`av_hwframe_ctx_init`, 从n5.0版本函数和n8.0函数和上面提到的字段变化可以看到去除了AVHWFramesInternal字段，将该pool_internal这个由ffmepg内部管理的字段放入了新增struct FFHWFramesContext中的pool_internal

且从函数`av_hwframe_ctx_init`函数中可以看到ffmpeg管理的pool_internal和硬件驱动侧的pool都会指向同一个由硬件驱动侧frames_init()创建的buffer
```c
    if (ctxi->pool_internal && !ctx->pool)
        ctx->pool = ctxi->pool_internal;

```

所以对于原本patch中对于ctx->internal->pool_internal中的的创建直接改为ctx->pool的创建，逻辑不变
之后出现新报错如下

![Pasted image 20251010165719.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251010165719.png)
改字段已经换成frames_hwctx_size，替换之后重新编译出现报错
![Pasted image 20251010170525.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251010170525.png)
可以新版本hw_device_setup_for_decode函数入参变化没有ist，
InputStream* ist换成对应的DecoderPriv* dp
ist->decoder_opts换成对应的dp->standalone_init.opts
替换之后编译成功

```sh
# 展示可用的encoders可以看到ascend
ffmpeg -encoders

```

之前n4.4.1的版本打完patch之后上述命令能看到ascend的encoder，但是n8.0按照上述patch和debug修改之后没有看到类似的组件
![Pasted image 20251011165647.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251011165647.png)
且`nm /opt/8.0ffmpeg/lib/libavcodec.a |grep ascend`没有对应的字段
排查发现是新版encoder和decoder注册写法变化，需要对齐


### ffmpeg n8.0 encoder/decoder注册

如上所述，旧版本n4.4.1注册写法与n8.0的版本不兼容，即使ffmpeg在patch ascend之后成功编译出包但是ffmpeg命令并未看到对应的编解码器
首先先分析n4.4.1编译注册流程:
```sh
export ASCEND_HOME=/usr/local/Ascend
. /usr/local/Ascend/ascend-toolkit/set_env.sh
./configure \
    --prefix=/opt/8.0ffmpeg \
    --enable-shared \
    --extra-cflags="-I${ASCEND_HOME}/ascend-toolkit/latest/acllib/include" \
    --extra-ldflags="-L${ASCEND_HOME}/ascend-toolkit/latest/acllib/lib64" \
    --extra-libs="-lacl_dvpp_mpi -lascendcl" \
    --enable-ascend \
    && make -j && make install

```
这里主要分析`--enable-ascend`在configure中的
```sh
set_all(){
    value=$1
    shift
    for var in $*; do
        eval $var=$value
    done
}

enable(){
    set_all yes $*
}

for opt do
    optval="${opt#*=}"
    case "$opt" in
	            --enable-?*|--disable-?*)
	            eval $(echo "$opt" | sed 's/--/action=/;s/-/ option=/;s/-/_/g')
	            # action='enabel' option='ascend'
	            if is_in $option $COMPONENT_LIST; then
	                test $action = disable && action=unset
	                eval $action \$(toupper ${option%s})_LIST
	            elif is_in $option $CMDLINE_SELECT; then
		            # 'ascend' is in CMDLINE_SELECT, exec below
		            # enable ascend
	                $action $option
	            else
	                die_unknown $opt
	            fi

```
从以上逻辑可以看到`--enable-ascend`会在configure中设置环境变量`ascend=yes`

![Pasted image 20251014154820.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251014154820.png)

![Pasted image 20251014154836.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251014154836.png)


![Pasted image 20251014180834.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251014180834.png)
可以看到n4.4.1相当于n8.0版本有CONFIG_H264_ASCEND_DECODER=yes之类的定义，这个定义设置直接控制o文件中是否包含对应的函数，Makefile文件也通过对应的环境变量控制.o文件是否加入so文件
![Pasted image 20251014154919.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251014154919.png)
![Pasted image 20251014154931.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251014154931.png)
`CONFIG_`开头的环境变量和宏是通过configure中以下内容实现，主要是第一个参数是变量名前缀，第二个参数是文件列表，后面的参数都是需要和CONFIG_进行拼接的变量名
```sh
print_config(){
    pfx=$1
    files=$2
    shift 2
    map 'eval echo "$v \${$v:-no}"' "$@" |
    awk "BEGIN { split(\"$files\", files) }
        {
            c = \"$pfx\" toupper(\$1);
            v = \$2;
            sub(/yes/, 1, v);
            sub(/no/,  0, v);
            for (f in files) {
                file = files[f];
                if (file ~ /\\.h\$/) {
                    printf(\"#define %s %d\\n\", c, v) >>file;
                } else if (file ~ /\\.asm\$/) {
                    printf(\"define %s %d\\n\", c, v) >>file;
                } else if (file ~ /\\.mak\$/) {
                    n = -v ? \"\" : \"!\";
                    printf(\"%s%s=yes\\n\", n, c) >>file;
                } else if (file ~ /\\.texi\$/) {
                    pre = -v ? \"\" : \"@c \";
                    yesno = \$2;
                    c2 = tolower(c);
                    gsub(/_/, \"-\", c2);
                    printf(\"%s@set %s %s\\n\", pre, c2, yesno) >>file;
                }
            }
        }"
}
cp_if_changed(){
    cmp -s "$1" "$2" && { test "$quiet" != "yes" && echo "$2 is unchanged"; } && return
    mkdir -p "$(dirname $2)"
    cp -f "$1" "$2"
}
#  TMPH means TMP Header file，会生成config.h，config_components.h等文件内容
cp_if_changed $TMPH config.h


```
上述从configure摘抄出来的代码逻辑主要是在/tmp中创建一个configure运行期间中间文件/tmp/test.h，之后通过指定的--enable-ascend之后产生的一些环境变量，通过pirnt_config将这些变量通过宏的形式写入(不同文件后缀不同写入格式，比如mak文件以`a=yes`形式写入)
![Pasted image 20251015112011.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251015112011.png)
从上述可以看到，原本n4.4.1的所有的宏都是写入config.h文件，新版部分宏是写到新增文件config_components.h，因此只需要在ascend_enc.h头文件添加`#include "config_components.h"`即可

```sh
	

```

### ffmpeg 命令行转码测试
在以上基础上n8.0版本ffmpeg命令行可以识别到h264 ascend的encoder/decoder，通过转码测试功能是否正常:
```sh
ffmpeg -loglevel debug -i bigbuck_bunny_8bit_2000kbps_720p_60.0fps_h264.mp4 -c:v h264_ascend -an -bsf:v h264_mp4toannexb output.264
ffmpeg -loglevel debug -hwaccel ascend -c:v h264_ascend -i output.264 -c:v h264_ascend out.264

```
测试n4.4.1两个命令都成功执行，但是适配之后n8.0出现第二个命令decode报错，定位到函数签名未对齐
![Pasted image 20251016145829.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251016145829.png)
更改注册解码器内部字段宏正常
![Pasted image 20251016150003.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251016150003.png)
第一个命令两个版本编码输出的h264裸流文件大小一致且md5值一致，日志输出都是600 packets muxed
但是存在一个问题，第二个命令输出的文件n8.0输出文件大小是152M，n4.4.1的为30M，排查发现flush未注册到解码器中(上图已添加)
添加flush编译安装之后问题依旧
```sh
ffmpeg -loglevel debug -f rawvideo -pix_fmt nv12 -s 1280x720 -r 60 \
       -i /dev/zero -c:v h264_ascend -frames:v 600 test.264
    
ffmpeg -loglevel debug -hwaccel ascend -c:v h264_ascend -i output.264 -f null -

ffmpeg -loglevel debug -hwaccel ascend -c:v h264_ascend -i output.264 -c:v h264_ascend out.264


```
测试发现都用n8.0的不指定输出到文件，600 packets muxed对的上，但是指定输出文件之后就是8000 packets muxed, 且日志中存在相对于n4.4.1或者不指定输出文件多出来的关键字"12 dup!"，搜索关键字看到
是video_sync_process中根据pts进行补充帧的函数，
-f null 模式不建立输出 filter，也不进行 vsync 计算

Demuxer → Decoder → FilterGraph →  **OutputFilter → Encoder** → Muxer

```sh
ffmpeg -hwaccel ascend -c:v h264_ascend -i input.264 \
  -c:v h264_ascend -fps_mode passthrough out.264

```
passthrought测试正常，说明问题可以确定是video_sync_process函数
本地播放文件内容是正常的

帧同步补帧通常和pts等字段有关, 新版本frame少了一些字段，为了适配进行了删除，需要分析影响
```sh
get_vdec_frame_info

```

针对帧同步需要分析n4.4.1和n8.0版本相关函数
n4.4.1对应关键函数为`do_video_out`
n8.0为`video_sync_process`
而关键的字段就是duration

```cardlink
url: https://www.hiascend.com/document/detail/zh/canncommercial/82RC1/API/ispapi/ispdevapi_0032.html
title: "hi_video_frame_info-公共数据类型-ISP系统控制及3A算法注册-ISP图像调优接口-CANN商用版8.2.RC1开发文档-昇腾社区"
description: "<!DOCTYPE html> hi_video_frame_info 说明定义视频图像帧信息结构体。 定义typedef struct { hi_video_frame v_frame; hi_u32 pool_id; hi_mod_id mod_id; } hi_video_frame_info; 成员 成员名称 描述 v_frame 视频图像帧属性结构体。 pool_id 视频缓存池ID，预"
host: www.hiascend.com
favicon: https://www.hiascend.com/_static3/favicon.ico
image: https://www.hiascend.com/_static3/logo.BmJjMz5z.png
```

cann中hi_video_fram中无duration，但是新版本video_sync_process依赖duration

![Pasted image 20251017163738.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251017163738.png)
在himpi_get_frame设置frame->duration=0之后问题解决?!
### ffmpeg OPAQUE_REF 编解码支持解析

查看ffmpeg n8.0的libavcodec/nvenc_h264.c编码注册代码
![Pasted image 20251013172853.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251013172853.png)
可以看到编码器对应的支持的特性通过`.p.capabilities`进行指定


OPAQUE_REF主要依靠ffmpeg框架侧能力，设备侧只需要保证这块私有数据引用能正常传递
该字段主要是解决h264编解码DTS(Decoding TimeStamp解码顺序)和PTS(Presentation TimeStamp显示顺序)顺序不一致, 

| 编码顺序 | 解码顺序 | 显示顺序 |
| ---- | ---- | ---- |
| I0   | I0   | I0   |
| B1   | P3   | B1   |
| B2   | B1   | B2   |
| P3   | B2   | P3   |

opaque ref似乎基本都是框架侧的工作，直接基于当前程序进行测试
原来程序中默认是`YUV420P` to `BGR24`加速, 直接跑会出现`No accelerated colorspace conversion found from yuv420p to bgr24`，这个可能和解码器注册ff_h264_ascend_decoder CODEC_PIXFMTS_ARRAY制定的几个格式不符合，ascend解码后默认是`AV_PIX_FMT_NV12`，程序中改为`AV_PIX_FMT_NV12`,之后没有相关提示，
但是出现`av_hwframe_transfer_data失败`
