# ["/usr/bin/python^M: bad interpreter" [duplicate]](https://stackoverflow.com/questions/9975011/usr-bin-pythonm-bad-interpreter)


The issue is not EOF but EOL. The shell sees a ^M as well as the end of line and thus tries to find `/usr/bin/python^M` .

The usual way of getting into this state is to edit the python file with a MSDOS/Windows editor and then run on Unix. The simplest fix is to run dos2unix on the file or edit the file in an editor that explicitly allows saving with Unix end of lines.


You may find the answers here: [./configure : /bin/sh^M : bad interpreter](https://stackoverflow.com/questions/2920416/configure-bin-shm-bad-interpreter)

As a Mac OS X user, I didn't find the command `dos2unix`. Alternatively, I use vi/vim: `:set fileformat=unix` and then save the file `:wq`