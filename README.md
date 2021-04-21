# chromakey

live green screen solution with ffmpeg and v4l2 pseudo device

In order to get this work you need to have a recent kernel with v4l2 loopback device enabled, and ffmpeg 3.4 or greater with chroma key support. Make sure to check which camera device to use:
`v4l2-ctl --list-devices` 

Then load 
```bash
sudo apt install v4l2loopback-utils v4l2loopback-dkms ffmpeg
sudo modprobe v4l2loopback video_nr=2 exclusive_caps=1 card_label="green-screen"
```
The `exclusive_caps=1` makes it work with chrome browser and google conference, shamelessly lifted from [here](https://gist.github.com/miraculixx/49a3a8cad895f8a72b3a6bdc089d0580) Thanks figuring this out!


To check if `mjpg` is supported:
```
v4l2-ctl  --list-formats -d /dev/video3
ioctl: VIDIOC_ENUM_FMT
	Type: Video Capture

	[0]: 'YUYV' (YUYV 4:2:2)
	[1]: 'H264' (H.264, compressed)
	[2]: 'MJPG' (Motion-JPEG, compressed)
```

Download and copy this script into `/usr/local/bin` then execute with


when background image is specified, the behavior is same is before:
```
chromakey -i path_to_background_image.png  -c /dev/video3 -o /dev/video2
```

alternatively you can grab an area of the screen: 
```
chromakey -c /dev/video3 -o /dev/video2 -x_pos=0 y_pos=800 
```

```
steven@io:~/projects/chromakey$ ./chromakey -h

chromakey replaces specified color key with background image of live video
when '--image' is not specified it grabs the screen at '--x_pos' and '--y_pos'
of area specified with '--size'
USAGE: chromakey [arguments] 
ARGS:
  -h, --help             # This help message
  -v, --verbose          # Enable verbose messages
  -k, --key              # specify color key (DEFAULT: green)
  -m, --similarity       # chromakey similarity (DEFAULT: 0.19)
  -b, --blend              # specify color key (DEFAULT: 0.07)
  -s, --size             # video stream size in pixels: 1280x720
  -i, --image            # specify image for background (DEFAULT: )
  -c, --camera           # the real input camera device path (DEFAULT: /dev/video0)
  -o, --output           # v4l2 pseudo device  (DEFAULT: /dev/video1)
  -r, --rate             # frame rate  (DEFAULT: 30)
  -x, --x_pos            # xgrab: x position   (DEFAULT: 0)
  -y, --y_pos            # xgrab: y position  (DEFAULT: 0)

Copyright (c) <Steven Varga 2010-2021> <steven@vargaconsulting.ca, Toronto, On>
```

The live chromakey-ed videostream will be available on specified output device, usually `/dev/video1` and can be checked with ay v4l2 compatible device
here is firefox:
`https://webrtc.github.io/samples/src/content/devices/input-output/`
Skype also known to work.

