# SPKMuxer

This is a CLI PHP script, that can help you to mix your audio recording (e.g. dubbing) into a video, in the following way:
 * One channel remains the original, that comes from the input video
 * The another channel will be replaced with your audio
 * No video frames will be recoded, so it is very fast (1hr 1080p video in ~3mins)
 * You can set the original video's in and end points
 * You can shift your audio's beginning relative to the video's in point
 * You can set the in-point of the audio (so basically it can cut off the from the beginning of your audio)
 * You can change the volume's of the original and the added audio too

```
$ ./Spkmuxer --help

    --video         Input video with audio included
    --audio         The new audio that should replace one channel
    --vcutin        The in-point of the video (in seconds or hh:mm:ss)

    --vcutout       The out-point of the video (hh:mm:ss)
    OR
    --vduration     The length of the input video (in seconds or hh:mm:ss)

    --audio-delay   The delay of the added audio relative to the video (hh:mm:ss or in millisecs, can not be 0)
    --audio-seek    The in-point of the added audio (in seconds or hh:mm:ss)

    --volume-orig   Original video's volume (float: 0: mute, <1 attenuate, ==1: original,  >1: louder)
    --volume-added  Added audio's volume (float: 0: mute, <1 attenuate, ==1: original,  >1: louder)

    --help I never understood why they write --help into --help.
    
    
                    
                       <---------------+ vcutout +-------------->+
                                                                 |
                                             <---+ vduration +-->+
                                                                 |
                       +---------------------+-------------------------------+
                                             |                   |           |   ^
                                             |                   |           |   |
                video  <-----+ vcutin +----->+                   |           |   | volume-orig
                                             |                   |           |   |
                                             |                   |           |   +
                       +---------------------+-------------------------------+
                                                                 |
                                                                 |
                               +-------------------+-------------------------+
                                                   |             |           |   ^
                       <-------+ audio-delay +---->+             |           |   |
                audio                              |             |           |   | volume-added
                               <--+ audio-seek +-->+             |           |   |
                                                   |             |           |   +
                               +-------------------+-------------------------+
                                                                 |
                                                                 |
                                                                 +

```

# Example
```
$ php Spkmuxer --video video.mp4 --audio new_audio.wav --vcutin 00:00:05 --audio-delay 2300          
Command: 
----------------------------
time ffmpeg -y \
 -ss '0' \
 -i 'new_audio.wav' \
 -i 'video.mp4' \
 -filter_complex "
 [0:a]aformat=sample_fmts=fltp:sample_rates=44100:channel_layouts=stereo,volume=1,adelay=2300|2300[a1];
 [1:a]aformat=sample_fmts=fltp:sample_rates=44100:channel_layouts=stereo,volume=1[a2];
 [a1][a2]amerge,pan=stereo|c0<c0+c1|c1<c2+c3" \
  -c:v copy \
 -ss '00:00:05' \
 'video_muxed.mp4'
----------------------------

ffmpeg version 3.2.2 Copyright (c) 2000-2016 the FFmpeg developers
  built with gcc 5.4.0 (Ubuntu 5.4.0-6ubuntu1~16.04.4) 20160609
  configuration: --prefix=/data/opt/own/ffmpeg --pkg-config-flags=--static --extra-cflags=-I/data/opt/own/ffmpeg/include --extra-ldflags=-L/data/opt/own/ffmpeg/lib --bindir=/data/opt/own/bin --enable-gpl --enable-libass --enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-nonfree
  libavutil      55. 34.100 / 55. 34.100
  libavcodec     57. 64.101 / 57. 64.101
  libavformat    57. 56.100 / 57. 56.100
  libavdevice    57.  1.100 / 57.  1.100
  libavfilter     6. 65.100 /  6. 65.100
  libswscale      4.  2.100 /  4.  2.100
  libswresample   2.  3.100 /  2.  3.100
  libpostproc    54.  1.100 / 54.  1.100
Guessed Channel Layout for Input Stream #0.0 : mono
Input #0, wav, from 'new_audio.wav':
  Duration: 00:01:50.36, bitrate: 768 kb/s
    Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 48000 Hz, mono, s16, 768 kb/s
Input #1, mov,mp4,m4a,3gp,3g2,mj2, from 'video.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Lavf57.56.100
    location-eng    : +47.5249+019.1491/
    location        : +47.5249+019.1491/
  Duration: 00:00:23.33, start: 0.000000, bitrate: 1884 kb/s
    Stream #1:0(eng): Video: h264 (High 4:2:2 Intra) (avc1 / 0x31637661), yuv422p, 960x540 [SAR 1:1 DAR 16:9], 1561 kb/s, 30 fps, 30 tbr, 15360 tbn, 60 tbc (default)
    Metadata:
      handler_name    : VideoHandler
    Stream #1:1(eng): Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 319 kb/s (default)
    Metadata:
      handler_name    : SoundHandler
[Parsed_amerge_5 @ 0x43b08c0] No channel layout for input 1
[Parsed_amerge_5 @ 0x43b08c0] Input channel layouts overlap: output layout will be determined by the number of distinct input channels
Output #0, mp4, to 'video_muxed.mp4':
  Metadata:
    encoder         : Lavf57.56.100
    Stream #0:0: Audio: aac (LC) ([64][0][0][0] / 0x0040), 44100 Hz, stereo, fltp, 128 kb/s (default)
    Metadata:
      encoder         : Lavc57.64.101 aac
    Stream #0:1(eng): Video: h264 (High 4:2:2 Intra) ([33][0][0][0] / 0x0021), yuv422p, 960x540 [SAR 1:1 DAR 16:9], q=2-31, 1561 kb/s, 30 fps, 30 tbr, 15360 tbn, 15360 tbc (default)
    Metadata:
      handler_name    : VideoHandler
Stream mapping:
  Stream #0:0 (pcm_s16le) -> aformat
  Stream #1:1 (aac) -> aformat
  pan -> Stream #0:0 (aac)
  Stream #1:0 -> #0:1 (copy)
Press [q] to stop, [?] for help
frame=  548 fps=493 q=-1.0 Lsize=    3883kB time=00:00:18.29 bitrate=1738.5kbits/s speed=16.5x    
video:3580kB audio:286kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.423141%
[aac @ 0x43ac5e0] Qavg: 282.957
1.14user 0.02system 0:01.18elapsed 98%CPU (0avgtext+0avgdata 27184maxresident)k
88inputs+7768outputs (0major+2460minor)pagefaults 0swaps

```
