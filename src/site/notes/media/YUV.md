---
{"dg-publish":true,"permalink":"/media/YUV/","noteIcon":"3"}
---

- **Y (Luma，亮度分量)**
    
    - 表示图像的明暗程度，相当于黑白照片。
        
    - 范围通常是 [0, 255]（8bit），0=黑，255=白。
        
    
- **U (Blue-difference Chroma，蓝色色度分量)**
    
    - 表示该像素颜色和亮度的差异，偏向蓝色的程度。
        
    - U = B – Y，反映“像素比亮度多了多少蓝色”。
        
    
- **V (Red-difference Chroma，红色色度分量)**
    
    - 表示该像素颜色和亮度的差异，偏向红色的程度。
        
    - V = R – Y，反映“像素比亮度多了多少红色”。


YUV420P
以宽4 pixel高2 pixel的区间为例，
每行4个像素都有一个独立的Y，
每行4个像素共用2个 U/V
垂直方向，第二行没有额外的色度采样，复用第一行的U/V
```
Y 平面 (4x4):
Y0  Y1  Y2  Y3
Y4  Y5  Y6  Y7
Y8  Y9  Y10 Y11
Y12 Y13 Y14 Y15

U 平面 (2x2):
U0  U1
U2  U3

V 平面 (2x2):
V0  V1
V2  V3
```

YUV420SP(NV12)
采样和上面类似，不过内存布局不一样，NV12为半平面Semi-Planar


```
Y 平面 (4x4):
Y0  Y1  Y2  Y3
Y4  Y5  Y6  Y7
Y8  Y9  Y10 Y11
Y12 Y13 Y14 Y15

UV 平面 (2x2, 交错存储):
U0 V0  U1 V1
U2 V2  U3 V3
```