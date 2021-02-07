# postprocessRecording

Jellyfin's Live TV recordings do not play well on my Nvidia Shield Android TV boxes.  Jellyfin's builtin player does not fast-forward over commericals at all well and it's hard to pause or rewind too.  Switching to the use of the external VLC player in Jellyfin improves playback navigation quite a bit but this player cannot play the transport stream (*.ts) files that are Jellyfin's Live TV recordings.  I discovered that if I re-multiplexed the *.ts files into *.mp4 files using ffmpeg, then the VLC player was able to play the Live TV recording *.mp4 file without any problem and this solved the playback navigation issue altogether.  Re-multiplexing in this fashion, without re-encoding, ensured that there was no loss of video or audio quality in the recording.

So that's what the postprocessRecording script does, it first runs the comskip program against the recording's *.ts file to find what it thinks are the commercials, generating a ffmetadata format file containing the commecial breaks.  Then it runs ffmpeg, copying the video and the audio streams from the *.ts file into a *.mp4 file, while adding the chapter metadata from the ffmetadata file.  It then makes the *.ts file into a hidden file by adding a dot (.) to the front of it's name.  

I kept the *.ts file as a hidden file because, as it turns out, if I tried to use my video editing tool of choice, kdenlive, to edit the recording and convert if to a TV Show video in Jellyfin, it has trouble using the post-processed *.mp4 file.  It sees the audio and video as totally out of sync in the *.mp4 file.  This isn't a problem when VLC plays the *.mp4 file, but it is for kdenlive.  I don't know why.  The work-around I found was to keep the *.ts file around and edit it with kdenlive when I wanted to convert a recording into a video.  Renaming the *.ts file with a dot in front of its name makes it so that Jellyfin doesn't see it any more and won't try to present it for playback.

I made a small script called ?SOMETHING? that is run out of cron on a regular basis to scan the Live TV recordings directory for orphaned hidden *.ts files that have been left behind when their associated recording files have been deleted through the Jellyfin UI.  It deletes any of these it finds.

## Installation

Copy the script to the Jellyfin server.  Specify it as the DVR post-processing command in the Jellyfin administrator's Dashboard.  Check the configuration variables at the top of the script to make sure that the
directory the script uses for its log file makes sense.

