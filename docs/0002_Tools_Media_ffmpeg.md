---
Title | Tools Media ffmpeg
-- | --
Created @ | `2022-08-23T03:37:05Z`
Updated @| `2023-07-22T15:16:53Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/2)

---
# ffmpeg 多媒体工具


## UseCase


### 录制桌面

```
ffmpeg -f x11grab -s 2560x1440 -i :1.0 -r 25 -vcodec libx264 output.mp4
```


### 移除重复帧


```
ffmpeg -i input.mp4 -vf mpdecimate,setpts=N/FRAME_RATE/TB out.mp4
```


### 倍速视频

```
ffmpeg -i input.mp4 -an  -filter:v "setpts=0.25*PTS" output.mp4
```
- 通过修改 PTS(Presentation Time Stamp - 显示时间戳) 实现


### 制作 Gif

```
ffmpeg -i input.mp4 -r framerate output.gif
```

### 剪切子视频


```
ffmpeg -i input.mp4 -ss 00:10:03 -t 00:03:00 -vcodec copy -acodec copy output.mp4
```
- `-ss` 开始时间戳
- `-t` 长度

### 截取图片

```
ffmpeg -i input.mp4 -ss 00:00:30 -vframe 1 ouput.jpg -an -vcodec mjpeg
```


## Reference
- [Remove sequentially duplicate frames when using FFmpeg](https://stackoverflow.com/questions/37088517/remove-sequentially-duplicate-frames-when-using-ffmpeg)
- [ffmpeg音视频加速](https://www.jianshu.com/p/ea4af542df6a)

