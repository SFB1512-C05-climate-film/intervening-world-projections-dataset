# 3. MacOS:
## 3.1. Preparations
### 3.1.1. Install Homebrew

We use Homebrew to install FFmpeg and ImageMagick. Homebrew is a free package manager for macOS. To install Homebrew, open a terminal window and enter the following command:

```Bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 3.1.2. Installing FFmpeg and ImageMagick

FFmpeg is a free multimedia framework that we will use to extract individual frames from a video file. These frames are then combined into the finished MovieBarcode using ImageMagick, a free image editing programme.

To install both programmes, enter the following commands in the Terminal:

```Bash
brew install ffmpeg imagemagick
```

Then check that the installation was successful:

```Bash
ffmpeg -version
magick -version
```

## 3.2. Creating the barcode

Now create a working directory and navigate to it in the terminal. For example, to create a directory under `~/moviebarcode`:

```Bash
mkdir ~/moviebarcode
cd ~/moviebarcode
```

For best results, use a directory path and file name that does not contain spaces or uppercase letters.

Place the video file from which you want to generate the barcode in this directory. In this guide, we will use a sample video called `barcode_test.mkv`. In the following commands, replace this placeholder with the name of your video file.

Enter all of the following commands directly into the working directory.

### 3.2.1. Designing the final barcode

To determine the parameters for extracting the frames, we must first define the barcode's final dimensions. In this guide, we will create a barcode with dimensions of `2520 x 1080 px`, as well as a smaller version with dimensions of `1680 x 720 px`. These dimensions are based on the resolution of the source video from which the barcode is to be generated; the height is the decisive factor here:

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

The 'height' value specifies the maximum useful height of the final barcode in pixels. Anything above this would require additional image information that is not available in the source file. While this is possible, it should generally be avoided unless the source file has too low a resolution to generate a usable barcode.

At this point, the width of the final barcode can be set can be set fairly freely, even if very short source files may not contain enough frames to fill the required width. While this is not usually problematic, it can cause the barcode to appear blockier, as frames are output multiple times to fill the X-axis.

For the purposes of this guide, we will set the width to `2520 px` and the final resolution to `2520 x 1080 px`.

### 3.2.2. Extracting the individual frames from the source file

To fill the width of `2520 px`, we need 2520 individual frames from the source file – one frame per pixel width. To ensure an even distribution of these frames over the video's runtime, we first need to determine the runtime in seconds.

```Bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 barcode_test.mkv
```

The output is the duration of the video in seconds. In this case, `297.708`. This can be used to calculate the number of frames per second (FPS) required:

```Bash
awk 'BEGIN { print sprintf("%.3f", <Finale-Breite> / <Dauer-in-Sekunden>) }'
```

```Bash
# Example
awk 'BEGIN { print sprintf("%.3f", 2520 / 297.708) }'
```

This results in an FPS value rounded to three decimal places—in this case, `8.465`. The frames can then be extracted using the following command, which also reduces them to a width of `1 px` each.

```Bash
ffmpeg -i barcode_test.mkv -vf "fps=8.465,scale=1:ih:flags=lanczos" frame_%04d.png
```

The previously determined FPS value is entered here after `fps=`. Please note that the value must be entered with a period, not a comma. The argument `scale=1:ih` instructs the programme to reduce the individual frames to a width of 1px while maintaining the height. If you want a different height for the final barcode, this can be specified in pixels instead of `ih` (for example: `scale=1:720` for a height of `720 px`).

The individual frames are stored in the working directory and numbered consecutively.

#### 3.2.2.1. Extracting the individual frames as a Bash script

The process of determining the FPS and extracting the individual frames can be automated with the following Bash script. Be sure to use the file name of your source file as the value for `VIDEO` and the desired width of your final barcode as the value for `FRAME_COUNT`:

```Bash
VIDEO="barcode_test.mkv"
FRAME_COUNT=2520

# Determine video length
DURATION=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$VIDEO")

# Calculate FPS target
FPS=$(LC_NUMERIC=C awk "BEGIN { printf \"%.3f\", $FRAME_COUNT / $DURATION }")

echo "Duration: $DURATION seconds, FPS: $FPS"

# Extract frames
ffmpeg -i "$VIDEO" -vf "fps=$FPS,scale=1:ih:flags=lanczos" frame_%04d.png
```

### 3.2.3. Combining the individual frames into a barcode and scaling

The extracted frames now just need to be combined into the final barcode:

```Bash
magick frame_*.png +append moviebarcode_1080.png
```

We can then generate a smaller version of the barcode:

```Bash
magick moviebarcode_1080.png -resize x720 moviebarcode_720.png
```

The parameter `x720` specifies the height of the scaled version in pixels and can be adjusted accordingly.

### 3.2.4. Automating the entire barcode creation process via Bash script

Enter the desired parameters in the first lines:

```Bash
# Parameters
VIDEO="barcode_test.mkv"
FRAME_COUNT=2520
OUTFILE="moviebarcode"
PRIMARY_HEIGHT=1080
SCALED_HEIGHT=720

# Determine video length
DURATION=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$VIDEO")
FPS=$(LC_NUMERIC=C awk "BEGIN { printf \"%.3f\", $FRAME_COUNT / $DURATION }")

echo "Dauer: $DURATION Sekunden, FPS: $FPS"

# Extract frames
ffmpeg -i "$VIDEO" -vf "fps=$FPS,scale=1:$PRIMARY_HEIGHT:flags=lanczos" frame_%04d.png

# Append frames
magick frame_*.png +append "${OUTFILE}_${PRIMARY_HEIGHT}.png"

# Generate scaled version
magick "${OUTFILE}_${PRIMARY_HEIGHT}.png" -resize x$SCALED_HEIGHT "${OUTFILE}_720.png"

# Delete extracted frames
rm frame_*.png
```
