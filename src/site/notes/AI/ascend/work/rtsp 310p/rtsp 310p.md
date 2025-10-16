`=yes/1等变量写入到Makefile和头文件,最终导致ascend相关的o文件未合入so文件


针对ascend_enc.c中对齐由原来的AVcodec注册改为对齐新版本的FFcodec，之后跑出现错误
![Pasted image 20251015000506.png](/img/user/AI/ascend/work/rtsp%20310p/attachments/Pasted%20image%2020251015000506.png)
这是由于修改了allcodecs.c文件将ascend相关的编解码器type改为FFcodec之后find_things_extern能识别到并最终产生`