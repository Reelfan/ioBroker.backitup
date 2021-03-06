***Translation by https://www.deepl.com/translator***

Backitup is a backup solution that allows cyclical backup of an IoBroker installation and a Homematic CCU.

In the current version the adapter only works on Linux, because the error-free execution of the shell script does NOT work on other distributions.

## Table of contents:
1st backup type
   - 1.1 Minimum Backup (Standard IoBroker Backup)
   - 1.2 Full backup
   - 1.3 CCU Backup (original CCU / pivCCU / Raspberrymatic)
   - 1.4 Optional mysql backup (local host)

2. preparation

3. Ftp vs. CIFS

4. Use
   - 4.1 Created data points
   - 4.3 Formatting the History Log with CCS
   - 4.4 Displaying the backup status in the OneClick button

5. Restore a backup
   - 5.1 Restoring a minimal backup
   - 5.2 Restore full backup
   - 5.3 Restore Raspberrymatic/CCU backup

6 Troubleshooting
   - 6.1 Activate logging
   - 6.2 Enable debugging

7. Occurred errors / solutions
   - 7.1 Web interface not accessible after restore
   - 7.2 JS data point cannot be written to
   - 7.3 Error message: "Command not found
   - 7.4 Complete backup hangs
   - 7.5 Changed values in Dp are not accepted

8. Changelog


## 1. Backup types:

Backitup offers the possibility to perform three different backup types (optionally with DB backup) cyclically or at the push of a button. By default, each backup is stored in the /opt/iobroker/backups/ directory. Optionally, an FTP upload can be set up or alternatively a CIFS mount can be used.

1. Standard backup
   - This backup corresponds to the backup contained in IoBroker which can be started in the console by calling"./iobroker backup". However, here it is performed by the settings defined in the adapter configuration or the OneClick Backup widget without having to use the console.
2. Full backup
   - This backup backs up the complete IoBroker folder including all subfolders and their files including file permissions. The file size should not be ignored, as such a backup often has several hundred MB.
To make sure that all current states are backed up, you have to set this in the configuration of the IoBroker Stop/Start hacks.
3. CCU Backup (Homematic)
   - This backup offers the possibility to backup 3 different variants of a Homematic installation (CCU-Original / pivCCU / Raspberrymatic). This backup can also be performed using the settings defined in the adapter configuration or the OneClick Backup widget.
4. mysql backup (local host)
   - If activated, this separately adjustable backup is created for each backup, regardless of whether "minimal" or "complete", and is also deleted after the specified retention time has elapsed. FTP or CIFS are also valid for this backup if set for the other IoBroker backup types.

## 3. prep:

The following steps should be performed to use the adapter (if the backup script v1/v2/v3 was used, first delete everything (disable or delete Data Points/Enum.functions/Shell-Script and JavaScript!)


## 3. Use Ftp service or CIFS for optional backup to a Nas?

  - Advantages of CIFS:
    - Fewer write cycles on your disk (possibly relevant if Raspberry with SD card is used to protect it)
    - It is possible to have the "old backups" automatically deleted on the nas
  - Disadvantages of CIFS:
    - If mounting is not possible, no backup is created!
    - Old backups" can be automatically deleted on the Nas. In the worst case there is no backup available if you need it.
  - Path information (note spelling):
    - CIFS: "Share Name/Path Specification
    - FTP:"/Path specification
  - Enter optional port for FTP upload in the NAS settings under IP address:
    - IP Address:Port


## 4. Use:

The adapter creates 7 data points for use in Vis
	- oneClick.ccu -> serves as trigger for a CCU backup (can be set to true in Vis by a button)
	- oneClick.minimal -> serves as trigger for a standard backup (can be set to true in Vis by a button)
	- oneClick.total -> serves as trigger for a complete backup (can be set to true in Vis by a button)

	- history.html -> diehnt as history log which is customizable in Vis via CCS from the design.
	- history.ccuLastTime -> saves the creation date and time of the last CCU backup
	- history.minimalLastTime -> saves the creation date and time of the last standard backup
	- history.totalLastTime -> saves the creation date and time of the last full backup

2. show history log in Vis
   - It is possible to display the history log in an html widget by entering the following line in HTML:

```
{backitup.0.history.html}
```
Syntax: {BackitupInstance.history.html}


3. CCS formatting of the history log
```
   .backup-html{
       display:block;
       width:100%;
   /*    overflow-y:scroll; */
   }
   .backup-type-minimal
       {
           float: left;
           color: white;
           font-size: 18px;
       }
   .backup-type-total
       {
           float: left;
           color: yellow;
           font-size: 18px;
       }
   .backup-type-ccu
       {
           float: left;
           color: red;
           font-size: 18px;
       }
   ```
4. OneClick button with status text
   - If a OneClick data point is set to true the corresponding backup starts and after a predefined time this data point is set to false again so it is possible to create a button with status, adjust the following line and enter it in Vis as button text:
```
{value: backitup.0.oneClick.minimal; value === "true" || value === true ? "Minimal Backup </br> will be created" : "Minimal Backup </br> starten"}

```
Syntax: {value: <BackitupInstance>.oneClick.<trigger>; value ==="true" || value === true ? "Text during backup creation" : "Standard text"}

## 5. Restore:

1. restore a minimal / normal IoBroker backup:
    - The backup must be in the "opt/iobroker/backups/" directory as usual
    - It can be restored from the console using the command: "iobroker restore (number of the backup from the list)".

2. restore a complete backup:
    - Run the command: "sudo iobroker stop" from the console
    - The created backup must be copied to the "root/" directory
    - Execute the command:" sudo tar -xzvf Backupname.tar.gz -C / " from the console
    - Wait - During recovery you will be shown what is currently being done
    - Run the command: "sudo iobroker start" from the console

3. restore a Raspberrymatic / CCU backup:
    - Copy *.sbk file via SCP into the directory " /usr/local/tmp directory" on the Raspberrymatic
    - Log in via the console as root user on the Raspberrymatic
    - Run the command: "/bin/restoreBackup.sh /user/local/tmp/EuerBackupFilename" on the Raspberrymatic.
    - Execute the command: "reboot" on the Raspberrymatic to restart the PI

Alternatively, the backup can be restored via the web interface as usual.

## 6. Troubleshooting:

1. In the adapter configuration there is the possibility to activate log, so the IoBroker log lists different messages (e.g. backup times and states) which can be used for troubleshooting

In addition there is the possibility to activate debug now the command which is passed to the backitup.sh is displayed in the IoBroker log. This command can be entered one to one into the console (with Putty or similar) to limit errors.

## 7. occured errors / solutions:

Here is a list of the problems encountered so far and their solutions, if any.

1. olifall (from the forum) had the problem that after the restore the web interface of the IoBroker was no longer accessible, he could fix this by following steps via the console:
    - sudo iobroker status
    - Message ="No connection to states 127.0.0.0.0:6379[redis]"
    - sudo apt-get install redis-server

2. during testing some data points could not be described / changed by others, this error could not be readjusted and therefore could not be corrected.

3. error message: "Command not found
Due to the differences between Unix and Windows, the backitup.sh may not be changed under Windows (Editor).
Explanation:
Under DOS, a line end is represented in text files by the sequence return (decimal code 13) and new line (decimal code 10). Unix, on the other hand, only uses new line.

4. iobroker hangs during complete backup / no longer starts
Some users reported that the IoBroker complete backup does not run correctly or that the IoBroker is stopped and not started anymore. For this purpose it is possible to deactivate the stop/start of the IoBroker during a complete backup in the adapter configuration data points.
