# *Remove-Ads*
A BASH-Script to Extract Adverts from MP4 / MP3 / M4A Video/Music Files

## *Basics*
This script file is designed for use by any-user (no root access required) within a Terminal. It’s principal purpose is to make it easy & reliable to trim unwanted sections from video/music files. It has been tested for 18 months under *Devuan* (a systemD-free version of *Debian*). In addition to BASH, the main utilities required are common items such as FFMPEG, GREP & AWK. Each required app is tested for before use.

## *Begin*
The following will assume that you have created a dir `~/.local/sbin` within which you store the bash-file; this is to set the script as executable for current user only:

```bash
$ chmod 0700 ~/.local/sbin/remove_ads
$ la ~/.local/sbin/remove_ads
-rwx------ 1 user user 22816 May 29 15:19 /home/user/.local/sbin/remove_ads
```
## *Help*
Attempting to run the bare script with zero parameters shows a brief Help message:

```bash
$ ~/.local/sbin/remove_ads
!Error! No MP4NAME nor ACTION in command.
usage: /home/user/.local/sbin/remove_ads MP4NAME -ACTION
```
Adding a `-h` parameter will then give a longer help message:

```bash
$ ~/.local/sbin/remove_ads -h
usage: /home/user/.local/sbin/remove_ads MP4NAME -ACTION

1. CUT-out sections from MP4 video, or
2. JOIN the cutouts into a new video
utilities:
3. ADD keyFrames to a mp4
4. LIST keyFrames from mp4 to disk

    MP4NAME      : mp4-source filename
ACTIONs:
   -a ADD "time1,time2,…"
      Time format: 'HH:MM:SS(.m)' [hours:minutes:seconds(.milli-secs)]
            time1: keyFrame1
            time2: keyFrame2
   -c CUT "time1,time2"
      Time format: 'HH:MM:SS(.m)' [hours:minutes:seconds(.milli-secs)]
            time1: start of section
            time2:   end of section
   -h HELP       : Show Help
  --help         : Show Full Help
   -j JOIN       : Stitch all CUT-files together
      Makes use of parts.txt (a plain-text list of files)
   -k KEYFRAME   : (utility) List keyframes to disk-file

Note: CUT + JOIN + HELP may also be used on mp3/m4a audio files
      read 'mp4' as 'mp4 or mp3 or m4a' in HELP for those options

WORKFLOW:
# https://trac.ffmpeg.org/wiki/Concatenate#demuxer
# http://trac.ffmpeg.org/wiki/Seeking
In brief:–
  a) Place the MP4-with-ads into an empty directory
  b) Locate the Start-time + End-time of all Ad-breaks
  c) -c CUT each ad-free section out to disk
  d) Check that parts.txt contains all ad-free sections
  e) -j JOIN all ad-free sections into a new mp4
```
… whilst adding a `--help` parameter will then drown you in help:

