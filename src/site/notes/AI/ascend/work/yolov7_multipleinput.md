---
{"dg-publish":true,"permalink":"/AI/ascend/work/yolov7_multipleinput/","noteIcon":"3"}
---

代码逻辑分析:
https://gitee.com/xu-xiaolong28/samples/tree/master/inference/modelInference/sampleYOLOV7MultiInput
![Pasted image 20240827154706.png](/img/user/AI/ascend/work/attachments/Pasted%20image%2020240827154706.png)

- 管理线程：将线程和队列打包在一起，并完成进程创建、消息队列创建、消息发送和消息接收守护。
- 数据输入线程：对输入图片或视频进行解码。
- 数据预处理线程：对数据输入线程传过来的YUV图片进行处理（resize等操作）。
- 推理线程：使用YOLOV7模型进行推理。
- 数据后处理线程：分析推理结果，输出框点及标签信息。
- 数据输出线程：将框点及标签信息标识到输出数据上。

配置文件参考：
```json
{
    "device_config":[
        {
            "device_id":0,
            "model_config":[
                {
                    "infer_thread_name":"infer_thread_0",
                    "model_path":"../model/yolov7x.om",
                    "model_width":640,
                    "model_heigth":640,
                    "model_batch": 1,
                    "postnum": 4,
                    "io_info":[
                        {
                            "input_path":"../data/car0.mp4",
                            "input_type":"video",
                            "output_path":"../out/output",
                            "output_type":"pic",
                            "channel_id":0
                        },

                        {
                            "input_path":"../data/car0.mp4",
                            "input_type":"video",
                            "output_path":"../out/output",
                            "output_type":"pic",
                            "channel_id":1
                        },

                        {
                            "input_path":"../data/car0.mp4",
                            "input_type":"video",
                            "output_path":"../out/output",
                            "output_type":"pic",
                            "channel_id":2
                        },

                        {
                            "input_path":"../data/car0.mp4",
                            "input_type":"video",
                            "output_path":"../out/output",
                            "output_type":"pic",
                            "channel_id":3
                        }
                    ]
                }
            ]
        }
    ]
}

```

### 应用拉起

管理线程函数体持续从自己队列里面取消息，获取到消息之后调用对应的工作线程进行process工作，也就是说队列是属于管理线程的，工作线程从管理线程取消息，往管理线程发消息
![Pasted image 20240827164641.png](/img/user/AI/ascend/work/attachments/Pasted%20image%2020240827164641.png)

为啥线程能持续轮转直至输入被推理结束就在于datainput线程会判断当前帧是不是最后一帧，如果不是的话它会一直往自己的管理线程抛message让一整套流水线不会停止
![Pasted image 20240827164450.png](/img/user/AI/ascend/work/attachments/Pasted%20image%2020240827164450.png)

推理线程为配置文件中model_config的长度，一般一个om文件一个推理线程，数据输入、数据预处理、数据输出线程为model_config.io_info长度，即几路视频几组线程，**数据后处理线程和数据数据输入线程则是model_config.postnum:1的关系**

![Pasted image 20240827161439.png](/img/user/AI/ascend/work/attachments/Pasted%20image%2020240827161439.png)

线程之间关系由表示线程类别和channelId组合的字符串进行串联起来
![Pasted image 20240827161657.png](/img/user/AI/ascend/work/attachments/Pasted%20image%2020240827161657.png)

配置文件中的model_batch配置一次datainput线程读取的帧数
![Pasted image 20240827162514.png](/img/user/AI/ascend/work/attachments/Pasted%20image%2020240827162514.png)








- [ ] 1. 配置文件新增多路视屏汇聚模式
- [ ] 2.多路视频逐帧拼接进行推理或者多路视频逐帧分开推理之后结果拼接输出(后一种方法是否并行度更高)
- [ ] 3.


拼接示例代码:
```py
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    // 读取两张图片
    cv::Mat img1 = cv::imread("image1.jpg");
    cv::Mat img2 = cv::imread("image2.jpg");

    // 检查图片是否成功加载
    if (img1.empty() || img2.empty()) {
        std::cout << "Could not open or find the images!" << std::endl;
        return -1;
    }

    // 调整大小确保两张图片的高度或宽度一致（根据拼接方式）
    if (img1.size() != img2.size()) {
        cv::resize(img2, img2, img1.size());
    }

    // 水平拼接
    cv::Mat h_concat;
    cv::hconcat(img1, img2, h_concat);

    // 垂直拼接
    cv::Mat v_concat;
    cv::vconcat(img1, img2, v_concat);

    // 显示拼接后的图片
    cv::imshow("Horizontal Concatenation", h_concat);
    cv::imshow("Vertical Concatenation", v_concat);

    // 保存拼接后的图片
    cv::imwrite("h_concat.jpg", h_concat);
    cv::imwrite("v_concat.jpg", v_concat);

    cv::waitKey(0);
    cv::destroyAllWindows();

    return 0;
}

```