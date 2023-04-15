---
layout: post
title: 'Tip #15: Convert Video to GIF'
date: 2023-04-15 09:00 +0100
description: 
image: 
category:
- tips
- windows
tags: batch gif
mermaid: false
---
Windows built-in **Snipping Tool** ships now with a video recorder. I have been waiting for this feature forever. The default file format is *MP4*. For this blog, my screen capture videos are converted to *Animated GIFs*. To convert these mp4's to gif's I this [great way](https://www.bannerbear.com/blog/how-to-make-a-gif-from-a-video-using-ffmpeg/).

### Manual

The all powerful **ffmpeg** does a great job and there are win64 binaries too.

I begun with these two lines:

```batch
REM Generate the palette from the input file
ffmpeg -i "your.mp4" -filter_complex "[0:v] palettegen" temp_palette.png
REM Create the GIF using the palette
ffmpeg -i "your.mp4" -i temp_palette.png -filter_complex "[0:v][1:v] paletteuse" "your.gif"
```

### Batch file

While that approach works for single use, I wanted a more automated solution. I recalled that dropping a file onto a batch file would pass the file path as a parameter. My idea was to create a batch file that could accept dropped mp4 files and generate an animated gif using the above commands in the same directory.

This is what I came up with: (Note: ffmpeg.exe must be in the same directory as the batch file).

```batch
@echo off

REM Set input file name and output file name
set "input_file=%~1"
set "output_file=%~n1.gif"

echo.
echo "InputFile: %input_file%"
echo "OutputFile: %output_file%"

REM Generate the palette from the input file
%~dp0ffmpeg -i "%input_file%" -filter_complex "[0:v] palettegen" palette.png

REM Create the GIF using the palette
%~dp0ffmpeg -i "%input_file%" -i palette.png -filter_complex "[0:v][1:v] paletteuse" "%output_file%"

REM Cleanup intermediate files
del palette.png
echo Completed.
```

![Drag and drop mp4 to batch](/assets/img/tip-15/batchconvertogif.gif)_drag and drop mp4_

### Automated

I wanted to go beyond just dragging and dropping a file onto a batch file. I aimed to add this functionality to the explorer’s context menu, allowing me to run it on any mp4 file, regardless of its location.

![Explorer context menu](/assets/img/tip-15/explorercontextmenu.png)_Convert Video to GIF_

The steps required for this are:

- download ffmpeg.exe
  - ```curl -o "%temp%\ffmpeg.zip" -L "https://github.com/GyanD/codexffmpeg/releases/download/6.0/ffmpeg-6.0-essentials_build.zip"```
- copy the batch file, ffmpeg and an [icon](/assets/img/tip-15/VideoToGIF.ico) to a common location, I chose %ProgramData%
- Register context menu in the shell (registry)
  - ```reg add "HKEY_CLASSES_ROOT\*\shell\Convert Video to GIF" /ve /d "Convert Video to GIF" /f```
  - ```reg add "HKEY_CLASSES_ROOT\*\shell\Convert Video to GIF" /v "Icon" /d "\"C:\\ProgramData\\VideoToGif\\VideoToGIF.ico\"" /f```
  - ```reg add "HKEY_CLASSES_ROOT\*\shell\Convert Video to GIF\command" /ve /d "\"C:\\ProgramData\\VideoToGIF\\VideoToGIF.bat\" \"%%1\"" /f```

I created an install.bat file that must be run as an administrator to handle all of these steps.

```batch
@echo off
NET FILE >NUL 2>&1
IF %ERRORLEVEL% NEQ 0 (
    ECHO This script must be run as Administrator!
    PAUSE
    EXIT /B
)

setlocal

REM Create VideoToGIF folder in ProgramData
echo Creating VideoToGIF folder...
if not exist "%ProgramData%\VideoToGIF" mkdir "%ProgramData%\VideoToGIF"

REM Download ffmpeg to temporary directory and copy to VideoToGIF folder
echo Downloading ffmpeg and copying to VideoToGIF folder... 
curl -o "%temp%\ffmpeg.zip" -L "https://github.com/GyanD/codexffmpeg/releases/download/6.0/ffmpeg-6.0-essentials_build.zip"
powershell -Command "$tempDir = [System.IO.Path]::GetTempPath(); Expand-Archive "$tempDir\ffmpeg.zip" -DestinationPath "%temp%\ffmpeg" -Force"
del %temp%\ffmpeg.zip
copy "%temp%\ffmpeg\ffmpeg-6.0-essentials_build\bin\ffmpeg.exe" "%ProgramData%\VideoToGIF"
rmdir /s /q %temp%\ffmpeg
copy "%~dp0\VideoToGIF.bat" "%ProgramData%\VideoToGIF"
copy "%~dp0\VideoToGIF.ico" "%ProgramData%\VideoToGIF"

REM Adding context menu Convert to GIF
echo Importing registry keys...
reg add "HKEY_CLASSES_ROOT\*\shell\Convert Video to GIF" /ve /d "Convert Video to GIF" /f
reg add "HKEY_CLASSES_ROOT\*\shell\Convert Video to GIF" /v "Icon" /d "\"C:\\ProgramData\\VideoToGif\\VideoToGIF.ico\"" /f
reg add "HKEY_CLASSES_ROOT\*\shell\Convert Video to GIF\command" /ve /d "\"C:\\ProgramData\\VideoToGIF\\VideoToGIF.bat\" \"%%1\"" /f

echo Installation complete.
pause
```

Find all files in this [VideoToGIF.zip](/assets/img/tip-15/VideoToGIF.zip)

### Notes

- Converting videos to animated gif's works best with small videos with little color changes
