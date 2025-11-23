## MaxCLLFind ##
PQ HDR Analyzer plugin for AVISynth, analyzes MaxCLL and MaxFALL and writes that to a text file after closing the application that is calling AVISynth.
<br>
<br>
The created textfile's name is "MaxCLLFind_Results0.txt". 
<br>
If it already exists, a new file called MaxCLLFind_Results1.txt will be created and the integer will keep increasing at each iteration.
<br>
<br>
**Caution:** This may not be the correct way to calculate MaxFALL and MaxCLL. For (Max)FALL I averaged the nit intensities of every single channel of every pixel in the frame. For MaxCLL I simply took the brightest channel of any pixel in any of the frames. There may be some weighting necessary akin to what was done here: https://github.com/HDRWCG/HDRStaticMetadata


### Usage
```
clip.MaxCLLFind()
```
Load in VirtualDub and click Play. After video finished playing, close VirtualDub. 
<br>
Alternatively, you can use FFMpeg to seek through the AVS Script without actually processing anything.
```
ffmpeg.exe -hide_banner -benchmark -i "AVS Script.avs" -map 0:0? -an -f null out.null
```
<br>
The plugin also writes the Average FALL (frame average light level) into the text file. If you want this result to be accurate, make sure to not load any frame more than once.
<br>
This plugin only accepts RGB inputs. If your HDR clip isn't RGB, convert it first. 
<br>
For example, let's say you are loading a HDR HEVC YUV file, do this:

```
clip = clip.ConvertToRGB64(matrix="Rec2020")
clip.MaxCLLFind()
```

Supported are the packed RGB formats RGB24, RGB32, RGB48, RGB64 and the planar RGB formats RGBP8, RGBP10, RGBP12, RGBP14, RGBP16. It is not advisable to use the formats RGB24, RGB32 and RGBP8 for HDR clips. The planar formats are processed about 30% faster than the packed formats, however the default maxFall calculation is unsuppored for them.

### Alternate MaxFALL algorithm
The default MaxFALL algorithm uses the SMPTE recommendation of averaging max(R,G,B) across all pixels, meaning the brightest channel of each pixel goes into the average. If you want the average of all channels of all pixels (not the official recommendation) instead, do this:
```
clip.MaxCLLFind(maxFallAlgorithm=1)
```
This is more for your own curiosity and might lead to playback problems like flickering if used as actual HDR metadata, since it typically leads to slightly lower average intensity readings and if the TV bases its own dimming on the official recommendation, it might dim the image when it reaches a higher FALL than your calculated MaxFALL, which will almost certainly happen. 
<br>
The official maxFall algorithm is not supported by the planar formats. If your input is planar either disable the maxFall calculation or use the unofficial algorithm or convert your clip to a packed format like RGB48. The maxFall calculation is disabled by:
<br>
```
clip.MaxCLLFind(maxFallAlgorithm=-1)
```

### Results Output
By default, the output values are written in the MaxCLLFind_Results0.txt every 100 frames, which means that those are gonna be reported from the beginning to the end of the clip so that you can see how they're updated as the clip progresses.
<br>
```
MaxCLLFind(maxFallAlgorithm=0, LogIntermediateStats=true)
```
<br>
For instance:
<br>
Stats at frame 1:
<br>
MaxCLL: 0, raw value: 0 0 at X 0 Y 0 at frame 0
<br>
MinCLL: 0, raw value: 0 0 at X 0 Y 2159 at frame 0
<br>
MaxFALL: 0 at frame 0
<br>
FALL Average: 0 across 1 frames.
<br>
<br>
Stats at frame 101:
<br>
MaxCLL: 400.011, raw value: 42767 0.652583 at X 0 Y 1101 at frame 7
<br>
MinCLL: 0, raw value: 0 0 at X 0 Y 2159 at frame 0
<br>
MaxFALL: 8.55332 at frame 41
<br>
FALL Average: 4.07527 across 101 frames.
<br>
<br>
Stats at frame 201:
<br>
MaxCLL: 400.011, raw value: 42767 0.652583 at X 0 Y 1101 at frame 7
<br>
MinCLL: 0, raw value: 0 0 at X 0 Y 2159 at frame 0
<br>
MaxFALL: 8.55332 at frame 41
<br>
FALL Average: 2.84126 across 201 frames.
<br>
<br>
If you're only interested in the final values, set LogIntermediateStats to false when calling the function.
<br>

```
MaxCLLFind(maxFallAlgorithm=0, LogIntermediateStats=false)
```

<br>
and only the last value will be reported once the processing ends.
<br>
Stats at frame 201:
<br>
MaxCLL: 400.011, raw value: 42767 0.652583 at X 0 Y 1101 at frame 7
<br>
MinCLL: 0, raw value: 0 0 at X 0 Y 2159 at frame 0
<br>
MaxFALL: 8.55332 at frame 41
<br>
FALL Average: 2.84126 across 201 frames.
<br>

### Contribute

If you feel like improving the code, refactoring or cleaning up, feel free.


## TODO

- Add functionality to import cutlist for dynamic scene-based metadata. I'm not sure how this would be implemented in an encode, but I read that this possibility exists, so it would be nice to have.

## Changelog
*2025-11-24 - Added option to return just the last output without any intermediate values.*

*2019-12-30 - Fixed MaxFALL Algorithm to be based on the official SMPTE recommendation. This algorithm computes the frame average brightness based on the average of the brightest channel of each pixel, or in other words, max(R,G,B). The old algorithm was using the average of all channels of all pixels, leading to slightly lower resulting values in the tests I did. The old algorithm can be still optionally used via the parameter maxFallAlgorithm=1*
