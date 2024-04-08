# Intro

Apparently (according to Raunit), FFMPEG underlies a lot of video manipulation software. We're going to use it to edit videos for lecture generation. To that end there are a few commands we need to learn.

## Picture in picture

```
ffmpeg -loop 1 -i AAH.jpg -i "Example Video AI Studio-2.mp4" -filter_complex "[1]scale=iw/4:ih/4 [new]; [0][new] overlay=main_w-overlay_w-10:main_h-overlay_h-10:shortest=1" output.mp4          
```

So what I understand this is doing is importing ```AAAH.jpg``` and ```Example Video AI Studio-2.mp4```. It's looping ```AAH.jpg``` over once. The complex filter seems to say "grab input video 1 (```Example Video AI Studio-2.mp4```), scale it down to a quarter of the width and height, call it new. Grab overlay new on input 0 such that the x coord is main width - overlay width - 10 and the same thing for y coord. In other words, put it near the bottom right. finally, tell the process that the shortest video is the second one (without this I managed to make a 2 hour video cuz the image loop goes on forever unless capped) put it all in output.mp4

Per some [obscure guide I found](https://www.oodlestechnologies.com/blogs/PICTURE-IN-PICTURE-effect-using-FFMPEG/) you need the video whose audio will be used to be the first input, but this doesn't seem to hold true with my video.

## Stitching
```ffmpeg -f concat -i videos.txt -c copy output_joined.mp4```
Just puts the videos together. 

## Transitions
[Doc](https://ottverse.com/crossfade-between-videos-ffmpeg-xfade-filter/)