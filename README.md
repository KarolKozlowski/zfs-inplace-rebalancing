# zfs-inplace-rebalancing
Simple bash script to rebalance pool data between all mirrors when adding vdevs to a pool.

[![asciicast](https://asciinema.org/a/350222.svg)](https://asciinema.org/a/350222)

## How it works

This script recursively traverses all the files in a given directory. Each file is copied with a `.rebalance` suffix, retaining all file attributes. The original is then deleted and the *copy* is renamed back to the name of the original file. When copying a file ZFS will spread the data blocks across all vdevs, effectively distributing/rebalancing the data of the original file (more or less) evenly. This allows the pool data to be rebalanced without the need for a separate backup pool/drive.

Note that this process is not entirely "in-place", since a file has to be fully copied before the original is deleted. Therefore you have to have enough space to create a copy of the biggest file in your target directory for it to work.

At no point in time are both versions of the original file deleted.
Since file attributes are fully retained, it is not possible to verify if an individual file has been rebalanced.

## Prerequisites

### Balance Status

To check the current balance of a pool use:

```
zpool list -v
```

### No Deduplication

Due to the working principle of this script, which essentially creates a duplicate file on purpose, deduplication will most definitely prevent it from working as intended. If you use deduplication you probably have to resort to a more expensive rebalancing method that involves additional drives.

### Data selection

Due to the working principle of this script, it is crucial that you **only run it on data that is not actively accessed**, since the original file will be deleted.

## Usage

**ALWAYS HAVE A BACKUP OF YOUR DATA!**

```
chmod +x ./zfs-inplace-rebalancing.sh
./zfs-inplace-rebalancing.sh /pool/path/to/rebalance
```

Although this script **does** have a progress output (files as well as percentage) it might be a good idea to try a small subfolder first, or process your pool folder layout in manually selected badges. This can also limit the damage done, if anything bad happens.

When aborting the script midway through, be sure to check the last lines of its output. When cancelling before or during the renaming process a ".rebalance" file might be left and you have to rename it manually.

## Attributions

This script was inspired by [zfs-balancer](https://github.com/programster/zfs-balancer).

## Disclaimer

This software is provided "as is" and "as available", without any warranty.  
**ALWAYS HAVE A BACKUP OF YOUR DATA!**
