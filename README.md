# SPKMuxer

This is a CLI PHP script, that can help you to mix your audio recording (e.g. dubbing) into a video, in the following way:
 * One channel remains the original, that comes from the input video
 * The another channel will be replaced with your audio
 * No video frames will be recoded, so it is very fast (1hr 1080p video in ~3mins)
 * You can set the original video's in and end points
 * You can shift your audio's beginning relative to the video's in point
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
