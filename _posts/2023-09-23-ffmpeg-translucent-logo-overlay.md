---
layout: post
title:  "Use FFmpeg to overlay a translucent logo"
date:   2023-09-17 21:00:00 -0500
categories: til ffmpeg
---

Source: [*ffmpeg: overlay a png image on a video with custom transparency?*](https://stackoverflow.com/questions/38753739)

```shell
ffmpeg \
 -i input_video.mp4 \
 -i watermark.png \
 -filter_complex "
   [1:v] format=argb,geq=r='r(X,Y)':a='0.3*alpha(X,Y)' [watermark]; 
   [0:v][watermark] overlay=10:10
  " \
 output_video.mp4
```

Explanation (with help from ChatGPT):
* `[1:v]` applies the subsequent format to the **video (v)** channel of the `watermark.png` (index 1) 
* `[watermark]` at the end is a label for the output
* `[0:v][watermark]` overlays `[watermark]` unto the **video** channel of the `input_video` (index 0)
* `format=argb` allows supporting images with no alpha transparency
* `geq=r='r(X,Y)':a='0.3*alpha(X,Y)` *geq (general equation)* applies transparency of `0.3` for each pixel
