# inpile
Monitor a directory for new files, move those files out elsewhere, softlink and extract using inotifyd/incrontab, rsync (and dtrx).

## Requirements
`sudo apt install inotify-tools incron rsync`

Recommended:
`sudo apt install dtrx`

## Installation

1. Add the current user (or the preferred user) to the incron allowed users

        echo `whoami` | sudo tee -a /etc/incron.allow

2. Copy the script to somewhere in `$PATH`, like `/usr/local/bin`

        sudo cp ./inpile /usr/local/bin/

3. Mark the script as executable:

        sudo chmod a+x /usr/local/bin/inpile

## Usage

    inpile [-h] [-c <configfile>] -f <sourcedir> -s <sourcefile> -d <destination dir>

## Example

You have a directory `/media/storage/synced` which you would like to pull files out of into another folder `/media/storage/library` (and extract).

1. Create the configfile using the [Example below](#example-configfile-sync-pauseconf).
2. Create a incrontab entry (`incrontab -e`):

        /media/storage/synced IN_CLOSE_WRITE,IN_MOVED_TO inpile -c /media/storage/sync-pause.conf -f $@ -s $@/$# -d /media/storage/library

3. Check syslog for changes: `sudo tail -f /var/log/syslog`. You may need to create a change to initiate the first sync.
4. If you do not get any updates, replace the line in `incrontab -e` with:

        /media/storage/synced IN_CLOSE_WRITE,IN_MOVED_TO echo "$@ $# $% $&"
5. Then monitor syslog for the appropriate action to monitor.

### Example Result

    user@host:/media/storage$ tree
    .
    ├── library
    ├── sync-pause.conf
    └── synced
        ├── source.txt
        └── testdir
            └── test2
    user@host:/media/storage$ touch synced/test.txt
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

Variables able to be set in configfile:
- before: Command to run before rsync, mv, and softlink
- after: Command to run after rsync, mv, and softlink
- sourcedir (not required if passed via script argument)
- destination (not required if passed via script argument)

Variables available for use in configfile:
- $sourcefile
- $newfile
- $sourcedir (if passed via script argument)
- $destination (if passed via script argument)


## Example configfile `sync-pause.conf`

    before="mega-sync -s 0"
    after="mega-sync -r 0; if [ '$sourcefile' != '' ]; then cd `dirname $newfile`; dtrx -r -q -n $newfile; fi;"
    #destination=
    #sourcedir=

