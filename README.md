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
[Doc](https://ottverse.com/crossfade-between-videos-ffmpeg-xfade-filter/) this has a list of available transitions. 

### Crossfade

#### Fade to Black and back in
I was able to make a crossfade transition with the following command, "borrowed" from [Stack Overflow](https://video.stackexchange.com/questions/17502/concat-two-video-files-with-fade-effect-using-ffmpeg)

```
ffmpeg -i output.mp4 -i preview_video_target.mp4 -filter_complex "color=black:1920x1080:d=video1Duration+video2Duration-filterDuration[base]; [0:v]setpts=PTS-STARTPTS[v0]; [1:v]format=yuva420p,fade=in:st=0:d=filterDuration:alpha=1, setpts=PTS-STARTPTS+((video1Duration-filterDuration)/TB)[v1]; [base][v0]overlay[tmp]; [tmp][v1]overlay,format=yuv420p[fv]; [0:a][1:a]acrossfade=d=filterDuration[fa]" -map "[fv]" -map "[fa]" crossfade.mp4
```

where ```video1Duration```, ```video2Duration``` and ```filterDuration```  are just placeholders for your actual values. ```1920x1080``` can be changed as well. This managed to keep the two audio streams as well, so that's nice to know. 

update: turns out the video and the audio have desynced with that command, so that's no fun

TODO: try opengl transitions, and try to understand the acrossfade filter on audio. This can be done and when I figure it out it's gonna be awesome.

This works
```
ffmpeg -i Clip1API.mp4 -i LectureSlides2.mp4 -filter_complex "color=black:1280x720:d=46.84[base]; [0:v]setpts=PTS-STARTPTS[v0]; [1:v]format=yuva420p,fade=in:st=0:d=1:alpha=1, setpts=PTS-STARTPTS+((14.40)/TB)[v1]; [base][v0]overlay[tmp]; [tmp][v1]overlay,format=yuv420p[fv]; [0:a][1:a]acrossfade=d=0.5[fa]" -map "[fv]" -map "[fa]" crossfade_mixed_video_v2.mp4
```
Interesting to note that the audio cross fade is 0.5 whereas the video fade is 1. 

This format also worked here:
```
ffmpeg -i Clip1API.mp4 -i LeectureSlides2.mov -filter_complex "color=black:1280x720:d=45.67[base]; [0:v]setpts=PTS-STARTPTS[v0]; [1:v]format=yuva420p,fade=in:st=0:d=1:alpha=1, setpts=PTS-STARTPTS+((14.40)/TB)[v1]; [base][v0]overlay[tmp]; [tmp][v1]overlay,format=yuv420p[fv]; [0:a][1:a]acrossfade=d=0.5[fa]" -map "[fv]" -map "[fa]" crossfade_mixed_video_v1.mp4
```
This is all very interesting. 


Interesting case that deviates from the norm:
```
ffmpeg -i LectureSlides1.mov -i LectureSlides2.mov -filter_complex "color=black:1280x720:d=26.6[base]; [0:v]setpts=PTS-STARTPTS[v0]; [1:v]format=yuva420p,fade=in:st=0:d=1:alpha=1, setpts=PTS-STARTPTS+((12.67)/TB)[v1]; [base][v0]overlay[tmp]; [tmp][v1]overlay,format=yuv420p[fv]; [0:a][1:a]acrossfade=d=1[fa]" -map "[fv]" -map "[fa]" crossfade_video2_v1.mp4
```
This is intertesting because I accidentally cut the videos wrong and the two files are basically identical. In my new structure the correct form is ```LeectureSlides2.mov```

Even worse is that:
```
ffmpeg -i LectureSlides1.mov -i LeectureSlides2.mov -filter_complex "color=black:1280x720:d=43.94[base]; [0:v]setpts=PTS-STARTPTS[v0]; [1:v]format=yuva420p,fade=in:st=0:d=1:alpha=1, setpts=PTS-STARTPTS+((12.67)/TB)[v1]; [base][v0]overlay[tmp]; [tmp][v1]overlay,format=yuv420p[fv]; [0:a][1:a]acrossfade=d=1[fa]" -map "[fv]" -map "[fa]" crossfade_video2_v1.mp4
``` 
works as well. 

```
ffmpeg -i Clip1API.mp4 -i LectureSlides2.mp4 -filter_complex "color=black:1280x720:d=46.84[base]; [0:v]setpts=PTS-STARTPTS[v0]; [1:v]format=yuva420p,fade=in:st=0:d=1:alpha=1, setpts=PTS-STARTPTS+((14.40)/TB)[v1]; [base][v0]overlay[tmp]; [tmp][v1]overlay,format=yuv420p[fv]; [0:a][1:a]acrossfade=d=0.5[fa]" -map "[fv]" -map "[fa]" crossfade_mixed_video_v3.mp4
```
Works too, between unrelated videos. I'll put further investigation into a TODO because today I've watched way too many different clips of the same stuff just ever so slightly different.

ALl the above cross fades are made by creating a base black panel underneath the two clips fade the first out and then the first in

#### Xfade
This is using a command called xfade

This worked but a fluke
```
ffmpeg -i LectureSlides1.mov -i LectureSlides2.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=1:offset=12.67[v];[0:a][1:a]acrossfade=duration=1[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p xfade1.mp4
```
LectureSlides1.mov has a 13.67 duration. Makes sense then that the offset should be one second before the end of the video.

```
ffmpeg -i LectureSlides1.mp4 -i LectureSlides2.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=0.5:offset=12.86[v];[0:a][1:a]acrossfade=duration=1[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p xfade1.mp4
```
This one makes less sense. LectureSlides1.mp4 is 14.36. 12.86 is 1.5 seconds off the end. And the duration is 0.5. All that implies that the audio starts up quicker than in the other one. I have one idea:

- The different voice IDs take different pauses. .mov and.mp4 files have a different voice id I think. The weird thing is that the cross from .mov to .mp4 works but the other one doesn't.
- 

#### Delay in and Delay out
What if we make every clip have one second fade in and fade out?

```
ffmpeg -i Slide1_delay_inout.mp4 -i Slide2_delay_inout.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=1:offset=13.42[v];[0:a][1:a]acrossfade=duration=1[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p xfade1.mp4
```
Slide1_delay_inout.mp4 has duration 14.92. 

Both clips have a one second delay in and out. IN THEORY. THAT's WHAT I INDICATED, but honestly idk if the API read it correctly

The offset is 1.5 sec off the end of the video but the audio only has 1 sec duration.
```
ffmpeg -i Slide2_delay_inout.mp4 -i Slide3_delay_inout.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=1:offset=29.06[v];[0:a][1:a]acrossfade=duration=1[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p xfade1.mp4
```
This also works. Slide3 has a 30.56 duration. No clue why this works.

# Examining Files
You can use ffprobe to examine files. 

Using that I found something interesting with the above mentioned files. I found the following durations:

- Slide1_delay_inout.mp4 video: 14.92
- Slide1_delay_inout.mp4 audio: 14.41
- Slide2_delay_inout.mp4 video: 30.56
- Slide2_delay_inout.mp4 audio: 30.04

The audio streams are consistently 0.5 sec shorter than the video. If all of that shortened time appears at the end of the clip, then that may well explain the weird offset values. If acrossfade lasts 1 sec but the clip end 0.5 before the video does, then acrossfade starts 1.5 sec before the end of the video, which is exactly the value that worked in the examples in [Delay in and Delay out](#delay-in-and-delay-out).



API:
- [ ] Make a no delay in and delay out and check the length (audio is 0.5 sec shorter than the video)
- [ ] Make a no delay out and delay in and check the duration (audio is 0.5 sec shorter than the video!!)

Web interface. 
- [ ] Make a no delay in or out and check the duration
- [ ] Make a longer delay out and check the duration. (This one's interesting. 84.16 sec vs 83.9 - only a 0.26 difference. Note that the web interface uses a different voice. )

Note that the LectureSlidesX.mp4 clips use a different voice ID. Its delay is also internally consistent. 
LectureSlides1.mp4 -> V:14.36 A: 14.09 D: ~0.27
LectureSlides2.mp4 -> V:32.44 A: 32.17 D: ~0.27

According to this we should be able to crossfade the LectureSlides videos with a one second crossfade like so

```
ffmpeg -i LectureSlides1.mp4 -i LectureSlides2.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=1:offset=13.09[v];[0:a][1:a]acrossfade=duration=1[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p xfadetest.mp4
```

where offset values is (14.36-1-0.27) = 13.09

Tried it and its (pretty) spot on. Remains to be seen what happens when you chain them together, whether things add up. 

So it seems the desired format is:
```
ffmpeg -i $input1 -i $input2 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=$transition_duration:offset=($input1.video.length - $transition_duration - ($input1.video.length - $input1.audio.length))[v];[0:a][1:a]acrossfade=duration=$transition_duration[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p xfadetest.mp4
```

where ```$input1```, ```input2``` and ```transition_duration``` are all modifiable parameters. Let's try a different type of transition.

### Slide left

```
ffmpeg -i LectureSlides1.mp4 -i LectureSlides2.mp4 -filter_complex "[0:v][1:v]xfade=transition=slideleft:duration=1:offset=13.09[v];[0:a][1:a]acrossfade=duration=1[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p slidelefttest.mp4
```

Works spectacularly if I do say so myself :\)
Note however, that he started speaking pretty quickly. May be worth it to pad the beginning of clips that have moving transitions. It should work pretty great with clips that have delay at the beginning, like

```
ffmpeg -i Slide1_delay_inout.mp4 -i Slide2_delay_inout.mp4 -filter_complex "[0:v][1:v]xfade=transition=slideleft:duration=1:offset=13.42[v];[0:a][1:a]acrossfade=duration=1[a]" -map "[v]" -map "[a]" -pix_fmt yuv420p slidelefttest.mp4
```

This one look spectacular!!

### Chaining Transitions
In theory we should be 

```
ffmpeg -i Slide1_delay_inout.mp4 -i LeectureSlides2.mov -i Clip1Web.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=1:offset=13.41[x1];[x1][2:v]xfade=transition=fade:duration=1:offset=43.193[x2];[0:a][1:a]acrossfade=d=1[ax1];[ax1][2:a]acrossfade=d=1[ax2]" -map "[x2]" -map "[ax2]" -pix_fmt yuv420p chained_xfade_test.mp4
```
It works. beautiful <3
Original durations:
- Slide1_delay_inpout.mp4 V: 14.92 A: 14.41 D: 0.51
- LeectureSlides2.mov V:31.266 A: 30.783 D: 0.483
- Clip1Web.mp4 not important for this because it's the final video.

The composite clip, denoted by the intermediate variable ```x1``` and ```ax1``` for its audio, has V: (14.92+31.266 - 1) = 45.186 A: (14.41 + 30.783 - 1 [transition duration]) = 44.193 D: 0.993. The offset for the second xfade should thus be 44.193-1 (or in original form with no simplfying 45.186-(45.186-44.193) - 1 ). Seems this equation works perfectly for transitions. The math is relatively simple too. 


NOTE: the pix_fmt yuv420p parameter is necessary for the resulting video to be playable be most players like quicktime. Use it for compatibility purposes. 

## Side by Side
I know how to make a PIP-style effect with a webM clip from the V1 heygen API using ffmpeg. It would be interesting to figure out how to make a side by side effect for videos - slides on one hand, avatar on the other side. An example for this is the initial video that Carlos showed us. 

## Mid-speech cut
Make mid-speech cuts from side by side or pip to avatar only compositions and vice-versa. Use Carlos's video for inspiration.

## Slide transitions without avatar transitions
Again from Carlos's video. This will probably need very detailed caption information which I hope HeyGen can provide (I'm a little scared that their captions on the website are big blocks of text as opposed to detailed line my line)

Darren suggested I just apply a slide left/right transition on two copies of the same clip mid clip just that on one half there's one slide and there's another clip in the second half. Will have to look into it. 

## Include other video sources
This shouldn't be too hard, should just be an extension of the above stuff

## Slide animations
Some professors like their slides to have animations. Unsure of how many but I'm guessing it's something I should support. 