# postprocessRecording

Jellyfin's Live TV recordings do not play well on my Nvidia Shield Android TV boxes.  Jellyfin's builtin player does not fast-forward over commericals at all well and it's hard to pause or rewind too.  Switching to the use of the external VLC player in Jellyfin improves playback navigation quite a bit but this player cannot play the transport stream (*.ts) files that are Jellyfin's Live TV recordings.  I discovered that if I re-multiplexed the *.ts files into *.mp4 files using ffmpeg, then the VLC player was able to play the Live TV recording *.mp4 file without any problem and this solved the playback navigation issue altogether.  Re-multiplexing in this fashion, without re-encoding, ensured that there was no loss of video or audio quality in the recording.

So that's what the postprocessRecording script does, it first runs the comskip program against the recording's *.ts file to find what it thinks are the commercials, generating a ffmetadata format file containing the commecial breaks.  Then it runs ffmpeg, copying the video and the audio streams from the *.ts file into a *.mp4 file, while adding the chapter metadata from the ffmetadata file.  It then makes the *.ts file into a hidden file by adding a dot (.) to the front of it's name.  

I kept the *.ts file as a hidden file because, as it turns out, if I tried to use my video editing tool of choice, kdenlive, to edit the recording and convert if to a TV Show video in Jellyfin, it has trouble using the post-processed *.mp4 file.  It sees the audio and video as totally out of sync in the *.mp4 file.  This isn't a problem when VLC plays the *.mp4 file, but it is for kdenlive.  I don't know why.  The work-around I found was to keep the *.ts file around and edit it with kdenlive when I wanted to convert a recording into a video.  Renaming the *.ts file with a dot in front of its name makes it so that Jellyfin doesn't see it any more and won't try to present it for playback.  I have another script called <code>cleanupJellyfinRecordings</code> that I run on a regular schedule to delete any orphaned, hidden *.ts files.  These are files that are left behind when the user delets a recording from the Jellyfin UI.

The <code>postprocessRecording</code> script also adds information about any recording that it post-processes to a database.  This supports another script I wrote called [keywordRecording](https://github.com/dlk3/Jellyfin-hacks/tree/master/postprocessRecording).  That script schedules recordings in Jellyfin whenever it finds a show in the program guide that matches a set of regular expression search strings.  It needed a way to tell if a specific program had ever been recorded in the past so that it would not schedule it to be recorded again.  It uses the database that postprocessRecording creates for this purpose.

## Installation

Copy the script to the Jellyfin server.  Specify it as the DVR post-processing command in the Jellyfin administrator's Dashboard.  Check the configuration variables at the top of the script to make sure that the
directory the script uses for its log file makes sense.

Create a new database in your favorite SQL database system to hold the LiveTV recording records that this script will create.  Create a user who has rights to create and write to tables within the database, if your database engine requires this.  Set the <code>db_url</code> variable in the script to specify the database you have created for SQLAlchemy.  See https://docs.sqlalchemy.org/en/13/core/engines.html#sqlalchemy.create_engine for details on how to format the URL for your preferred database engine.

Copy the <code>cleanupJellfinRecordings</code> script to the Jellyfin server.  Set the <code>recordings_dir</code> variable in the script to point at the directory that holds Jellyfin's LiveTV recordings.  Schedule the script to run daily through <code>cron</code> or a similar facility on the server.

##  Usage

Jellyfin will automtaically call the script whenever a LiveTV timer finishes recording a program.

You should periodically check the log file for errors, that's <code>keywordRecording.log</code> in the directory you specified at the top of the script.

If the logging level is set to logging.DEBUG, then additional log files will be created in the /tmp directory.  These files record the responses that the script got from the API calls it made to Jellyfin as part of its search for programs to record.  These response are so large that they harm the readability of the main log file if they are included there.
