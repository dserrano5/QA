# 0.8 blockchain data upgrade

Submit completed tests and debug.log files to:  [Gavin Andresen](mailto:gavin@bitcoinfoundation.org)

Test environment:

(TEST RUNNING PERSON: edit this to describe the machine you're testing on; e.g. Windows 7, running Bitcoin-Qt version 0.7.2)

## Bounties:

- CLAIMED 0.5 BTC : [Test starting from version 0.7.2](https://github.com/weapon-x/QA/blob/master/TestPlanExecution.md)
- CLAIMED 0.5 BTC : Test starting from version 0.3.24
- CLAIMED 0.5 BTC : Test on Windows (any version), FAT32 filesystem 
- CLAIMED 0.5 BTC : Test on Windows XP, NTFS filesystem (subStrata)

Bounties will go to the first people who follow the instructions in [TestPlanExecution.md](TestPlanExecution.md) and this test plan and
submit test results and timestamped debug.log files. These bounties are being paid from the 72 BTC donated
to the Bitcoin Testing Project.

## Test Setup

1. Begin by running an old version, fully-synced on the main network.
2. Shut down bitcoind/Bitcoin-Qt, wait for it to exit completely.
3. Backup your wallet.dat (if it contains any bitcoins; better safe than sorry!)
4. Remove [data directory](https://en.bitcoin.it/wiki/Data_directory) debug.log

## Tests

### Upgrade

Download and install test binaries from the latest successful pull-tester build [(link here)](http://jenkins.bluematt.me/pull-tester/f4445f9982a760869c430f3d4b1302f7eb509bd8/bitcoin/)
Use bitcoin-qt.exe if you are on Windows, bitcoin-qt if Linux (sorry, Mac builds not available).
(or, if you are able, [build binaries yourself](https://github.com/bitcoin/bitcoin/pull/2099) )

1. Note how much disk space is being used by the bitcoin data directory.
2. Run the new bitcoind/Bitcoin-Qt binary.

EXPECT (all tests except for FAT32 filesystem):

1. blktree/ subdirectory created in bitcoin data directory
2. small (less than 1GB) increase in disk space usage
3. startup takes longer than ususual (blocks are re-indexed)
4. but startup does not take hours (blocks are not re-downloaded)

PASS/FAIL


EXPECT (FAT32 or other filesystem that does not support hardlinks):

1. blktree/ subdirectory created in bitcoin data directory
2. same experience as a brand-new user: entire blockchain is re-downloaded

PASS/FAIL


### Downgrade

1. Shut down the new bitcoind/Bitcoin-Qt, wait for it to completely exit
2. Run the old version of bitcoind/Bitcoin-Qt

EXPECT:

1. Quick startup
2. Old version is synchronized to blockchain from where it last exited

PASS/FAIL

### Final steps

Email your debug.log file(s) and a link to your forked github repository with test results to [Gavin](mailto:gavin@bitcoinfoundation.org).
(if the debug.log files are too large to attach to an email, upload them somewhere Gavin can see them--
mediafire.com, dropbox, shared google doc, etc)
