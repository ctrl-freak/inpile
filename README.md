# inpile
Monitor a directory for new files, move those files out elsewhere, softlink and extract using inotifyd/incrontab, rsync (and dtrx).

## Requirements
`sudo apt install inotify-tools incron rsync`

Recommended:
`sudo apt install dtrx`

## Installation

Add the current user (or the preferred user) to the incron allowed users

    echo `whoami` | sudo tee -a /etc/incron.allow

Copy the `inpile` command/script to somewhere in `$PATH`, like `/usr/local/bin`

## Usage
`$ inpile -h`

    inpile [-h] [-c <configfile>] -f <sourcedir> -s <sourcefile> -d <destination dir>

## Example

You have a directory `/media/storage/synced` which you would like to pull files out of an unzip into another folder `/media/storage/library`

1. Create the configfile using the Example below.
2. Create a incrontab entry (`incrontab -e`):

        /media/storage/synced IN_CLOSE_WRITE inpile -c /media/storage/sync-pause.conf -f $@ -s $@/$# -d /media/storage/library

3. Check syslog for changes: `sudo tail -f /var/log/syslog`. You may need to create a change to initiate the first sync.

### Example Result

    user@host:/media/storage$ tree
    .
    ├── library
    ├── sync-pause.conf
    └── synced
        ├── source.txt
        └── testdir
            └── test2
    user@host:/media/storage$ touch source/test.txt
    user@host:/media/storage$ tree
    .
    ├── library
    │   ├── testdir
    │   │   └── test2
    │   └── test.txt
    ├── sync-pause.conf
    └── synced
        ├── source.txt
        ├── testdir
        │   └── test2
        └── test.txt -> /media/storage/library/test.txt
    user@host:/media/storage$ vim synced/source.txt
    user@host:/media/storage$ tree
    .
    ├── library
    │   ├── source.txt
    │   ├── testdir
    │   │   └── test2
    │   └── test.txt
    ├── sync-pause.conf
    └── synced
        ├── source.txt -> /media/storage/library/source.txt
        ├── testdir
        │   └── test2
        └── test.txt -> /media/storage/library/test.txt



## Configuration

Variables available for use in configfile:
- $sourcedir
- $sourcefile
- $destination
- $newfile

## Example configfile `sync-pause.conf`

    before="mega-sync -s 0"
    after="mega-sync -r 0; if [ '$sourcefile' != '' ]; then dtrx -r -q -n $newfile; fi;"

