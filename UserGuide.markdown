
Brandysnap User Guide
=====================

(expanded version of the Quick Start guide)

This is preliminary documentation for the [brandysnap](http://www.brandysnap.org) script, version 0.1.4, 4 July 2011.

**************************************************************
This is part of the brandysnap documentation.</br>
Copyright &copy; 2011  Chris Dennis  chris@starsoftanalysis.co.uk</br>
See the file fdl-1.3.txt for copying conditions.
**************************************************************

Introduction
------------

Brandysnap is (yet another) script that uses rsync to create snapshot backups of files and directories.

It is still being developed, and should be considered very experimental software.  Use it at your own risk!

It is designed to be flexible and robust.  In particular, it copes well with situations where snapshots are not created as regularly as they should be.  This can be for a variety of reasons -- hardware failure, operator failure, weekends, holidays, someone forgot to plug in the external USB drive...

Rather than assigning 'importance' to snapshots according to when they were made (for example, keeping Friday afternoon backups for longer than Tuesday morning ones), it looks at what snapshots exist and decides if any can be deleted.  If the Friday afternoon backup is missing, it will keep the nearest one it can find.

The basic rule is 'backup first, ask questions later'.  Brandysnap assumes that snapshots are cheap -- in both time and space.  In other words, making a snapshot is quick, and uses relatively little disk space because of the use of hard links.  (Of course the first snapshot is not cheap.  And snapshots over a network are never particularly quick.)

It therefore creates a new snapshot every time it is run, and then thinks about which snapshots, if any, are no longer required.

Brandysnap tries to cope with any situation.  Either the source or the destination can be on a remote computer (but not both at the same time).

Brandysnap is designed to be run regularly from cron (or similar automation system).  It can also be run manually if required without upsetting things.

Brandysnap has been developed on Linux.  It is written in Perl and uses a selection of fairly standard modules (all available in Ubuntu repositories, and no doubt elsewhere too).  It relies on hard links [see ...] and will only work properly when creating snapshots on a filing system that can handle hard links, such as ext3 or ext4 [and others...].

Lots of snapshots don't make a file more secure: if the file hasn't changed, and is the same in all snapshots, then it is only stored once on the disk.  If it gets corrupted for any reason, then it will be corrupted in all snapshots.  (If the file is DELETED from one snapshot, it will still exist in the others, because of the way hard links work.) Think of the whole set of snapshots as one backup, with multiple versions of files that have changed.


Very Quick Start
----------------

See the README file for brief installation instructions.

If you want to try it out without reading further, create a configuration file called brandysnap-test1.conf containing something like this:

    source      = /path/to/something/to/backup
    destination = /path/to/writable/directory
    template    = brandysnap-test1
    spec        = 4h1,1h1,1d7
    lockfile    = ~/brandysnap-test1.lock

and then run

    brandysnap --dry-run --conf brandysnap-test1.conf

The `dry-run` option will mean that nothing will actually be done.  Once you're happy than the script won't break anything, try it without the `dry-run`.    

For more options, run `brandysnap --help`.


Remote Sources and Destinations
-------------------------------

Brandysnap assumes that any messing about with passwords has been taken care of.

In other words, it will call rsync on remote directories without supplying a password.

There are various ways of setting up rsync so that it doesn't prompt for a password -- they are not described here.  [ a reference or two would be helpful here] So it's important to get the password-less access sorted out first.  In fact rsync may be run many times during one run of brandysnap, so suppand rsync would prompt for a password each time.

Multiple destinations can be configured, in which case snapshots will be created on all destinations that are available.  This can be used to put snapshots on several external USB drives in rotation, for example.  Or you could specify one local and one remote destination for security.  But note that multiple destinations won't be in sync with each other that's not the intention.  They will end up with different lists of snapshots if they are available at different times


Preserving Ownership
--------------------

In order to preserver ownership of files, brandysnap needs to be run as root or via 'sudo'.

[More detail needed here re use of sudo and `remote-rsync-cmd = 'sudo rsync'`]


<a name="keepingSnapshots">Keeping Snapshots</a>
------------------------------------------------

The number of snapshots that are made depends on how often brandysnap is run.  It creates a new snapshot every time.  What is more interesting is how many snapshots are KEPT.  This is 
determined by a series of specifications, or 'specs', such as

	4d5,3w3,1d11

which is a concise way of saying 'keep four snapshots a day from the last five days, then three a week for three weeks, then one a month for eleven months.  If brandysnap has run successfully for the last year, then there will be at least 4x5 + 3x3 + 1x11 = 40 snapshots.

Most of the tricky stuff in brandysnap is in deciding which snapshots to keep.

Each spec is of the form

	<frequency><period><count>

The 'frequency' is the number of snapshots to be kept in each period.  It can be any number from 1 to ...whatever is reasonable.  Note that this is not the number of snapshots that will be CREATED -- that is determined simply by how often brandysnap is run, and that will usually be down to the way that cron is configured.

The 'period' is a single letter indicating the time period.  It can be one of

* h - hour	
* d - day
* w - week
* m - month
* y - year

The period can be given in either upper or lower case.

The 'count' indicates the number of periods, as a number from 1 to as many as you like.

If the count is left out, the period is 'padded' to make up to the next period, working backwards in time from 'now'.  For example, 

	4d,2w4

will be interpreted as `4d7,2w4`.  The 'day' specification is expanded to a week's worth of days to align with the next spec which is in weeks.

If the last spec has no count, it will be padded 'forever'.  The number of snapshots will only be limited by the available disk space.  And when the disk is full, the oldest snapshots will be deleted.

More spec examples:

* `1d` - just keep 1 backup every day, with no limit to the number of backups.
* `1h24,4d6,3w3,4m11` - one an hour for the first day, then 4 a day for the rest of the week then 3 a week for the rest of the month, then 4 a month to give a whole year of snapshots.
* ...

Snapshots also get deleted as time passes.  If a day with four snapshots gets to old enough to fall within a `3w` spec, then the extra snapshots will be deleted.


Definition of 'snapshot' vs full/incremental backups
----------------------------------------------------
Lots of snapshots don't make a file more secure: if the file hasn't changed, and is the same in all snapshots, 
then it is only stored once on the disk.  If it gets corrupted for any reason, then it will be corrupted
in all snapshots.  (If the file is DELETED from one snapshot, it will still exist in the others, because
of the way hard links work.)

Think of the whole set of snapshots as one backup, with multiple versions of files that have changed.


Options
-------

All options can be given either on the command line or in the configuration file.  Command line options override configuration file ones.  They are case-insensitive.

On the command line, options must be preceded by one or two hyphens, and can be abbreviated as long as they do not become ambiguous.  An 'equals' sign (`=`) is optional.  For example:

    brandysnap --source xyz -verbose=1 --conf=bs1.conf -cal false

In the configuration file, leave off the hyphens and don't abbreviate the options.  Lines beginning with '#' are considered to be comments and are ignored.  [These differences between the command line and the configuration file will be fixed...]

Some options (such as `source` and `destination`) can be specified more than once.  Note that command line options _always_ override configuration file ones, even for multiple options.  So if you have a list of `exclude` options in the configuration file, and try to add an extra one on the command line, those in the file will be ignored.


'`~`' can be used to specify local files and directories e.g.

    --logfile = ~/brandysnap.log

The '`~`' will be expanded to the home directory of the user who _runs_ brandysnap.  
'`~`' can also be used on remote directories, e.g. `chris@example.com:~/documents`.  In this case, the '`~`' will be expanded by rsync to mean the home directory of the user specified (or implied) before the '`@`' symbol, in this case '`/home/chris/`'.
'`~`' can NOT be used in any of the `include`/`exclude` options.

* Options marked with '!' in the following list are required.
* Options marked with '*' in the following list can be specified more than once.
* '`yes/no`' options can be specified as any of `yes`/`true`/`on`/`1` or `no`/`false`/`off`/`0`.

### Main options

#### `config <file>` !
The name of a file to look in for further options.  Configuration file options will be overridden by command-line ones, irrespective of where the `config` option appears on the command line.

#### `source <file/dir>` *!
A local or remote file or directory to add to the snapshot.  Examples:

	source ~/Documents
	source /home
	source chris@example.com:~/Documents

More than one source can be specified, in which case each source will be rsync'd, one at a time, to each destination in turn.
Rsync can not copy from a remote source to a remote destination, so any source/destination pairs which are both remote will be skipped.
Each source must be readable by the user who runs brandysnap.  If any files or directories within the source are not readable, brandysnap will carry on regardless.

See the section on remote authorisation.

By default, brandysnap uses the rsync options `--archive --hard-links --one-filesystem`, so the whole of each source will be copied recursively without following symbolic links.  See the `rsync-options` option for ways to change this.

#### `destination <dir>` *!
A local or remote directory for use as the snapshot destination.  Examples:

	destination /backups/
	dest chris@example.com:/backups

More than one destination can be specified (see `source`).  

Each destination must be writable by the user who runs brandysnap.

See the section on remote authorisation.

#### `template <name>` !
The directory name of each snapshot is of the form

	<template>-<timestamp>

See the [Snapshot Names section](#snapshotNames) for more details.
Example:

	template docs

#### `spec <string>` !   
The snapshot-keeping specification.  See the [Keeping Snapshots section](#keepingSnapshots) for full details.

#### `lockfile <file>` !
To prevent separate runs of brandysnap using the same destinations at the same time, you need to give 
the name of temporary file which will be created and locked while brandysnap is running.
The user running brandysnap must have permission to create and delete this file.  For example:

	lockfile /tmp/brandysnap-docs.lock

#### `logfile <file>`
The name of a file which will be used to log the output from brandysnap.  Examples:

	logfile /var/log/brandysnap.log
	logfile ~/bs-docs.log

The user running brandysnap must have permission to create and write to the log file.

### Tuning options

#### `calendar <yes/no>`
In calendar mode, hours start on the hours, days start at midnight, weeks start on Sunday (but see the `weekstart` option), months start on the 1st of the month, years start on the 1st of January.  Padding is added where necessary to align periods with the calendar.  When calendar mode is turned off, periods are not aligned and are contiguous, ending 'now'.  See the [Calendar Mode section](#calendarMode) below for further details.  (default: `yes`)

#### `safe <yes/no>`
In safe mode, snapshots are only considered for deletion if the specified periods are 'complete' -- i.e. they have the required number of snapshots.  If safe mode is turned off, all periods are considered complete, and extra snapshots in any of them will be deleted. See the [Safe Mode section](#safeMode) below for further details.  (default: `yes`)

The `xbest` options can be used to tune the snapshot-matching algorithm which decides which snapshots should be deleted.  The defaults assume that the latest snapshots within a period are the most valuable, and should be kept.  Note that if calendar mode is turned off, the `xbest` options are relative to the start of the period: for example `wbest = 3` means the middle of the week, even if the week happens to start at 5:30am on a Tuesday.

#### `hbest <0..59>`     
`hbest` determines the favoured minute within an hour for an hourly specification. For example, to prefer hourly snapshots created in the middle of an hours, use `hbest 30`.  (default: `59`)

#### `dbest <0..23.9>`
Determines the favoured time within day in hours.  For example, to prefer daily snapshots created at 5pm, use `dbest 17`. (default: `23.9`)

#### `wbest <1..7>`
Determines the favoured day within a week, with 1=Sunday, 7=Saturday.  For example, to prefer weekly snapshots created on Friday, use `wbest 6`. (default: `1`)

#### `mbest <1..31>`
Determines the favoured day within a month.  For example, to prefer monthly snapshots created at the beginning of the month, use `mbest 1`.  [This may be improved in the future to allow preferences such as 'the last Friday in the month'. If the value specified is greater than the number of days in a particular month, the last day of the month is used.  To always select the last day of the month, use `mbest 31`.  (default: `31`)

#### `ybest <1..366>`
Determines the favoured day within a year.  In leap years, the value `366` is automatically changed to `365`, so `366` always means 'the last day of the year'. For example, to prefer yearly snapshots in the middle of the year, use `ybest 180`. (default: `366`)

#### `weekstart <1..7>`
Sets the first day of week.  If you consider that weeks start on Monday, use `weekstart 2`.  `1`=Sunday, `7`=Saturday.  (default: `1`)

### Helpful options

#### `help <yes/no>`
Prints out a brief summary of options, and then stops. (default: `no`)

#### `version <yes/no>`
Prints out the brandysnap version number only and then stops. (default: `no`)

#### `verbose <0..3>`
This options sets the verbosity of the printed output, on a scale from `0` to `3`.  Use higher values to see more about what brandysnap and rsync are doing.  (default: `1`)

#### `loglevel <0..3>`     
Sets the verbosity level of output in the log file, on a scale from `0` to `3`.  If no `logfile` is defined, this option is effectively set to `0`.  (default: `1`)

#### `dry-run <yes/no>`
In `dry-run` mode, brandysnap goes through the motions, but doesn't actually create or delete any snapshots.  The `dry-run` option is also passed through to rsync. (default: `no`)

### Rsync options

#### `rsync-cmd <path>`
The location of the rsync programme on your system.  The default is just `rsync` which means brandysnap looks for rsync in you normal path. On some systems, you might need to set it to something else such as

	rsync-cmd /usr/bin/rsync

(default: `rsync`)

#### `compress <yes/no>`
Enable rsync compression for remote transfers. Note that this only applies compression for transfer across the network: files are expanded again on the destination.  (default: `yes`)

#### `include`/`include-from`/`exclude`/`exclude-from <pattern-or-file>` *
These four options are passed through to rsync unchecked and unchanged.  '~' is NOT expanded to a home directory.  See the rsync documentation for details.  (default: none)

#### `bwlimit-in` <n>
Band-width limit for receiving in kbps.  Set it to 0 for no limit.  (default: `0`)

#### `bwlimit-out` <n>         
Band-width limit for sending in kbps. Set it to 0 for no limit.  (default: `0`)

#### `rsync-opts <options>`
Options to pass to rsync, in addition to those that brandysnap will always use (i.e. --relative and --link-dest). Use this only if you know what you are doing.  (default: -aHx --numeric-ids)

### Advanced options

#### `delete <yes/no>`
Delete no-longer-required snapshots.  If this option is turned off, brandysnap will create new snapshots but not delete any old ones. (default: `yes`)

#### `delete-cp <yes/no>`
Include the 'current period' when considering which snapshots to delete.  See the description of [current period](#currentPeriod) below. (default: `yes`)

#### `expire-old <yes/no>`
Consider _all_ snapshots (oldest first) as expirable to make room when the destination is full. (default: `no`)

#### `restart <yes/no>`
If a previous run of brandysnap was interrupted for any reason, use this option to re-do the same snapshot (simply by relying on rsync's ability to not copy files that have not changed).  Any files in the source that have changed since the previous run _will_ be updated.  If more than one destination is being used, rsync will be run for _all_ destinations, even if some of them completed successfully before.

#### `snapshot <yes/no>`
Create a new snapshot.  If this option is turned off, no new snapshot will be created during this run of brandysnap but old snapshots may be deleted. (default: `yes`)

#### `status <yes/no>`
Print a status report only, with no snapshots being created or deleted. (default: `no`)

#### `strict <yes/no>`
Use strict mode -- see the [Strict Mode section](#strictMode). (default: `no`)

###Development options

These options are use by developers only.

#### `debug <0..3>`
Print and log debugging information. (default: `0`)

#### `stacktrace <yes/no>`
Print a stack trace on error. (default: `yes`)

#### `test <n>`
Run test case 'n'.

####  

<a name="calendarMode">Calendar mode</a>
----------------------------------------

In 'calendar mode', which is the default, brandysnap works in terms of real weeks and months.  Days always start at midnight, weeks at midnight on Sunday etc. (but see `--weekstart` option).  In non-calendar mode, the specs are interpreted more simply, working backwards from the moment when brandysnap is run.  There will be no gap between periods: days and weeks can start at any time, depending on when the previous spec ran out.

<a name="safeMode">Safe mode</a>
--------------------------------

In 'safe mode', which the default, specs will only match against the list of existing snapshots if there are enough snapshots to satisfy the spec's definition.  Incomplete specs will be skipped.  This has the result that brandysnap is less likely to delete snapshots.  This is designed to cater for situations when brandysnap has not run successfully as often as it should have, for whatever reason.  For example, because of weekends or holidays, or because the destination wasn't available because an external USB drive wasn't connected (or two or more USB drives are being used in rotation).  e.g if the spec is `4d5`, it's now Monday and brandysnap did not run at the weekend, then the days with fewer than 4 snapshots (i.e. Saturday and Sunday) will be skipped; counting the 5 days will start on Friday and work backwards from there.  Safe mode can be turned off via the `--safe` option.

Strict mode
-----------

In 'strict mode', which is not the default, brandysnap will not run if there are minor problems with the specs.  Normally, it will display information about how it has interpreted the specs, and carry on.

Weeks and months and years
--------------------------

The fact that months and years do not have whole or fixed numbers of 
weeks makes counting periods awkward.  Brandysnap deals with this by 
skipping over the extra days, and not deleting any of their snapshots. 

Status report
-------------

Brandysnap displays a status report on all existing snapshots at the end of each run.

However, if the destination is on a remote computer, the status report does include details of the disk space used by each snapshot because of the process of retrieving that information is slow.

The full status report can be seen for remote destinations by running brandysnap with the `--status` option in addition to the usual configuration.  And even then, can only display 'Real size', not 'Delete size', because rsync doesn't give information about the number of hard links.

<a name="snapshotNames">Snapshot names</a>
------------------------------------------

Each snapshot is a separate directory within the destination, with a name of the form

	<template>-<timestamp:YYYYMMDD-hhmmss>

where the 'template' is specified by the --template option.  For example

	bs1-20110616-121159

That format is fixed -- it is used to identify snapshots; any directory that doesn't match that pattern will be ignored.

***********************************************************

Known Issues
------------

As of 16 July 2011 and version 0.1.5, the following issues and bugs are known.

* The option `--remote-rsync-cmd` can only be specified once, and applies to all remote sources and directories.
* Under certain circumstances, rsync can fail when there are files that are hard-linked together
and for which you do not have read permission.  This is fixed in rsync 3.0.9 and later.  You can get
round it by specifying `--rsync-opts` with the usual options but omitting `--hard-links`.
* If the timings of snapshots are slightly out, a snapshot may appear to be in 
the wrong period, which will affect deletions.  
For example, when doing one snapshot a day with calendar mode turned off, if 
yesterday's snapshot was at 11:40:03, and today's is at 11:40:01, 
then yesterday's will be considered as part of today, 
because it is less than 24 hours ago, so it will be deleted.  A fix is planned to address this issue.
