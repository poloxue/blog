---
title: "MoviePy 内置特效一览：基于 Python 编辑视频的新境界"
date: 2024-01-10T17:37:57+08:00
draft: true
comment: true
description: "本文是关于 MoviePy 支持的内置特效一览，演示 MoviePy 内置的所有特效。"
---

本文是关于 MoviePy 支持的内置特效一览，演示 MoviePy 内置的所有特效。

### 视频加速减速 - accel_decel 

```python
clip = VideoFileClip("input_video.mp4")
modified_clip = clip.fx(vfx.accel_decel, 2)  # 将视频速度加速两倍
```

### 黑白效果转换 - blackwhite

```python
# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 将视频转换为黑白效果
modified_clip = clip.fx(vfx.blackwhite)

# 保存修改后的视频
modified_clip.write_videofile("blackwhite_output.mp4")
```

### `blink` | 闪烁效果

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 在指定时间段内制造闪烁效果
modified_clip = clip.fx(vfx.blink, 1, 3)  # 在第1秒到第3秒之间进行闪烁效果

# 保存修改后的视频
modified_clip.write_videofile("blink_output.mp4")
```

### `colorx` | 调整强度亮度

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 调整视频的颜色强度和亮度
modified_clip = clip.fx(vfx.colorx, intensity=1.5, lightness=0.5)  # 增加1.5倍的颜色强度，降低0.5倍的亮度

# 保存修改后的视频
modified_clip.write_videofile("colorx_output.mp4")
```

### `even_size` | 按偶数尺寸裁剪视频

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 裁剪视频为偶数尺寸
modified_clip = clip.fx(vfx.even_size)

# 保存修改后的视频
modified_clip.write_videofile("even_size_output.mp4")
```

### `fadein` | 淡入效果

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 在前3秒内应用淡入效果
modified_clip = clip.fx(vfx.fadein, duration=3)

# 保存修改后的视频
modified_clip.write_videofile("fadein_output.mp4")
```

### `fadeout` | 淡出效果

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 在后3秒内应用淡出效果
modified_clip = clip.fx(vfx.fadeout, duration=3)

# 保存修改后的视频
modified_clip.write_videofile("fadeout_output.mp4")
```

### `freeze` | 在某时间点 t 处暂停一段时间

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 在第5秒处暂停视频1秒钟
modified_clip = clip.fx(vfx.freeze, t=5, freeze_duration=1)

# 保存修改后的视频
modified_clip.write_videofile("freeze_output.mp4")
```

### `freeze_region` | 冻结视频片段的一个区域，同时其余部分保持动画状态

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 在第3到5秒期间冻结视频的一个特定区域（x1, y1, x2, y2）
modified_clip = clip.fx(vfx.freeze_region, t_start=3, t_end=5, region=(100, 100, 300, 300))

# 保存修改后的视频
modified_clip.write_videofile("freeze_region_output.mp4")
```

### `gamma_corr` | Gamma 校正

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 进行 Gamma 校正（gamma=1.5）
modified_clip = clip.fx(vfx.gamma_corr, gamma=1.5)

# 保存修改后的视频
modified_clip.write_videofile("gamma_corr_output.mp4")
```

### `headblur` | 模糊视频帧中的移动部分（例如一个头部）

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 对视频中移动的部分应用模糊效果
modified_clip = clip.fx(vfx.headblur, ksize=20)  # 使用大小为20的内核进行模糊处理

# 保存修改后的视频
modified_clip.write_videofile("headblur_output.mp4")
```

### `invert_colors` | 颜色反转

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 反转视频的颜色
modified_clip = clip.fx(vfx.invert_colors)

# 保存修改后的视频
modified_clip.write_videofile("invert_colors_output.mp4")
```

### `loop` | 循环播放视频

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 循环播放视频3次
modified_clip = clip.fx(vfx.loop, n=3)

# 保存修改后的视频
modified_clip.write_videofile("loop_output.mp4")
```

### `lum_contrast` | 视频片段的亮度对比度校正

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 调整视频的亮度和对比度
modified_clip = clip.fx(vfx.lum_contrast, lum=1.5, contrast=1.2)  # 增加1.5倍亮度，增加1.2倍对比度

# 保存修改后的视频
modified_clip.write_videofile("lum_contrast_output.mp4")
```

### `make_loopable` | 使视频片段在自身结尾逐渐淡入，实现无限循环播放

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 使视频片段在结尾逐渐淡入，实现无限循环播放
modified_clip = clip.fx(vfx.make_loopable)

# 保存修改后的视频
modified_clip.write_videofile("make_loopable_output.mp4")
```

### `margin` | 在整个帧的周围绘制外部边距

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 在整个帧周围添加外边距
modified_clip = clip.fx(vfx.margin, left=20, right=20, top=10, bottom=10)

# 保存修改后的视频
modified_clip.write_videofile("margin_output.mp4")
```

### `mask_and` | 将两个蒙版的逻辑 'and'（最小值）合并

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 将两个蒙版的逻辑 'and'（最小值）合并
mask_clip = VideoFileClip("mask_clip.mp4")  # 载入第二个蒙版视频
modified_clip = clip.fx(vfx.mask_and, mask_clip)

