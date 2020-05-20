# Rsync


!!! quote
    The swiss army knife of backups and file transfers

rsync is a utility for efficiently transferring and synchronizing files between two locations either locally or across networked computers. It is particularly useful because it facilitates fast incremental file transfer, ideal for backups.

## Examples
### Commonly used default options
``` bash
/usr/local/bin/rsync -avHP --exclude=.snapshot/ --exclude=.zfs /isilon/fs/vol1/hag/user1 /ironbank/gr/ >> user1.log &
```

`/usr/local/bin/rsync` instead of `rsync` so that my scripts dont have to worry about $PATH being set properly.

`a` - Archive mode preserving all attributes 

`v` - Verbose - listing even the names of files being transferred - for logging purposes. May be too much in some cases.

`H` - Preserve Hard links

`P` - keep partially transferred files and show progress

`--exclude` - Exclude a pattern. In my case Netapp's .snapshot format and ZFS's .zfs format to exclude copying snapshots. 

`>>` - Append to the log

`&` - Run this as a background process in shell.

### Replicate a directory tree at destination
``` bash
rsync -a --relative /new/x/y/z/ user@remote:/pre_existing/dir/
```
You get you will end up with /pre_existing/dir/new/x/y/z/

To replicate partial directory path, 
``` bash
rsync -a --relative /new/x/./y/z/ user@remote:/pre_existing/dir/
```
You get /pre_existing/dir/y/z/ at destination.