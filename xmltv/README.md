# keywordSearch

This script searches the TV guide data file that a XMLTV grabber generates for program titles, subtitles,  descriptions, or cast/crew names that match a collection of search strings.  When it finds a match it sends a request to Jellyfin asking that it create a timer that will record the program.  Jellyfin must be using XMLTV as its TV guide program data provider and the keywordSearch script must reference the same XMLTV file or URL that Jellyfin does.

## Installation

* Copy the keywordSearch script to a convenient directory on the Jellyfin server.  As distributed, the script assumes it will be placed in the /opt/xmltv directory. 
* Edit the script and modify the values in the configuration section at the top.  At a minimum the following variables need to be set:
  * The <code>searches</code> array needs to modified to specify the searches you want the script to perform.  This is an array of Python regular expressions.  Some examples are included but complete details on defining Python regular expressions can be found at https://docs.python.org/3/library/re.html
  * The <code>userid</code> and <code>password</code> variables need to be set to the Jellyfin userid and password that the script can use to access Jellyfin.  Creating a unique, separate id for use by the script can help track its activity in the Jellyfin dashboard.
  * The <code>xmltv_file</code> variable needs to be set to the full path of the file that is output by the XMLTV grabber that provides TV guide data to Jellyfin.
    * Obtain the location of the file from <code>Dashboard -> Live TV</code> in jellyfin.

## Dependency On [postprocessRecording](https://github.com/dlk3/Jellyfin-hacks/tree/master/postprocessRecording) Script

It's optional, but one of the checks that keywordSerarch does is to determine if the show it has found in its search has ever been recorded by Jellyfin before.  If it finds that has been, then it skips recording it again.  This search uses a database that is created by my [postprocessRecording](https://github.com/dlk3/Jellyfin-hacks/tree/master/postprocessRecording) script.  Wheneve it post-processes a recording file it records information about that file into a database table.  keywordRecording can then use that same database table to lookup programs that it has found while searching the program guide.

If postprocessRecording isn't being used, i.e., the database table it would have created does not exist, then keywordRecording ignores this check.

## Use

The script should be scheduled to run regularly, probably once a day, using cron, a systemd timer, or some other similar facility.  Be sure to check file permissions to ensure that the userid under which the script runs has read access to the XMLTV output file and write access to the directory where the script will write its log file, as specified in the configuration section at the top of the script.

# mkconfig

This script reads the TV channel lineup off of a Silicon Dust HDHomerun tuner, downloads a corresponding channel lineup from your Schedules Direct account, and then uses those lineups to construct a configuration file that can be used by an XLMTV grabber script to download TV guide program data from Schedules Direct for only those channels you care about.

This script is based on the fact that I spent the time to go through the complete channel lineup that my cable company provides to my HDHomerun tuner and to mark as deleted those channels that I will never ever watch.  This is done by accessing the http://x.x.x.x/ineup.html URL in the browser, where <code>x.x.x.x</code> is the IP address of my HDHomerun device, and marking those channels to be ignored with a red "X".  Once that is complete, those channels will no longer be available for viewing in the HDHomerun Android app or Android TV app, which makes navigation in those apps easier.  To similarly limit the channels visible in the program guides in MythTV and Jellyfin, I needed a way to build a XMLTV config file that contained only those channels that were left undeleted.  That is what mkconfig does.

## Installation

Copy the script to your system and modify the configuration variables at the top of the script.  You need to specify your Schedules Direct userid, password and the name of the Schedules Direct lineup that matches the cable service your HDHomerun is connected to.  You may also want to modify the paths specified in the script for the XMLTV cache file and the output XMLTV configuration  file.

## Use

This script is run manually, on the rare occasions when you need to update the channel lineup in your XMLTV configuration file.  These occasions occur when your cable company makes changes to their channel offerings and the channel lineup on your HDHomerun tuner changes.
