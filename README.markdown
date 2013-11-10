Brandysnap
==========

Brandysnap is (yet another) script that uses rsync to create snapshot
backups of files and directories.

It is designed to be flexible and robust.  In particular, it copes well
with situations where snapshots are not created as regularly as they
should be.  This can be for a variety of reasons -- hardware failure,
operator failure, weekends, holidays, someone forgot to plug in the 
external USB drive, etc.

Rather than assigning 'importance' to snapshots according to when they
were made (for example, keeping Friday afternoon backups for longer
than Tuesday morning ones), it looks at what snapshots exist and 
decides if any can be deleted.  If the Friday afternoon backup is
missing, it will keep the nearest one it can find.

The basic rule is 'backup first, ask questions later'.  Brandysnap 
assumes that snapshots (other than the first one) are cheap -- in both 
time and space.  In other 
words, making a snapshot is quick, and uses relatively little disk 
space because of the use of hard links.

Brandysnap should be run regularly, via cron or something similar.  It 
creates a new snapshot each time it is run (usually).
Then it looks at all existing snapshots to see if any old
snapshots can be deleted because they are no longer required, as
defined by the configured 'specification'.  

Features
--------

* It does everything via rsync, which means that either the source or 
the destination can be 'remote' (i.e. accessed via SSH or similar).  That also 
means that, if root access is required, it only has to be granted specifically
for rsync, not for the whole of brandysnap.

* It ignores anything other than files and directories, but preserves hard
and soft links. 

* It handles multiple sources and directories in one run.  The configuration file
can have 'sections' to ensure that, for example, exclusions are applied to the correct
source, and options for tweaking remote rsync access can be tailored to 
each site.

* The things it doesn't do are compression and deduplication on the destination.
There are no plans to include those features: they belong in the filing system.
btrfs, for example, already includes compression, and may do deduplication one day.
Nor does it do encryption.

Requirements
------------

Brandysnap has been developed on and runs on Linux.
I have heard that it also runs on Mac OS X (at least a previous version did).  Work to let 
it run on Windows in the Cygwin environment is under way.

It requires Perl 5 version 5.10.1 or later, and the following non-core modules:

* Date::Manip
* Math::Combinatorics	
* JSON

It also requires rsync, which should available in your version of Linux.
Brandysnap has been developed with rsync version 3.0.7 to 3.1.0.  It should work with 
any version of rsync that has the '--link-dest' option.


Installation
------------

There is not (yet) an installation script or package: brandysnap is a single Perl script
which just has to be run.  All you need to do is

1.  Download the script and all the related files by clicking on the
    'Downloads' button on https://github.com/StarsoftAnalysis/brandysnap
    and unpack the files into a suitable directory on your computer.

2.  Make sure you've got rsync and the necessary Perl modules as described above.

3.  Use a text editor to create a configuration file: refer to the documentation (i.e. 
    brandysnap.html or brandysnap.pod) for details.

4.  Run the script, with a command something like this:

        ./brandysnap --conf brandysnap-test1.conf --dry-run


Future Plans
------------

I have ideas for a few brandysnap-related scripts that would facilitate
searching for versions of backed-up files, restoring selected files from
brandysnap snapshots, etc.

But first I need to restructure some of the code into a Perl module 
that the new scripts could access.

Alternatives
------------

Brandysnap is one of a number of rsync-based scripts for managing snapshots.
LBackup is another, and the [LBackup website](http://www.lbackup.org/) has a list 
of [other alternatives](http://www.lbackup.org/alternatives).

**************************************************************
This is part of the brandysnap documentation.<br>
Copyright &copy; 2011-2013  Chris Dennis  chris@starsoftanalysis.co.uk<br>
See the file fdl-1.3.txt for copying conditions.
**************************************************************
 
Last modified: 28 October 2013
