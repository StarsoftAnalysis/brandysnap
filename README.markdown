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

--------------------------------------------------------------
This is part of the brandysnap documentation.
Copyright (C) 2011  Chris Dennis  chris@starsoftanalysis.co.uk
See the file fdl-1.3.txt for copying conditions.
--------------------------------------------------------------
 
