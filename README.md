# linux-hard-drive-tester
This is a set of scripts for testing hard drives. These scripts were designed to run on a server that had a large RAID array running in JOBD.  These scripts were made to test hundreds of drives for a production situation. If you are a large company who wants to test large amount of drives, then feel free to use these scripts.  

For installing, it is best to set the proper permissions for root (like chmod 500 or 700) for each of these scripts. Then put them all into a folder that is part of your PATH, like /usr/bin. Be sure to run them in root, because the commands they use require you to be in root

These scripts use smartctl, badblocks, dd, fio, parted, and fdisk. Be sure to have those programs installed.  

The purpose of these scripts is to run a surface scan on the hard drives, run fio to do a speed test on the drives, to run smartctl to grab some SMART info about the drives, and to fully "zero wipe" the drives. It technically does not zero wipe them, it actual one wipes them by writing ones to the drive instead of zeros. This was by design, because if a sector could not write a 1 to the drive, then the surface scan check would be sure to fail. This was also by design for SSDs. The purpose was to fill up the entire drive to make sure the nands could properly hold electricity (or "ones"). If the sectors were failing on a SSD then, hopefully, the firmware would reallocate the sector somewhere else. Or if the chips were on their last legs, then such a write from badblocks would break those "last legs" and cause the drive to fail.  (better for it it fail now than during production).

I have heard from many sources that it is not the best to write over an SSD like this, because it would reduce the lifespan of the drive. If you feel the same way, then do not use these scripts for SSDs.

These are the following scripts associated with this, along with a brief summary on what they do:


=====================


batch_test

batch_test will loop through /dev/sda to /dev/sdz. It will check to see if the drive has the system root partition on it. If it does, then it will skip it, if it does not, then it will proceed to wipe, test, and get SMART info about the drive. Batch_test was designed to only test a certain amount of drives at a time, due to the limitations of the computer it was designed for. It defaults to 4 drives. But you can pass in a number upon start to change that. The script will keep scanning to see if the processes for the drives are still running or not. Once the process stops running, then it will try to start the next drive (if it exists). And it will loop until it reaches drive /dev/sdz.  While the drives are testing, you can add more drives to the system. As long as the device identifier (/dev/sd) for the new drive does not surpass /dev/sdz, then the script will start on that drive once it loops around to it. However, this only happens if it is in the main loop. Once it exits the main loop, it will enter the "wait loop" where it simply waits for all processes to end. And it will reach that wait loop once the number of [current testing drives] is less than [number of drives to test].

batch_test only takes one argument:

batch_test [number of drives to test]

[number of drives to test] defaults to 4. If you test SSDs, then be aware that you can flood your bandwidth and cause things to go a lot slower, so set this number to what your pcie or raid card can handle.

You can use the default of 4 drives to test at a time by running batch_test with no arguments

This script will also create log files for each device. It will also create a dated folder for all of those log files. This was done so you can go back and review each drive at a later date. 

=====================

hdd_test

hdd_test is a script that is used by batch_test. You can use this script by itself. Just pass in the drive letter and it will begin testing that drive. It has been tested on 18 TB drives. I know it handles them. The badblocks command might need to be adjusted if the drive is a higher capacity. 

Syntax:

hdd_test [drive letter]

Example: hdd_test b

- this will test /dev/sdb 

** note: be default, /dev/sd is already coded in, so you do not need to pass in the path - just the drive letter (or last letter of /dev/sd_)

=====================

sdd_info

This script will try to grab the smart data for an ssd. The TBW is calculated differently between brands. Only a few different brands were researched and added into this. This info is not always accurate. And sometimes the TBW is not gathered. 

Syntax:

ssd_info [device path]

Example:

ssd_info /dev/sda 

*this script is used by hdd_test


=====================

hdd_lister_and_dd

This is a batch script that constantly lists the drives attached to the system in a loop. If it detects a drive with a partition, then it will dd the first million blocks of data to wipe the drive. However, if it detects a drive that has the root partition of your computer, then it will skip that drive. It will immediately erase USB drives, so do not put them into the computer when this script is running.

This was designed, because during hard drive testing, not all drives were good and would show up on the system. If a drive was too damaged, then it could prevent other drives from showing up on the raid system. It would take 10 to 20 seconds for a drive to show up as soon as it was attached to the raid card (which was running in JOBD mode). If the drive did now show up with this script, then the drive was removed and reallocated for further testing.


=====================

quick_review

This script will cat / grep the information out of each of the log files for each drive and put it into an easier to read format. It basically displayed all the information that was needed to determine if a drive was good or bad. 

Usage:

cd into the folder created by batch_test. Then run

quick_review
