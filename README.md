# Scripts for pyWinContext
An ever-growing collection of Windows scripts written to be used with [pyWinContext](https://github.com/VodBox/pyWinContext).

Most scripts use [FFmpeg](https://www.ffmpeg.org/), make sure it's installed beforehand.

## Video
#### remux MKV/FLV

```bat
ffmpeg -i "%~1" -map 0 -codec copy "%~dnp1.mp4"
```

#### re-encode MP4

```bat
ffmpeg -i "%~1" -c:v libx264 -preset medium -crf 18 -c:a aac -b:a 192k "%~dnp1_1.mp4"
```

#### convert any video to MP4

```bat
ffmpeg -i "%~1" -c:v copy -c:a copy "%~dnp1.mp4" || ffmpeg -i "%~1" -c:v libx264 -preset fast -crf 18 -c:a aac -b:a 192k "%~dnp1.mp4"
```

## Audio
#### convert audio/video files to MP3

```bat
set "output=%~dnp1.mp3"

if /i "%~x1"==".mp3" set "output=%~dnp1_128.mp3"

ffmpeg.exe -i "%~1" -c:a libmp3lame -b:a 128k "%output%"
```

#### convert audio/video to MONO
```bat
set "input_file=%~1"

for %%f in ("%input_file%") do set "name=%%~nf"

for %%f in ("%input_file%") do set "ext=%%~xf"

set "output_file=%name%-mono%ext%"

ffmpeg -i "%input_file%" -ac 1 "%output_file%"
```

#### convert audio/video to STEREO

```bat
set "input_file=%~1"

for %%f in ("%input_file%") do set "name=%%~nf"

for %%f in ("%input_file%") do set "ext=%%~xf"

set "output_file=%name%-stereo%ext%"

ffmpeg -i "%input_file%" -ac 2 "%output_file%"
```


## Image
#### upscale image using Waifu2x Caffe
1. download and set up [Waifu2x Caffe](https://github.com/lltcggie/waifu2x-caffe)

2. change the path in the script to the waifu2x-caffe-cui.exe and the upscaling model

3. set the scale with "-s" to 2.0, 3.0, 4.0 for upscaling by 2x 3x or 4x


```bat
"D:\waifu2x-caffe\waifu2x-caffe-cui.exe" -p cudnn -s 2.0 -n 1 --model_dir "D:\waifu2x-caffe\models\upconv_7_photo" -i "%~1" -o "%~n1_2x%~x1"
```

#### convert image sequence to MP4

the png you click on will be the first frame of the resulting MP4.

save the script as a .bat file before adding it to pyWinContext.

```bat
@echo off
setlocal enabledelayedexpansion

:: Display the clicked file
echo Clicked file: %~1
set "clickedFile=%~1"
set "dir=%~dp1"
set "baseName=%~n1"
set "ext=%~x1"

:: Display extracted components
echo Directory: %dir%
echo Base name: %baseName%
echo Extension: %ext%

:: Ensure the extension includes the dot
if not "%ext:~0,1%"=="." (
    set "ext=.ext"
    echo Fixed extension to: %ext%
)

:: Initialize variables
set "digits="
set "maxDigits=0"

:: Extract digits from the end of the baseName
echo Starting to extract digits...
for /l %%i in (0,1,255) do (
    set "char=!baseName:~-%%i,1!"
    if not "!char!"=="" if "!char!" geq "0" if "!char!" leq "9" (
        set "digits=!char!!digits!"
    ) else if defined digits (
        goto foundDigits
    )
)

:foundDigits
:: Display extracted digits
echo Digits extracted: %digits%

:: Count the number of digits
echo Counting digits...
set /a maxDigits=0
for /l %%i in (0,1,255) do (
    set "char=!digits:~%%i,1!"
    if not "!char!"=="" (
        set /a maxDigits+=1
    )
)

:: Display maxDigits
echo Number of digits: %maxDigits%

:: Extract prefix (filename before the numbers)
echo Extracting prefix...
set "prefix="
for /f "tokens=1* delims=0123456789" %%A in ("%baseName%") do set "prefix=%%A"
echo Prefix: %prefix%

:: Ask user for the desired frame rate
set "frameRate="
echo ------------------------------------------------
echo Enter the desired frame rate (e.g., 30, 24, 60) or just hit enter for 30 fps
set /p frameRate="Frame Rate: "

:: Display the frame rate before validation
echo Frame rate entered: %frameRate%

:: Validate frame rate input
if "%frameRate%"=="" (
    echo Invalid input. Using default frame rate of 30.
    set "frameRate=30"
)

:: Display final frame rate
echo Final frame rate: %frameRate%

:: Construct the sequence pattern
set "sequencePattern=%dir%%prefix%%%0%maxDigits%d%ext%"
echo Sequence pattern constructed: %sequencePattern%

:: Validate that files matching the sequence pattern exist
if not exist "%dir%%prefix%%digits%%ext%" (
    echo Error: No files matching the sequence pattern "%sequencePattern%" found.
    pause
    exit /b 1
)

:: Output filename
set "outputFile=%dir%%prefix%.mp4"

:: Display output filename
echo Output file: %outputFile%

:: Run FFmpeg to convert the sequence to MP4
echo Running FFmpeg...
ffmpeg -framerate %frameRate% -start_number %digits% -i "%sequencePattern%" -c:v libx264 -pix_fmt yuv420p "%outputFile%"

:: Check if FFmpeg ran successfully
if %errorlevel% equ 0 (
    echo Conversion complete! Output saved to: %outputFile%
) else (
    echo Error during conversion.
)

:: Pause to keep the window open
echo Script complete. Press any key to close.
pause
```

## Disclaimer

scripts are written by yours truly, with help from my colleague and friend, Chaty (as in ChatGPT).

all free, as in free beer. Consu.. err.. use them responsibly.

scripts might contain errors but are tested and fully working for my use case, otherwise I would not share them on GitHub.
