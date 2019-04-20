# Issue: A 1920x1080 video is rendered to 1920x1088 when HEVC codec is seleted.

## Background
MPEG encodes video using 16 x 16 blocks of pixels (or 16 x 32 for interlaced video). 1080 is not a multiple of 16, so the encoding is forced to encode 1080 video using 1088 rows of encoded pixels. The bottom 8 rows of encoded pixels can be black, grey, or filled with other garbage. For a 1080 stream, the sequence headers should say the vertical size is 1080, meaning that the decoder will produce 1080 output and ignore the bottom 8 encoded rows.

There are a few possibilities here:

* The decoder is disregarding the vertical size field in the sequence headers and producing all of the encoded rows in its decoded output, 1088 instead of the 1080 it should be. That's a decoder bug.
* The decoder is decoding correctly, but simply misreporting the vertical size in the video metadata as the encoded size instead of the size indicated in the sequence headers. That's a decoder bug, but just a cosmetic bug with the metadata it reports.
* The encoder is incorrectly setting the vertical size in the sequence headers to 1088 instead of 1080. That's an encoder bug.

So to understand where the problem lies, the encoded video stream needs to be checked with different decoders. `ffmpeg` almost certainly reports the vertical size correctly, so I'd recommend checking with that. If `ffmpeg` says 1088, then the video was encoded incorrectly. If `ffmpeg` says 1080, then the other decoder was decoding incorrectly and/or reporting the vertical size incorrectly.

## My Case
HEVC videos encoded by `NVEnc` may show incorrect resolution readings in File Explorer's file properties. 4K HEVC videos encoded by `NVEnc` will be rendered at 3840x2176 when using `cuvid` as decoder.

`ffmpeg` shows correct resolution readings and DXVA renders the video correctly.

## Conclusion
I'm not sure whether it's `NVEnc`'s fault at encoding time or decoder's bug.