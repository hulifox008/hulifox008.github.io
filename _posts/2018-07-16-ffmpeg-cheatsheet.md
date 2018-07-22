---
layout: default
title: "FFmpeg cheatsheet"
date:  2018-07-16 22:20:00 -0600
---
## Streaming from webcam using ffmpeg, HLS and hardware h264 encoding.

Streaming from a USB webcam, compress it using hardware h264, and serve the stream as HLS (HTTP Live Streaming).

The webcam is accessed as standard Linux V4L device. Hardware encoding is done by Intel GPU supports QuickSync. ffmpeg can access the hw encoder using VAAPI interface (provided by libva, which needs to installed first). I'm using a Intel N2807 processor in the test.

I'm using ffmpeg 3.3.6. Older versions around 3.2.x seem to have memory leak from VAAPI.

The streaming command:

/usr/local/bin/ffmpeg -f alsa -i hw:0 -vaapi\_device /dev/dri/renderD128  -f v4l2 -s 800x600  -r 25 -i /dev/video0  -vf 'format=nv12,hwupload' -c:v h264\_vaapi -r 25 -hls\_time 10 output.m3u8

This will generate video segments under current directory, and a index file named output.m3u8. Each segment is about 10 seconds. To adjust video quality, "-qp 24" can be added after '-c:v h264\_vaapi'. 24 seems to be a good balance. Smaller number, better quality. The directory contains segments and index file will then be served by a webserver. 

I also created a very simple page to play the streaming on iOS devices:

    <!doctype html>
    <html>
    <video controls>
    <source src="output.m3u8" type='video/mp4'>
    </video>
    </html>

ffmpeg will not delete any old segments. This can fill up the disk space. A simple script can do that:

    while true; do find . -name "*.ts" -mmin 5 | xargs rm; sleep 300; done

This will delete segments old than 5 minutes.