# 保存修改后的视频
modified_clip.write_videofile("mask_and_output.mp4")
```

### `mask_or` | 将两个蒙版的逻辑 'or'（最大值）合并

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 将两个蒙版的逻辑 'or'（最大值）合并
mask_clip = VideoFileClip("mask_clip.mp4")  # 载入第二个蒙版视频
modified_clip = clip.fx(vfx.mask_or, mask_clip)

# 保存修改后的视频
modified_clip.write_videofile("mask_or_output.mp4")
```

### `mask_color` | 返回一个包含蒙版的新剪辑，用于指示原始剪辑的给定颜色的透明区域

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 返回一个包含蒙版的新剪辑，用于指示原始剪辑的给定颜色的透明区域
mask_clip = ColorClip(size=clip.size, color=(255, 0, 0))  # 创建一个颜色为红色的蒙版
modified_clip = clip.fx(vfx.mask_color, mask=mask_clip)

# 保存修改后的视频
modified_clip.write_videofile("mask_color_output.mp4")
```

### `mirror_x` | 水平翻转视频

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 水平翻转视频
modified_clip = clip.fx(vfx.mirror_x)

# 保存修改后的视频
modified_clip.write_videofile("mirror_x_output.mp4")
```

### `mirror_y` | 垂直翻转视频

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 垂直翻转视频
modified_clip = clip.fx(vfx.mirror_y)

# 保存修改后的视频
modified_clip.write_videofile("mirror_y_output.mp4")
```

### `painting` | 将任何照片转换为某种绘画效果

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 将视频转换为绘画效果
modified_clip = clip.fx(vfx.painting)

# 保存修改后的视频
modified_clip.write_videofile("painting_output.mp4")
```

### `resize` | 调整视频大小

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 调整视频大小为 (640, 480)
modified_clip = clip.fx(vfx.resize, width=640, height=480)

# 保存修改后的视频
modified_clip.write_videofile("resize_output.mp4")
```

### `rotate` | 旋转视频

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 将视频逆时针旋转90度
modified_clip = clip.fx(vfx.rotate, angle=90)

# 保存修改后的视频
modified_clip.write_videofile("rotate_output.mp4")
```

### `scroll` | 横向或纵向滚动剪辑，例如制作片尾字幕

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 在视频底部添加纵向滚动的片尾字幕
txt_clip = TextClip("End of Video", fontsize=70, color='white')
txt_clip = txt_clip.set_position(('center', 'bottom')).set_duration(clip.duration)
scrolling_clip = CompositeVideoClip([clip, txt_clip])

# 保存修改后的视频
scrolling_clip.write_videofile("scroll_output.mp4")
```

### `supersample` | 用时间 t 的每一帧替换为在时间段 [t-d, t+d] 内取的 nframes 个等间隔帧的平均值，产生运动模糊效果

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 应用运动模糊效果
modified_clip = clip.fx(vfx.supersample, t=5, d=0.5, nframes=10)  # 在第5秒进行运动模糊效果

# 保存修改后的视频
modified_clip.write_videofile("supersample_output.mp4")
```

### `time_mirror` | 倒序播放视频

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 倒序播放视频
modified_clip = clip.fx(vfx.time_mirror)

# 保存修改后的视频
modified_clip.write_videofile("time_mirror_output.mp4")
```

### `time_symmetrize` | 正序+倒序播放视频

```python
from moviepy.editor import *

# 加载视频片段
clip = VideoFileClip("input_video.mp4")

# 正序+倒序播放视频
modified_clip = clip.fx(vfx.time_symmetrize)

# 保存修改后的视频
modified_clip.write_videofile("time_symmetrize_output.mp4")
```

## 视频特效

方法            | 说明
-------------   | --------
accel_decel     | 视频的加速减速
blackwhite      | 黑白效果转换
blink           | 闪烁效果
colorx          | 调整强度亮度
even_size       | 按偶数尺寸裁剪视频
fadein          | 淡入效果
fadeout         | 淡出效果
freeze          | 在某时间点 t 处暂停一段时间
freeze_region   | 冻结视频片段的一个区域，同时其余部分保持动画状态。
gamma_corr      | Gamma 校正
headblur        | 此函数用于模糊视频帧中的移动部分（例如一个头部）
invert_colors   | 返回颜色反转后的视频片段
loop            | 循环播放视频
lum_contrast    | 视频片段的亮度对比度校正
make_loopable   | 使视频片段在自身结尾逐渐淡入，这样可以无限循环播放
margin          | 在整个帧的周围绘制外部边距
mask_and        | 将两个蒙版的逻辑'and'（最小值）合并
mask_or         | 将两个蒙版的逻辑'or'（最大值）合并
mask_color      | 该函数返回一个新的剪辑，其中包含一个蒙版，用于指示原始剪辑的给定颜色的透明区域。
mirror_x        | 水平翻转视频
mirroy_y        | 垂直翻转视频
painting        | 将任何照片转换为某种绘画效果。
resize          | 调整视频大小
rotate          | 旋转视频
scroll          | 横向或纵向滚动剪辑，例如制作片尾字幕
supersample     | 这个功能会用时间t的每一帧替换为在时间段[t-d, t+d]内取的nframes个等间隔帧的平均值。这样可以产生运动模糊效果
time_mirror     | 倒序播放
time_symmetrize | 正序+倒序播放

