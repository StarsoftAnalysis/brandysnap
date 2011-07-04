Brandysnap
==========

Brandysnap is (yet another) script that uses rsync to create snapshot
backups of files and directories.

It is designed to be flexible and robust.  In particular, it copes well
with situations where snapshots are not created as regularly as they
should be.  This can be for a variety of reasons -- hardware failure,
operator failure, weekends, holidays, someone forgot to plug in the 
external USB drive...

Rather than assigning 'importance' to snapshots according to when they
were made (for example, keeping Friday afternoon backups for longer
than Tuesday morning ones), it looks at what snapshots exist and 
decides if any can be deleted.  If the Friday afternoon backup is
missing, it will keep the nearest one it can find.

The basic rule is 'backup first, ask questions later'.  Brandysnap 
assumes that snapshots are cheap -- in both time and space.  In other 
words, making a snapshot is quick, and uses relatively little disk 
space because of the use of hard links.

Brandysnap should be run regularly, via cron or something similar.  It 
creates a new snapshot each time it is run.

Then it looks at all existing snapshots to see if any old
snapshots can be deleted because they are no longer required, as
defined by the configured 'specification'.  

Requirements
------------

Brandysnap runs on Linux (it is currently being developed on Ubuntu 10.10).

It requires Perl 5 version 5.10.1 or later, and the following non-core modules:

* Math::Combinatorics	
* Config::General
* Filesys::DF

Installation
------------

There is not (yet) an installation script or package.  All you need to do is

1.  Download the script and all the related files by clicking on the
    'Downloads' button on https://github.com/StarsoftAnalysis/brandysnap
    and unpack the files into a suitable directory on your computer.

2.  Make sure you've got the necessary Perl modules, i.e. 

    * libmath-combinatorics-perl
    * libconfig-general-perl
    * libfilesys-df-perl

    which you can install with your favourite package manager, or from the 
    command line with something like:

        sudo apt-get install libmath-combinatorics-perl libconfig-general-perl libfilesys-df-perl

3.  Use a text editor to create a configuration file -- refer to the user guide for details.

4.  Run the script, with a command something like this:

        ./brandysnap --conf brandysnap-test1.conf --dry-run



**************************************************************
This is part of the brandysnap documentation.<br>
Copyright &copy; 2011  Chris Dennis  chris@starsoftanalysis.co.uk<br>
See the file fdl-1.3.txt for copying conditions.
**************************************************************
 
