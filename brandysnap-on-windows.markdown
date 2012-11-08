only works with cygwin. -- because rsync only runs on cygwin (I think)
and brandysnap uses cygwin features such as Unix-like signals.

Cygwin's own perl is recommended -- may work with others.

scheduled -- as here:

http://www.davidjnice.com/articles/cygwin-scheduled-tasks.html

but that pops up a window



If the destination filing system doesn't support hard links, then
each snapshot will be full size -- brandysnap doesn't check that
hard links will work, it's up to you.


Setting up Cygwin
-----------------

See http://www.gaztronics.net/rsync.php


cygdrive/c ACL
--------------
See http://www.cygwin.com/cygwin-ug-net/using.html#mount-table



