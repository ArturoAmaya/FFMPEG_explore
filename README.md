# Intro
Note that I'm not going to upload a single video here because the files are huge and github doesn't play nice with large files or non-text type files (everything changes all the time and the changes are huge)
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

NOTE: Interestingly enough LectureSlides1.mov has identical audio and video stream length and the audio is stream 0 and video is stream 1. A weirdo file, maybe generated by QuickTime player, and not HeyGen. 

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

let's try something basic like

```
ffmpeg -loop 1 -i Lecture10_3.jpg -i LectureSlides2.mov -filter_complex "hstack=shortest=1" -pix_fmt yuv420p side_by_side.mp4
```

This worked great to generate a double wide video clip. Not what we want. Preferably we want:

- Same original dimensions
- Slide size reduced but content uncut

This should be achievable by adding a background and then scaling the video. 

First pass places the slides on top left (as it should). 

```
ffmpeg -f lavfi -i color=c=white:s=1920x1080:r=24 -i LectureSlides1.mov -filter_complex "[0:v][1:v]overlay=shortest=1,format=yuv420p[out]" -map "[out]" -map 1:a color_background.mp4
```

Let's place it on the left but vertically scaled so it's in the middle of the new video:

```
ffmpeg -f lavfi -i color=c=blue:s=1920x1080:r=24 -i LectureSlides1.mov -filter_complex "[0:v][1:v]overlay=10:(main_h-overlay_h)/2:shortest=1,format=yuv420p[out]" -map "[out]" -map 1:a color_background.mp4
```

Let's scale it:
```
ffmpeg -f lavfi -i color=c=blue:s=1920x1080:r=24 -i LectureSlides1.mov -filter_complex "[1:v]scale=w=0.9*iw:h=0.9*ih[scaled_slides];[0:v][scaled_slides]overlay=10:(main_h-overlay_h)/2:shortest=1,format=yuv420p[out]" -map "[out]" -map 1:a color_background.mp4
```
- Avatar Clip cut or zoomed
```
ffmpeg -i Sidebysideclip.mp4 -filter_complex "crop=w=iw/2:h=ih-50:x=325:y=25" crop.mp4

```
- Roughly 2:1 ratio for slide to avatar

Putting it all together we should get something like:

```
ffmpeg -f lavfi -i color=c=blue:s=1920x1080:r=24 -i LectureSlides1.mov -i Sidebysideclip.mp4 -filter_complex "[2:v]crop=w=iw/2:h=0.9*ih:x=325:y=0[cropped_avatar];[1:v]scale=w=0.9*iw:h=0.9*ih[scaled_slides];[0:v][scaled_slides]overlay=10:(main_h-overlay_h)/2:shortest=1[bkgnd_slides];[bkgnd_slides][cropped_avatar]overlay=main_w-overlay_w-20:(main_h-overlay_h)/2:shortest=1[out]" -map "[out]" -map 2:a side_by_side_test.mp4
```

## Mid-speech cut
Make mid-speech cuts from side by side or pip to avatar only compositions and vice-versa. Use Carlos's video for inspiration. 
I generated caption using OpenAI's whisper model per [this guide](https://williamhuster.com/automatically-subtitle-videos/)

With that we can try to do a mid-speech cut. Something like

```
ffmpeg -f lavfi -i color=c=blue:s=1280x720:r=24 -i Lecture10_3.jpg -i "Example Video AI Studio-2.mp4" -filter_complex "[2:v]trim=start=0.0:end=26.0, setpts=PTS-STARTPTS [first_half_avatar];[2:v]trim=start=26.0:end=52.24, setpts=PTS-STARTPTS [second_half_avatar];[first_half_avatar]crop=w=iw/2:h=0.7*ih:x=300:y=0[cropped_first_half_avatar];[cropped_first_half_avatar]scale=w=0.7857*iw:h=0.7857*ih[scaled_first_half_avatar];[1:v]scale=w=0.55*iw:h=0.55*ih[scaled_slides];[0:v][scaled_first_half_avatar]overlay=main_w-overlay_w-10:(main_h-overlay_h)/2:shortest=1[first_half_avatar_and_bkgnd];[first_half_avatar_and_bkgnd][scaled_slides]overlay=10:(main_h-overlay_h)/2:shortest=0[first_half];[first_half][second_half_avatar]concat=n=2:v=1 [out]" -pix_fmt yuv420p -map "[out]" -map 2:a mid_speech_cut.mp4
```

## Slide transitions without avatar transitions
Again from Carlos's video. This will probably need very detailed caption information which I hope HeyGen can provide (I'm a little scared that their captions on the website are big blocks of text as opposed to detailed line by line. Having generated captions with Veed.io I'm less worried. Seems the vatar pauses on periods for a small amount, allowing the clip to transition well). 

What I'm going to do is do the video transition between two slides first then scale and place on background along with the speaker. TODO working example.

## Include other video sources
This shouldn't be too hard, should just be an extension of the above stuff.
TODO working example, but also need some way to get the video. I propose give me video download URL. 

## Slide animations
Some professors like their slides to have animations. Unsure of how many but I'm guessing it's something I should support. This is going to be ridiculously hard. I might ask for a video of the slide transitions with some kind of marker as to when they occur. Building the animations themselves may be a little beyond my abilities. 

# FFProbe
FFprobe can be used to extract information about the starting video files. 

## Video stream duration
```
ffprobe -v error -select_streams v:0 -show_entries stream=duration -of default=noprint_wrappers=1:nokey=1 Slide1_delay_inout.mp4
```

## Audio Stream Duration
```
ffprobe -v error -select_streams a:0 -show_entries stream=duration -of default=noprint_wrappers=1:nokey=1 Slide1_delay_inout.mp4
```

# Chaining commands
Everything I've done so far has been manual. I need to figure out a way to do automatic or programmatic chaining of the commands. There are two ways, of which there is an obvious choice:
- Do each block alone and write to the filesystem, and then combine the blocks and produce an output. PROS: easy to manage labels inside the commands. CONS: slow as ***** since there's so much read/write to the disk.
- Figure out a way to chain them all in the commands. PROs:One command, one run. If everything works it's just one run. CON: so hard to understand once you actually write it out. Might require preprocessing. 


Obvious choice is to do the hard work upfront and figure out a way to concat commands in the filter_complex. I think that will probably look like:
- Parsing the text
- Producing the relevant clips
- Extracting timing information about the generated clips
- Fill in the information to make compositions (i.e. sections of video without transitions)
- Fill in the information to make transitions.
- Put that all into one command. 

I will manage a basic incrementally upgrading script parser. V0.0 can just concatenate clips through the HeyGen API. 

- [ ] V0.01 Only handle full clips. Slides transition at the same time as the avatar. Fades only. Transition to docker. Run all command on CLI, not any wrapper. 