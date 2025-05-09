---
{"dg-publish":true,"permalink":"/Applications/obsidian/obsidian嵌入视频/","noteIcon":"3"}
---

obsidian嵌入视频可以直接通过`![]()`的格式，这种格式本地可以直接render出来对应的视频网页，但是无法同步到obsidian garden网页，因为本地是有内置的相关插件支持的，为了能在网页同样有嵌入视频网页的效果，可以使用通过的iframe标签

以youtube视频为例
### 内嵌图片格式:
![](https://www.youtube.com/watch?v=wjZofJX0v4M)

### iframe格式:
#### youtube视频

<div style="position: relative; width: 100%; max-width: 800px; height: 0; padding-bottom: 56.25%; margin: 0 auto;">
    <iframe 
        src="https://www.youtube.com/embed/wjZofJX0v4M?rel=0&modestbranding=1&autoplay=0&showinfo=0&fs=1&disablekb=1" 
        style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;" 
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
        allowfullscreen>
    </iframe>
</div>

iframe参数详解:
**参数详解**

1. rel=0

• 禁止播放结束时显示推荐视频。

• 仅显示和嵌入的视频内容相关的推荐，而非其他频道内容。

2. modestbranding=1

• 隐藏 YouTube 的 logo，保持页面简洁。

3. autoplay=0

• 禁止自动播放视频，用户手动点击播放。

4. showinfo=0

• 隐藏视频标题和上传者信息，仅保留播放控件。

5. fs=1

• 启用全屏播放按钮（可选，根据需求设置）。

6. disablekb=1

• 禁用键盘快捷键，防止用户误操作。

7. **外部容器样式**

• max-width: 800px; 控制视频宽度，避免在大屏幕上拉伸过宽。

• padding-bottom: 56.25%; 维持 16:9 宽高比。

• margin: 0 auto; 居中显示。

8. border: none;

• 移除 iframe 外框的边框。

#### bilibili视频

<div style="position: relative; width: 100%; max-width: 800px; height: 0; padding-bottom: 56.25%; margin: 0 auto;">
    <iframe 
        src="https://player.bilibili.com/player.html?bvid=BV1yo4y1U7Q3&autoplay=0" 
        style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;" 
        scrolling="no" 
        frameborder="no" 
        allowfullscreen="true">
    </iframe>
</div>