# 1. Windows:

## 1.1. Preparations

### 1.1.1. Install Chocolatey

Chocolatey is a package manager for Microsoft Windows. In this guide, we will use Chocolatey to install FFmpeg and ImageMagick. If you already have FFmpeg and ImageMagick installed or would like to install them manually, you can skip this step.

Open PowerShell as an administrator. (Windows key + X, then select “Windows PowerShell (Admin)”.)  
Enter the following command:

```ps1
Set-ExecutionPolicy Bypass -Scope Process -Force; `
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

You can then verify that the installation was successful:

```ps1
choco --version
```

### 1.1.2. Installing FFmpeg and ImageMagick

FFmpeg is a free multimedia framework that we will use to extract individual frames from a video file. These frames are then combined into the finished MovieBarcode using the free image editing programme ImageMagick.

To install both programmes with Chocolatey, enter the following command in PowerShell (Admin):

```ps1
choco install ffmpeg imagemagick -y
```

Chocolatey will then install both programmes and any necessary dependencies. You may be prompted by PowerShell to restart your computer. Restart your computer or at least close the current PowerShell instance and open a new one.
Then check that the installation was successful:

```ps1
ffmpeg -version
magick -version
```

Both commands should display the versions of the respective programmes without any error messages.

## 1.2. Creating the barcode

Now create a working directory and navigate to it in PowerShell. For example, to create a directory under `C:\moviebarcodes`:

```ps1
mkdir C:\moviebarcodes
cd C:\moviebarcodes
```

It is recommended to use a directory path and file name without spaces or uppercase letters.

Place the video file from which you want to generate the barcode in this directory. For this guide, we are working with a sample video with the file name `barcode_test.mkv`. In the following commands, this placeholder must be replaced with the name of your video file.

Run all of the following commands directly in the working directory.

### 1.2.1. Designing the final barcode

To determine the parameters for extracting the frames, we must first define the final dimensions of the barcode. In this guide, we will create a barcode with dimensions of `2520 × 1080 px`, as well as a smaller version with dimensions of `1680 × 720 px`. The dimensions are based on the resolution of the source video from which the barcode is to be generated, with the height being the decisive factor here:

```ps1
ffprobe -v error -select_streams v:0 -show_entries stream=width,height "barcode_test.mkv"
```

The output should look something like this:

```
[STREAM]
width=1920
height=1080
[/STREAM]
```

The `height` value specifies the maximum useful height of the final barcode in pixels, as anything above this would require additional image information that is not available in the source file. This is possible, but should generally be avoided – unless the source file has too low a resolution to generate a usable barcode.

The width of the final barcode can be set can be set fairly freely at this point, even if very short source files may not contain enough frames to fill the required width. This is usually not problematic, but it can cause the barcode to appear more coarse in width, as frames are output multiple times to fill the X-axis.

For the purposes of this guide, we will set a width of `2520 px` and, as already described, a final resolution of `2520 × 1080 px`.

### 1.2.2. Extracting the individual frames from the source file

To fill the width of `2520 px`, we need 2520 individual frames from the source file—one frame per pixel width. To ensure that these frames are distributed evenly over the runtime of the video, we first need to determine the runtime in seconds.

```ps1
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 .\barcode_test.mkv
```

The output can be used to calculate the number of frames per second (FPS) required in PowerShell:

```ps1
[math]::Round(<Finale-Breite> / <Dauer-in-Sekunden>, 3)
```

```ps1
# Example:
[math]::Round(2520 / 297.708, 3)
```

This results in an FPS value rounded to three decimal places – in this case `8.465`. The frames can then be extracted using the following command, which also reduces them to a width of `1 px` each:

```ps1
ffmpeg -i .\barcode_test.mkv -vf "fps=8.465,scale=1:ih:flags=lanczos" frame_%04d.png
```

The argument `scale=1:ih` instructs the programme to reduce the individual frames to a width of 1 px and maintain the height. If you want a different height for the final barcode, you can specify this in pixels instead of `ih` (for example: `scale=1:720` for a height of `720 px`).

The individual frames are now stored in the working directory with sequential numbers. Make sure you have enough storage space.

#### 1.2.2.1. PowerShell snippet for the extraction of the individual frames

The process of determining the FPS and extracting the individual frames can be automated with the following PowerShell snippet. Make sure to use the file name of your source file as the value for `$video` and the desired width of your final barcode as the value for `$frameCount`:

```ps1
$video = "barcode_test.mkv"
$frameCount = 2520

# Determine the video duration in seconds
$duration = ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 $video
$duration = [double]::Parse($duration.Trim(), [System.Globalization.CultureInfo]::InvariantCulture)

# Calculate the FPS target
$fps = [math]::Round($frameCount / $duration, 3)

Write-Output "Duration: $duration seconds"
Write-Output "Calculated FPS: $fps"

# Extract frames
ffmpeg -i $video -vf "fps=$fps,scale=1:ih:flags=lanczos" frame_%04d.png
```

### 1.2.3. Combining the individual frames into a barcode and scaling

Now the extracted individual frames just need to be combined into the final barcode:

```ps1
magick frame_*.png +append moviebarcode_1080.png
```

We can then generate a smaller version of the barcode:

```ps1
magick moviebarcode_1080.png -resize x720 moviebarcode_720.png
```

The parameter `x720` specifies the height of the scaled version in pixels and can be adjusted accordingly.

### 1.2.4. PowerShell snippet for generating the complete barcode

```ps1
$video = "barcode_test.mkv"
$frameCount = 2520

# Determine video duration
$duration = ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 $video
$duration = [double]::Parse($duration.Trim(), [System.Globalization.CultureInfo]::InvariantCulture)

# Calculate FPS target
$fps = [math]::Round($frameCount / $duration, 3)

Write-Output "Duration: $duration seconds"
Write-Output "Calculated FPS: $fps"

# Extract frames
ffmpeg -i $video -vf "fps=$fps,scale=1:ih:flags=lanczos" frame_%04d.png

# Append frames
magick frame_*.png +append moviebarcode.png

# Generate scaled version
magick moviebarcode.png -resize x720 moviebarcode_scaled.png
```