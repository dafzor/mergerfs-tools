# mergerfs-tools

Optional tools to help manage data in a mergerfs pool.

## INSTALL

All of these suplimental tools are self contained Python3 apps. Make sure you have Python 3 installed and either run `make install` or copy the file to `/usr/local/bin` or wherever you keep your binarys and make it executable (chmod +x).

## TOOLS
### mergerfs.ctl

A wrapper around the mergerfs xattr interface.

```
$ mergerfs.ctl -h
usage: mergerfs.ctl [-h] [-m MOUNT] {add,remove,list,get,set,info} ...

positional arguments:
  {add,remove,list,get,set,info}

optional arguments:
  -h, --help            show this help message and exit
    -m MOUNT, --mount MOUNT
                            mergerfs mount to act on
$ mergerfs.ctl info
- mount: /storage
  version: 2.14.0
  pid: 1234
  srcmounts:
    - /mnt/drive0
    - /mnt/drive1
$ mergerfs.ctl -m /storage add path /mnt/drive2
$ mergerfs.ctl info
- mount: /storage
  version: 2.14.0
  pid: 1234
  srcmounts:
    - /mnt/drive0
    - /mnt/drive1
    - /mnt/drive2
```

### mergerfs.fsck

Audits permissions and ownership of files and directories in a mergerfs mount and allows for manual and automatic fixing of them.

It's possible that files or directories can be duplicated across multiple drives and that their metadata become out of sync. Permissions, ownership, etc. This can cause some strange behavior depending on the mergerfs policies used. This tool helps find and fix those inconsistancies.

```
$ mergerfs.fsck -h
usage: mergerfs.fsck [-h] [-v] [-s] [-f {manual,newest,nonroot}] dir

audit a mergerfs mount for inconsistencies

positional arguments:
  dir                   starting directory

  optional arguments:
    -h, --help            show this help message and exit
    -v, --verbose         print details of audit item
    -s, --size            only consider if the size is the same
    -f {manual,newest,nonroot}, --fix {manual,newest,nonroot}
                          fix policy
$ mergerfs.fsck -v -f manual /path/to/dir
```

### mergerfs.dup

Duplicates files & directories across drives in pool copying files to drives with the most free space.

Run as `root`. Requires `rsync` to be installed.

**Important Warning:** This tool is meant to only be used in archival data that never changes. Any change to a file that's been duplicate will not be replicated in it's copy even though mergerfs will update it's modified timestamp. This means you'll have two versions of the file and the one visable in the mergerfs pool can change on remount, aditionaly using mergerfs.dedup may cause loss of data since it will the first file it sees instead of the oldest has it only checks for timestamp and not content.


```
usage: mergerfs.dup [-h] [-c COUNT] [-i INCLUDE] [-e EXCLUDE] dir

Duplicate files & directories across multiple drives in pool

positional arguments:
  dir                   starting directory

optional arguments:
  -h, --help            show this help message and exit
  -c COUNT, --count COUNT
                        number of versions
  -i INCLUDE, --include INCLUDE
                        fnmatch compatible file filter (can use multiple times)
  -e EXCLUDE, --exclude EXCLUDE
                        fnmatch compatible file filter (can use multiple times)
```


### mergerfs.dedup

Finds and deduplicate files.

**Important Warning:** This tool only compares timestamps and not file content. Updating a duplicate file in a mergerfs pool will change the content of a single file but the timestamp of all copies. Running dedupping in such conditions may cause dataloss.

```
# mergerfs.dedup -h
usage: mergerfs.dedup [-h] [-v] [-i]
                      [-d {manual,newest,largest,mostfreespace}] [-e]
                      [-I INCLUDE] [-E EXCLUDE]
                      dir

dedup files on a mergerfs mount

positional arguments:
  dir                   starting directory

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         print additional information: use once for duped
                        files, twice for file info
  -i, --ignore          use once to ignore duped files of the same size, twice
                        to ignore files with matching md5sums
  -d {manual,newest,largest,mostfreespace}, --dedup {manual,newest,largest,mostfreespace}
                        dedup policy: what file to keep
  -e, --execute         without this it will dryrun
  -I INCLUDE, --include INCLUDE
                        fnmatch compatible filter (can use multiple times
  -E EXCLUDE, --exclude EXCLUDE
                        fnmatch compatible filter (can use multiple times

# mergerfs.dedup /path/to/dir
Total savings: 38.0GB

# mergerfs.dedup -e -d manual /path/to/dir
...
```


### mergerfs.balance

Will move files from the most filled drive (percentage wise) to the least filled drive. Will do so till the most and least filled drives come within a user defined percentage range (defaults to 2%).

Run as `root`. Requires `rsync` to be installed.

```
usage: mergerfs.balance [-h] [-p PERCENTAGE] [-i INCLUDE] [-e EXCLUDE]
                        [-I INCLUDEPATH] [-E EXCLUDEPATH] [-s EXCLUDELT]
                        [-S EXCLUDEGT]
                        dir

balance files on a mergerfs mount based on percentage drive filled

positional arguments:
  dir                   starting directory

optional arguments:
  -h, --help            show this help message and exit
  -p PERCENTAGE         percentage range of freespace (default 2.0)
  -i INCLUDE, --include INCLUDE
                        fnmatch compatible file filter (can use multiple
                        times)
  -e EXCLUDE, --exclude EXCLUDE
                        fnmatch compatible file filter (can use multiple
                        times)
  -I INCLUDEPATH, --include-path INCLUDEPATH
                        fnmatch compatible path filter (can use multiple
                        times)
  -E EXCLUDEPATH, --exclude-path EXCLUDEPATH
                        fnmatch compatible path filter (can use multiple
                        times)
  -s EXCLUDELT          exclude files smaller than <int>[KMGT] bytes
  -S EXCLUDEGT          exclude files larger than <int>[KMGT] bytes

# mergerfs.balance /media
from: /mnt/drive1/foo/bar
to:   /mnt/drive2/foo/bar
rsync ...
```

### mergerfs.mktrash

Will create [FreeDesktop.org Trash specification](https://specifications.freedesktop.org/trash-spec/trashspec-1.0.html) compatible directories on a mergerfs mount. Helps minimize issues with apps which `rename` into the trash directory. This shouldn't be necessary if you're not using a path perservation policy.

```
$ mergerfs.mktrash /mountpoint
```

## SUPPORT

#### Contact / Issue submission
* github.com: https://github.com/trapexit/mergerfs-tools/issues
* email: trapexit@spawn.link
* twitter: https://twitter.com/_trapexit

#### Support development

This software is free to use and released under a very liberal license. That said if you like this software and would like to support its development donations are welcome.

* Bitcoin (BTC): 12CdMhEPQVmjz3SSynkAEuD5q9JmhTDCZA
* Bitcoin Cash (BCH): 1AjPqZZhu7GVEs6JFPjHmtsvmDL4euzMzp
* Ethereum (ETH): 0x09A166B11fCC127324C7fc5f1B572255b3046E94
* Litecoin (LTC): LXAsq6yc6zYU3EbcqyWtHBrH1Ypx4GjUjm
* Ripple (XRP): rNACR2hqGjpbHuCKwmJ4pDpd2zRfuRATcE
* PayPal: trapexit@spawn.link
* Patreon: https://www.patreon.com/trapexit

## LINKS

* http://github.com/trapexit/mergerfs
* http://github.com/trapexit/mergerfs-tools
* http://github.com/trapexit/scorch
* http://github.com/trapexit/backup-and-recovery-howtos