```bash
$ ~/.local/sbin/remove_ads --help
usage: /home/user/.local/sbin/remove_ads MP4NAME -ACTION

1. CUT-out sections from MP4 video, or
2. JOIN the cutouts into a new video
utilities:
3. ADD keyFrames to a mp4
4. LIST keyFrames from mp4 to disk

    MP4NAME      : mp4-source filename
ACTIONs:
   -a ADD "time1,time2,…"
      Time format: 'HH:MM:SS(.m)' [hours:minutes:seconds(.milli-secs)]
            time1: keyFrame1
            time2: keyFrame2
   -c CUT "time1,time2"
      Time format: 'HH:MM:SS(.m)' [hours:minutes:seconds(.milli-secs)]
            time1: start of section
            time2:   end of section
   -h HELP       : Show Help
  --help         : Show Full Help
   -j JOIN       : Stitch all CUT-files together
      Makes use of parts.txt (a plain-text list of files)
   -k KEYFRAME   : (utility) List keyframes to disk-file

Note: CUT + JOIN + HELP may also be used on mp3/m4a audio files
      read 'mp4' as 'mp4 or mp3 or m4a' in HELP for those options

WORKFLOW:
# https://trac.ffmpeg.org/wiki/Concatenate#demuxer
# http://trac.ffmpeg.org/wiki/Seeking
In brief:–
  a) Place the MP4-with-ads into an empty directory
  b) Locate the Start-time + End-time of all Ad-breaks
  c) -c CUT each ad-free section out to disk
  d) Check that parts.txt contains all ad-free sections
  e) -j JOIN all ad-free sections into a new mp4

At dreadful length:–
  1) Place the MP4-with-ads into an empty directory
  -------------------------------------------------

Your device will require free-space to at least 2 x the size of the original mp4

  2) Locate the Start-time + End-time of all Ad-breaks
  ----------------------------------------------------

MPV video-player, launched from the terminal, can display times to the msec:
eg
See https://mpv.io/manual/master/:–
$ mpv --osd-fractions Captain-Blood-1935.mp4
 (+) Video --vid=1 (*) (h264 720x576 24.999fps)
 (+) Audio --aid=1 (*) (aac 2ch 48000Hz)
AO: [pulse] 48000Hz stereo 2ch float
VO: [gpu] 720x576 => 768x576 yuv420p
(Paused) AV: 00:00:35.732 / 01:50:05.354 (0%) A-V:  0.003

thus: time of paused frame == "00:00:35.732"

Keyboard Control:–
LEFT and RIGHT:
  Seek backward/forward 5 seconds
UP and DOWN:
  Seek forward/backward 1 minute
.:
  Step forward & play one frame then pause (fast)
,:
  Step back & play one frame (slow)
q and Q:
  Stop playing and quit
  (remember current & resume later if 'Q')

  3) -c CUT each ad-free section out to disk
  ------------------------------------------

see https://ffmpeg.org/ffmpeg-utils.html#Time-duration
This program accepts either of the 2 formats below
(each in this instance would indicate zero seconds):–
  '00:00:00'  or
  '00:00:00.000'

Thus, as a minimum example of a video with a single ad-break,
you would trawl through the video to find 4 times:
    i) BEGIN-VIDEO
   ii) BEGIN-AD
  iii) END-AD
   iv) END-VIDEO

(the 4 items above are all intended to represent times in the accepted format)

You would then call this script twice with the following values:–
  1. /home/user/.local/sbin/remove_ads MP4NAME -c "BEGIN-VIDEO,BEGIN-AD"
  2. /home/user/.local/sbin/remove_ads MP4NAME -c "END-AD,END-VIDEO"

The script would save 2 mp4 files, called "cut-000.mp4" + "cut-001.mp4". It
would also create a file called parts.txt, which would contain the following lines:
  file 'cut-000.mp4'
  file 'cut-001.mp4'

  4) Check that parts.txt contains all ad-free sections
  -----------------------------------------------------

The files referenced within parts.txt — and only those files — will be
stitched together in the same order as they appear within parts.txt.

  5) -j JOIN all ad-free sections into a new mp4
  ----------------------------------------------

The new ad-free mp4 is called the same as 'MP4NAME' with "noad" added to the end
of the name. No intermediate files are deleted. Make sure that you empty the dir
after moving 'MP4NAME-noad' to a safe place.

Two utility functions are provided:

  A) -k list all KEYFRAMEs from mp4 to kf.txt
  --------------------------------------------

Non-destructive, this does what it says on the tin. KeyFrame times are shown in
the same hh:mm:ss.mss format as input times (see (3) above).

  B) -a ADD a set of KeyFrames to a mp4
  -------------------------------------

This utility is used when you either want very short CUTs from a mp4, and/or if
you cannot get an accurate CUT (either BEGIN or END) with an mp4, and perhaps a
scene-change is leaking on to the beginning/end of a CUT.

Try to use this only with short videos. Here is a decision matrix:
     • “-c copy” can NOT be used (KF would not be added)
     • typical add times are ~1:1 (ie ~1 hr for a 1hr mp4)
     • typical default use is 1:250 for KF:f (every 10 secs at 25 fps)
     • KF are *supposed* to be added by default at all scene-changes;
       my experience is KF sometimes missing/misplaced at scene-changes
     • force-adding a KF can change all other KF around it
     • ffmpeg uses force-KF millisec times as advisory, not obligatory
     • Thus, if you are having difficulty getting accurate -c CUT times:–
       ◦ over-CUT a section out of the mp4
       ◦ -a ADD a set of KF to the CUT-out section; include:–
            → a 00:00:00.000 KF (first one)
            → an END-time KF    (last one)
            → then BEGIN-Time + END-Time KFs for each section wanted
       ◦ it should now be possible to get accurate cuts on each KF

general notes:
     • “-c copy” means it takes seconds to process hours-long videos
     • ( the speed is achieved by placing “-ss” *before* “-i”; ffmpeg then   )
       ( seeks to the time using KFs; this all can mean very inaccurate cuts )
     • “-c copy” also means that each CUT-file MUST have the same format
       (in this case yes, as has been cut from the same source-file)
     • Each new video begins at 00:00:00, as does the no-ad mp4
     • No steps are taken for subtitles, chapters or any other streams
       (global metadata tends to be copied across eg Title)
     • This script has been tested only on MP4 videos
```
