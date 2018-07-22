---
layout: default
title: "Notes & Tips"
date:  2018-07-21 16:00:00 -0600
---
## Use VLC to stream video from Logitech c920 (Hardware h264 compression)

```
cvlc v4l2:///dev/video0:chroma=h264:width=1920:height=1080 --sout '#standard{access=http,mux=ts,dst=0.0.0.0:8080,name=stream,mime=video/ts}' -vvv
```

It looks cvlc (using 2.1.5) can set the pixelformat of webcam to H264. The following command may not necessay:

```
v4l2-ctl -d /dev/video0 -v pixelformat=1
```

Use ffmpeg to capture video, and save to segments:

```
ffmpeg -s 1920x1080  -f v4l2 -vcodec h264 -i /dev/video0 -copyinkf -vcodec copy  -f segment -segment_time 180 %08d.mp4
```

-segment_time is in seconds. 180 gives about 3 minutes per segment.

Streaming using ssh:

```
ssh 192.168.1.80 'ffmpeg -f v4l2 -video_size 800x600  -r 10 -i /dev/video1   -vcodec h264  -crf 25   -f flv pipe:1 ' | mplayer -
```

## Capture RTSP stream from Sannce H960 DVR

Following command uses ffmpeg to capture rtsp stream from Sannce H960 DVR. "channel" is the camera number, stream=0.sdp or 1.sdp to select main stream or sub stream.
```
ffmpeg -i 'rtsp://192.168.2.10:554/user=admin&password=&channel=4&stream=0.sdp?real_stream--rtp-caching=100' -r 25 -codec copy output.mp4}}}[[BR]]
```

Streaming using ssh tunnel:
```
ssh www.diy-fun.org "ffmpeg -i 'rtsp://192.168.2.10:554/user=admin&password=&channel=3&stream=0.sdp?real_stream--rtp-caching=100' -r 25 -codec copy -f mpegts pipe:1 2>/dev/null" | mplayer -
```

## Connect to HC-06 (Bluetooth serial adapter) at Linux command line

The key is to use "rfcomm" to establish serial connection, and "bluetoothctl" to do pair.
 1. start bluetoothd
```
/etc/init.d/bluetoothd start
```
 2. pair with the device.
```
 T500 fox # bluetoothctl 
 [NEW] Controller 00:1F:E2:E4:94:8A T500-0 [default]
 [NEW] Device 20:15:05:05:17:02 HC-06
 [NEW] Device BC:B1:F3:10:B1:63 SAMSUNG-SGH-I777
 [bluetooth]# agent on
 Agent registered
 [bluetooth]# scan on
 Discovery started
 [CHG] Controller 00:1F:E2:E4:94:8A Discovering: yes
 [CHG] Device 20:15:05:05:17:02 RSSI: -58
 [bluetooth]# devices
 Device BC:B1:F3:10:B1:63 SAMSUNG-SGH-I777
 Device 20:15:05:05:17:02 HC-06
 [bluetooth]# pair 20:15:05:05:17:02
 Attempting to pair with 20:15:05:05:17:02
 [CHG] Device 20:15:05:05:17:02 Connected: yes
 Request PIN code
 [agent] Enter PIN code: 1234
 [CHG] Device 20:15:05:05:17:02 Paired: yes
 Pairing successful
 [CHG] Device 20:15:05:05:17:02 Connected: no
```

 3. Establish serial connection:
```
 T500 fox # rfcomm connect rfcomm0 20:15:05:05:17:02
 Connected /dev/rfcomm0 to 20:15:05:05:17:02 on channel 1
 Press CTRL-C for hangup
```

## Disable DNS lookup in sshd to speed up connection
Set "UseDNS" to "no" in sshd_config. This will save couple seconds when trying to make a connection to ssh server, and if the client address cannot be resolved.

## Detect unused functions with GCC
Pass following flags to GCC compiler:
```
-ffunction-sections -fdata-sections
```
Then pass following flags to linker:
```
-Wl,--gc-sections -Wl,--print-gc-sections
```
GCC will then print a list of unused functions and other symbols. These options are not recommended on final build.


## Debugging deadlock of pthread application
Use gdb to print out who owns the mutex:
```
(gdb) bt
#0  0x0000003f83c0ee84 in ?? () from /lib/libpthread.so.0
#1  0x0000003f83c0a757 in ?? () from /lib/libpthread.so.0
#2  0x0000003f83c0a5b6 in pthread_mutex_lock () from /lib/libpthread.so.0
#3  0x0000003f8380115b in dlsym () from /lib/libdl.so.2
#4  0x00007f5f7dc966bb in ?? () from /usr/lib/libodbc.so.2
#5  0x00007f5f7dc95de9 in ?? () from /usr/lib/libodbc.so.2
#6  0x00007f5f7dc5f21e in ?? () from /usr/lib/libodbc.so.2
#7  0x00007f5f7dc619f0 in SQLConnect () from /usr/lib/libodbc.so.2
#8  0x00007f5f7f16a4ab in ODBCConnection::ODBCConnection(DBUrl&) () from /usr/lib/libcdbc.so
#9  0x00007f5f7f167510 in DBDriver::getConnection(DBUrl&) () from /usr/lib/libcdbc.so
#10 0x00007f5f7f3735c1 in Database::Database() () from /usr/lib/libAcsDB.so.0
#11 0x00007f5f7f3736bb in DB_Init() () from /usr/lib/libAcsDB.so.0
#12 0x00007f5f7f37537d in getUserPasswd () from /usr/lib/libAcsDB.so.0
#13 0x0000000000408da5 in ?? ()
#14 0x000000000040f286 in ?? ()
#15 0x0000003f83c081bb in ?? () from /lib/libpthread.so.0
#16 0x0000003f834eb0cd in clone () from /lib/libc.so.6
(gdb) frame 2
#2  0x0000003f83c0a5b6 in pthread_mutex_lock () from /lib/libpthread.so.0
(gdb) p ((pthread_mutex_t*)$rdi)->__data
$2 = {__lock = 2, __count = 1, __owner = 4317, __nusers = 1, __kind = 1, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}}
```

Use "frame" to go back into pthread_mutex_lock(). rdi register is pointing to mutex_t struct. !__owner shows pid 4317 is holding the lock.


## Generate Self-signed certificate using openssl
Generate private key:
```
openssl genrsa -out private.key 1024
```

Generate certificate request:
```
openssl req -new -key private.key  -out cert.csr -subj "/CN=System/OU=DSView/O=Avocent Corp./L=Huntsville/ST=Alabama/C=US"
```

Sign:
```
openssl x509 -req -days 365 -in cert.csr -signkey private.key  -out cert.crt -extfile x509v3_config
```

Convert to pkcs12 format:
```
openssl pkcs12 -export -out cert.p12 -inkey private.key -in cert.crt -certfile cert.crt
```

```
cat x509v3_config
basicConstraints=CA:true,pathlen:0
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
```
