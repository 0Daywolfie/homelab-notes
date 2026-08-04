# Linux Mint Boot Failure - Filesystem Corruption Recovery via fsck

## Problem
Laptop failed to boot normally into Linux Mint and dropped into a 
BusyBox (initramfs) recovery shell instead - the kernel could not 
automatically mount the root filesystem. Likely cause: unclean 
shutdown or system freeze during a prior session (system was 
actively running Packet Tracer + VirtualBox at the time).

## Diagnosis
From the initramfs prompt:
- Confirmed disk partitions were still present and detected:
  `ls /dev/sd*` showed /dev/sda, /dev/sda1, /dev/sda2
- `fdisk -l` was unavailable in the minimal BusyBox environment 
  (fdisk not included) - used `ls /dev/sd*` instead to identify 
  partitions
- Identified /dev/sda2 as the likely root filesystem (larger 
  partition) based on standard Mint/Ubuntu partition layout

## Solution
Ran a targeted filesystem check against the identified root partition:

    fsck /dev/sda2

fsck (util-linux 2.39.3, e2fsck 1.47.0) ran a full 5-pass check:
- Pass 1: Checking inodes, blocks, and sizes
  - Found and FIXED a corrupted orphan inode linked list
  - Found and FIXED inodes that were part of the corrupted orphan list
- Pass 1E: Optimizing extent trees
- Pass 2: Checking directory structure
- Pass 3: Checking directory connectivity
- Pass 4: Checking reference counts
- Pass 5: Checking group summary information
  - Fixed multiple "free blocks count wrong" and "free inodes count 
    wrong" bitmap mismatches (bookkeeping errors, not data loss)

Answered "yes" (y) to each Fix prompt fsck presented - this is the 
expected, safe response for these repair prompts and does not 
delete or damage existing data.

Final output confirmed: `/dev/sda2: FILE SYSTEM WAS MODIFIED` with 
a summary showing ~1.5 million files scanned successfully.

Rebooted after the fsck pass completed - system booted normally 
into Linux Mint with no data loss.

## Root cause understanding
Orphaned inodes and free-space bitmap mismatches are classic 
symptoms of an unclean shutdown (crash, freeze, forced power-off) 
rather than physical disk failure. The filesystem's internal 
bookkeeping gets out of sync with actual disk state when writes 
are interrupted mid-operation. fsck's job is specifically to 
reconcile that bookkeeping against the real on-disk state.

## Key lesson
- `initramfs`/BusyBox recovery mode is not data loss - it's the 
  kernel refusing to mount an inconsistent filesystem until it's 
  verified safe
- `fsck` requires an explicit target partition - running it with 
  no argument does nothing useful
- Answering "yes" to fsck Fix prompts is the correct, safe response 
  in the vast majority of cases - these are consistency repairs, 
  not deletions
- Always verify which partition is root before running repair tools 
  blind - `ls /dev/sd*` works even when `fdisk` isn't available in 
  minimal recovery environments

## Verification
- System booted normally into Linux Mint post-repair
- Confirmed homelab-notes and ccna-study-vault Git repos both intact 
  with clean working trees post-recovery
