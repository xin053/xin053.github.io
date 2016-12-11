---
title: pydub音频处理库使用详解
date: 2016-11-05 14:28:34
categories:
- Python模块学习
tags:
- Python
- pydub
---

## pydub简介

pydub是基于[ffmpeg](https://ffmpeg.org/),有关ffmpeg的介绍，可以看下[百度百科](http://baike.baidu.com/link?url=78zpGld0iPuvNeATHUpY1ug3h0RkskDgIkuwO1MR_jFBjy2ycV0C77uxcSeAhYJ2gINKNwrYZ5bBPfYW5D5-jK),windows下安装完ffmpeg之后配置其`bin`目录到`PATH`

## pydub使用

```python
from pydub import AudioSegment

song = AudioSegment.from_wav("never_gonna_give_you_up.wav")
song = AudioSegment.from_mp3("never_gonna_give_you_up.mp3")
ogg_version = AudioSegment.from_ogg("never_gonna_give_you_up.ogg")
flv_version = AudioSegment.from_flv("never_gonna_give_you_up.flv")

mp4_version = AudioSegment.from_file("never_gonna_give_you_up.mp4", "mp4")
wma_version = AudioSegment.from_file("never_gonna_give_you_up.wma", "wma")
aac_version = AudioSegment.from_file("never_gonna_give_you_up.aiff", "aac")
```

通过以上方法可以从不同格式的音频和视频文件中获取`AudioSegment`对象,进而对其进行一系列处理

<!-- more -->

### Slice audio

```python
# pydub does things in milliseconds
ten_seconds = 10 * 1000

first_10_seconds = song[:ten_seconds]

last_5_seconds = song[-5000:]
```

###  Make the beginning louder and the end quieter

```python
# boost volume by 6dB
beginning = first_10_seconds + 6

# reduce volume by 3dB
end = last_5_seconds - 3
```

### Concatenate audio (add one file to the end of another)

```python
without_the_middle = beginning + end
```

### AudioSegments are immutable

```python
# song is not modified
backwards = song.reverse()
```

将获取一个新的`AudioSegment`对象

### Save the results (again whatever ffmpeg supports)

```python
awesome.export("mashup.mp3", format="mp3")
awesome.export("mashup.mp3", format="mp3", tags={'artist': 'Various artists', 'album': 'Best of 2011', 'comments': 'This album is awesome!'})
awesome.export("mashup.mp3", format="mp3", bitrate="192k")
```

## 例子

### 将目录下的所有mp4文件和flv文件转换为mp3

```python
import os
import glob
from pydub import AudioSegment

video_dir = '/home/johndoe/downloaded_videos/'  # Path where the videos are located
extension_list = ('*.mp4', '*.flv')

os.chdir(video_dir) # change the workplace
for extension in extension_list:
    for video in glob.glob(extension):
        mp3_filename = os.path.splitext(os.path.basename(video))[0] + '.mp3'
        AudioSegment.from_file(video).export(mp3_filename, format='mp3')
```

## 参考文档

- [pydub官方文档](https://github.com/jiaaro/pydub)
- [pydub API Documentation](https://github.com/jiaaro/pydub/blob/master/API.markdown)

