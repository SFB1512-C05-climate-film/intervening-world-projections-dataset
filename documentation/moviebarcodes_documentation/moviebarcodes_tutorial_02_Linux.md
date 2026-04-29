# 2. Linux (Ubuntu):
## 2.1 Preparations

Installing FFmpeg and ImageMagick

FFmpeg is a free multimedia framework that we will use to extract individual frames from a video file. These frames will then be combined into the finished MovieBarcode using ImageMagick, a free image editing programme.

To install both programmes, enter the following commands in the terminal:

```Bash
sudo apt update
sudo apt install ffmpeg imagemagick -y
```

The next step is to verify that the installation process was completed successfully:

```Bash
ffmpeg -version
magick -version
```

>**Note**: This guide uses the version of ImageMagick available in the Ubuntu repository. This version is outdated, but much easier to install. However, if the repository version has been updated since then, or if you decide to install a more recent version (7+) yourself, you will need to replace the `convert` command in section 2.2.3 with `magick`.

## 2.2 Creating the barcode

Create a working directory and navigate to it in the terminal. For example, to create a directory under `~/moviebarcode`:

```Bash
mkdir ~/moviebarcode
cd ~/moviebarcode
```

For best results, use a directory path and file name that does not contain spaces or capital letters.

Place the video file from which you wish to generate the barcode in this directory. In this guide, we will use a sample video called `barcode_test.mkv`. In the following commands, replace this placeholder with the name of your video file.

Enter all of the following commands directly in the working directory.

### 2.2.1 Designing the final barcode

In order to determine the parameters for extracting the frames, we first need to define the barcode's final dimensions. In this guide, we will create two barcodes: one with dimensions of `2520 x 1080 px` and another with dimensions of `1680 x 720 px`. These dimensions are based on the resolution of the source video from which the barcode is generated; the height is the decisive factor here.

```Bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height barcode_test.mkv
```

The output should look something like this:

```
[STREAM]  
width=1920  
height=1080  
[/STREAM]
```

The `height` value specifies the maximum useful height of the final barcode in pixels. Anything above this would require additional image information that is not available in the source file. While this is possible, it should generally be avoided unless the source file has too low a resolution to generate a usable barcode.

At this point, the width of the final barcode can be set can be set fairly freely, even if very short source files may not contain enough frames to fill the required width. While this is not usually problematic, it can cause the barcode to appear more blocky, as frames are output multiple times to fill the X-axis.

For the purposes of this guide, we will set the width to `2520 px` and the final resolution to `2520 x 1080 px`.

### 2.2.2. Extracting the individual frames from the source file

To fill the width of `2,520 px`, we need 2520 individual frames from the source file – one frame per pixel width. To ensure an even distribution of these frames over the video's runtime, we first need to determine the runtime in seconds.

```Bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 .\barcode_test.mkv
```

The output is the duration of the video in seconds. In this case, it is `297.708`. This can be used to calculate the required number of frames per second (FPS).

```Bash
awk 'BEGIN { print sprintf("%.3f", <Finale-Breite> / <Dauer-in-Sekunden>) }'
```

```Bash
# Example
awk 'BEGIN { print sprintf("%.3f", 2520 / 297.708) }'
```

This results in an FPS value rounded to three decimal places — in this case, `8.465`. The frames can then be extracted using the following command, which reduces their width to `1 px` each.

```Bash
ffmpeg -i barcode_test.mkv -vf "fps=8.465,scale=1:ih:flags=lanczos" frame_%04d.png
```

Enter the previously determined FPS value after `fps=`. Please note that the FPS number must be entered with a decimal point, not a comma. The scale argument (`scale=1:ih`) instructs the programme to reduce the width of the individual frames to 1px while maintaining the height. If you want the final barcode to have a different height, you can specify this in pixels instead of `ih` (for example, `scale=1:720` for a height of `720 px`).

The individual frames are stored in the working directory and numbered consecutively.

#### 2.2.2.1. Extracting the individual frames as a Bash script

The process of determining the FPS and extracting the individual frames can be automated using the following Bash script. Be sure to use the file name of your source file as the value for `VIDEO` and the desired width of your final barcode as the value for `FRAME_COUNT`.

```Bash
VIDEO="barcode_test.mkv"
FRAME_COUNT=2520

# Determine video duration
DURATION=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$VIDEO")

# Calculate target FPS
FPS=$(LC_NUMERIC=C awk "BEGIN { printf \"%.3f\", $FRAME_COUNT / $DURATION }")

echo "Duration: $DURATION seconds, FPS: $FPS"

# Extract frames
ffmpeg -i "$VIDEO" -vf "fps=$FPS,scale=1:ih:flags=lanczos" frame_%04d.png
```

### 2.2.3. Combining the individual frames into a barcode and scaling

The extracted frames now just need to be combined into the final barcode:

```Bash
convert frame_*.png +append moviebarcode_1080.png
```

We can then generate a smaller version of the barcode:

```Bash
convert moviebarcode_1080.png -resize x720 moviebarcode_720.png
```

The parameter `x720` specifies the height of the scaled version in pixels and can be adjusted accordingly.

### 2.2.4. Automating the entire barcode creation process via Bash script

Enter the desired parameters in the first lines:

```Bash
# Parameters
VIDEO="barcode_test.mkv"
FRAME_COUNT=2520
OUTFILE="moviebarcode"
PRIMARY_HEIGHT="1080"
SCALED_HEIGHT="720"

# Determine video duration
DURATION=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$VIDEO")
FPS=$(LC_NUMERIC=C awk "BEGIN { printf \"%.3f\", $FRAME_COUNT / $DURATION }")

echo "Dauer: $DURATION Sekunden, FPS: $FPS"

# Extract frames
ffmpeg -i "$VIDEO" -vf "fps=$FPS,scale=1:$PRIMARY_HEIGHT:flags=lanczos" frame_%04d.png

# Append frames
convert frame_*.png +append "${OUTFILE}_${PRIMARY_HEIGHT}.png"

# Scaled version
convert "${OUTFILE}_${PRIMARY_HEIGHT}.png" -resize x$SCALED_HEIGHT "${OUTFILE}_720.png"

# Delete individual frames
rm frame_*.png
```