![Aptdec logo](textlogo.png)

Copyright (c) 2004-2009 Thierry Leconte (F4DWV), Xerbo (xerbo@protonmail.com) 2019-2020

## Description

Aptdec is an FOSS program that decodes images transmitted by NOAA weather satellites. These satellites transmit continuously (among other things), medium resolution (1px/4km) images of the earth on 137 MHz.  
These transmissions could be easily received with an simple antenna and cheap SDR. Then the transmission can easily be decoded in narrow band FM mode.

Aptdec can convert these audio files into `png` images.

For each audio file up to 6 images can be generated:

1. Raw image: contains the 2 transmitted channel images + telemetry and synchronization pulses.
2. Calibrated channel A image
3. Calibrated channel B image
4. Temperature compensated IR image
5. False color image
6. MCIR (Map Color InfraRed) image

The input audio file must be mono with a sample rate in the range of 4160-62400 Hz, lower samples rates will process faster.
Aptdec uses `libsndfile` to read the input audio, so any format supported by `libsndfile` may be used (however it has only tested with `.wav` files).

## Compilation

Aptdec is portable since it is written in standard C.
It has successfully compiled and ran on Debian with both `gcc`, `clang` and `tcc` and will most likely work on any Unix platform.
Just edit the Makefile and run `make` (no configure script as of right now). 

Aptdec uses `libsndfile`, `libpng` and `libm`.
The `snd.h` and `png.h` headers must be present on your system.
If they are not on standard path, edit the include path in the Makefile.

## Usage

To compile  
`make`

To run without installing  
`./aptdec [options] audio files...`

To install  
`sudo make install`

To run once installed  
`aptdec [options] audio files...`

To uninstall  
`sudo make uninstall`

## Options

```
-i [r|a|b|c|t|m]
Output image type
Raw (r), Channel A (a), Channel B (b), False Color (c), Temperature (t) or MCIR (m)
Default: "ab"

-d <dir>
Images destination directory (optional)
Default: Current directory

-s [15|16|17|18|19]
Satellite number
For temperature calibration
Default: "19"

-e [r|a|b|c|t|m]
Effects
Histogram equalise (h), Crop Telemetry (t), Denoise (d), Precipitation (p) or Linear equalise (l)
Defaults: off

-m <file>
Map file generated by wxmap

-o <filename>
Output image filename

-r
Realtime decode. When decoding in realtime it is highly recommended to choose a plain raw image.
```

## Output

Generated images are outputted in PNG and are 24 bit RGB for all image types apart from pure greyscale images.

Image names are `audiofile-x.png`, where `x` is:

 - `r` for raw images
 - Sensor ID (`1`, `2`, `3A`, `3B`, `4`, `5`) for channel A|B images
 - `c` for false color
 - `t` for temperature calibrated images
 - `m` for MCIR images

Currently there are 6 available effects:

 - `t` for crop telemetry, off by default, only has effects on raw images
 - `h` for histogram equalise, stretch the colors in the image to black and white
 - `d` for a median denoise filter
 - `p` for a precipitation overlay
 - `f` to flip the image (for southbound passes)
 - `l` to linearly equalise the image (recommended for falsecolor images)

## Examples

`aptdec -d images -i ab *.wav`

This will process all `.wav` files in the current directory, generate calibrated channel A and B images and put them in the `images` directory.

`aptdec -e dh -i b audio.wav`

Decode `audio.wav` with denoise and histogram equalization and save it into the current directory.

## Realtime decoding

As of recently a realtime output was added allowing realtime decoding of images.

```
mkfifo /tmp/aptaudio
aptdec /tmp/aptaudio
sox -t pulseaudio alsa_output.pci-0000_00_1b.0.analog-stereo.monitor -c 1 -t wav /tmp/aptaudio
```

Perform a realtime decode with the audio being played out of `alsa_output.pci-0000_00_1b.0.analog`.

## Further reading

[https://noaasis.noaa.gov/NOAASIS/pubs/Users_Guide-Building_Receive_Stations_March_2009.pdf](https://noaasis.noaa.gov/NOAASIS/pubs/Users_Guide-Building_Receive_Stations_March_2009.pdf)  
[https://web.archive.org/web/20141220021557/https://www.ncdc.noaa.gov/oa/pod-guide/ncdc/docs/klm/tables.htm](https://web.archive.org/web/20141220021557/https://www.ncdc.noaa.gov/oa/pod-guide/ncdc/docs/klm/tables.htm)

## License

See LICENSE.