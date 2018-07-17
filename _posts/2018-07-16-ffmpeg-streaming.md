---
layout: default
title: "Streaming from webcam using ffmpeg"
date:  2018-07-16 22:20:00 -0600
---
## Streaming from webcam using ffmpeg, HLS and hardware h264 encoding.

Streaming from a USB webcam, compress it using hardware h264, and serve the stream as HLS (HTTP Live Streaming).

The webcam is accessed as standard Linux V4L device. Hardware encoding is done by Intel GPU supports QuickSync. ffmpeg can access the hw encoder using VAAPI interface (provided by libva, which needs to installed first). I'm using a Intel N2807 processor in the test.

## ffmpeg version
The current release of ffmpeg 3.2.2 has serious memory leak when using vaapi interface. This was fixed in commit 221ffca6314ed3ba9d38464ea50cd85251c04e74. In the test, I rebuilt ffmpeg from master branch to include the fix.

ffmpeg repository is hosted at [https://github.com/FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg). The revision I'm building is fd010406c03923065ff9b835472f3f174e1c722d

## Configure ffmpeg:
./configure --enable-shared  --optflags=-O2  --disable-static --enable-avfilter --enable-avresample --disable-stripping --disable-indev=oss --disable-indev=jack --disable-outdev=oss  --enable-bzlib --disable-runtime-cpudetect --disable-debug --disable-gcrypt --disable-gnutls --disable-gmp --enable-gpl --enable-hardcoded-tables --enable-iconv --disable-lzma --enable-network --disable-openssl --enable-postproc --disable-libsmbclient --disable-ffplay --disable-sdl2 --enable-vaapi --disable-vdpau --disable-xlib --disable-libxcb --disable-libxcb-shm --disable-libxcb-xfixes --enable-zlib --disable-libcdio --disable-libiec61883 --disable-libdc1394 --disable-libcaca --disable-openal --disable-opengl --disable-libv4l2 --disable-libpulse --disable-libopencore-amrwb --disable-libopencore-amrnb --disable-libfdk-aac --disable-libopenjpeg --disable-libbluray --disable-libcelt --disable-libgme --disable-libgsm --disable-mmal --disable-libmodplug --disable-libopus --disable-libilbc --disable-librtmp --disable-libssh --disable-libschroedinger --disable-libspeex --disable-libvorbis --disable-libvpx --disable-libzvbi --disable-libbs2b --disable-chromaprint --disable-libflite --disable-frei0r --disable-libfribidi --disable-fontconfig --disable-ladspa --disable-libass --enable-libfreetype --disable-librubberband --disable-libzimg --disable-libsoxr --enable-pthreads --disable-libvo-amrwbenc --disable-libmp3lame --disable-libkvazaar --disable-nvenc --disable-libopenh264 --disable-libsnappy --disable-libtheora --disable-libtwolame --disable-libwavpack --disable-libwebp --enable-libx264 --disable-libx265 --disable-libxvid --disable-x11grab --disable-amd3dnow --disable-amd3dnowext --disable-aesni --disable-avx --disable-avx2 --disable-fma3 --disable-fma4 --disable-xop --cpu=host --disable-doc --disable-htmlpages --enable-manpages

## Streaming

LD\_LIBRARY\_PATH=/usr/local/lib /usr/local/bin/ffmpeg -f alsa -i hw:0 -vaapi\_device /dev/dri/renderD128  -f v4l2 -s 800x600  -r 25 -i /dev/video0  -vf 'format=nv12,hwupload' -c:v h264\_vaapi -r 25 -hls\_time 10 output.m3u8

I specified LD\_LIBRARY\_PATH and ffmpeg path, simply because I installed them to /usr/local, and it's not in my standard search path.

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

