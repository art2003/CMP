# CMP for MCC-91927
The job is to replace existing Medidata.Core.Objects.dll with new ones. The affected Rave versions and nodes are as below table.

## Affected Rave Versions
|Product Name |Assembly Version |Product Version |Nodes|
|---------------------------|--------------------|---------------------------|----------|
|Medidata Rave® 2013.2.0	|5.6.5.45 | 20140415223636-b6d4a73-u |Application nodes|
|Medidata Rave® 2013.2.0.1	|5.6.5.50 | 20140425133920-42ad77c-u |Application nodes|
|Medidata Rave® 2013.3.0	|5.6.5.66 | 20140425133901-edac9d8-u |Application nodes|
|Medidata Rave® 2013.3.0.1	|5.6.5.71 | 20140425160927-940900d-u |Application nodes|
|Medidata Rave® 2013.4.0	|5.6.5.92 | 20140425133800-285b96c-u |Application nodes and Web nodes|

This script uses "Build Version" as identity to handle the patch process.

## Workflow of the script
1. Connect WHOIS database to get deployment information for all sites (or say "URL" in Medidata language) and their sibling nodes.
2. Filter out those sites need to be patched.
2. Looply execute step 3~6 on each site
3.    Stop the core service of each sibling if it's an App server.
4.    Backup the original Medidata.Core.Objects.dll and replace it with the new dll on each sibling.
5.    Start the core service of each sibling if it's an App server.
6.    If any error happens between step 3~5, restore the dll from backup. Otherwise, insert one record into site's RavePatches table. The PatchNumber is constantly "MCC-91927".

## Features

### How to use

```
PS ~> .\CMP-MCC91927.ps1 $WhoisDBServerName$ [$OpeCoreServiceTimeOutSeconds$] [$RetryCoreServiceTimes$]
```

- **$WhoisDBServerName$** is the server name of WHOIS database and is required.
- **$OpeCoreServiceTimeOutSeconds$** is the time out in seconds to wait for starting or stopping core service. This is optional and default value is 30.
- **$RetryCoreServiceTimes$** is the retry times if starting or stopping core service failed. This is optional and default value is 3.

*Notice: You may consider to increase timeout and retry times to reduce core service operation failure.*

### Patch site as a whole or do nothing
The script ensures all sibling nodes of a single site are all patched or none. If error happens in the middle, the script will try to restore those have been patched from the backup (See "Log file and backup" below), so as to avoid discrepancy among these siblings.

### Safe to re-run
The script was designed to be rerunnable safely. It means it will detect if the patch has been finished on the target site. So the script will automatically skip those patched sites.

### Log file and backups
Log file will be generated each time the script is run. A folder with name like "_$Timestamp$" (e.g. "_30Apr2014 18.26.56 407") will be created at the same directory of CMP-MCC91927.ps1. Within this folder, there will be a log file called "log.txt", whose contents are identical to the message on command prompt. 

There also will be a "backup" folder where the patching target's original files will be backed up. The contents under "backup" folder can be used for manual restoring mispatched nodes.

See the following directory structure after running "CMP-MCC91927.ps1".

```
│   CMP-MCC91927.ps1
│
├───_20140430 195812.488
│       log.txt
│
└───_20140430 195824.754
    │   log.txt
    │
    └───backup
        ├───fakesite01.mdsol.com(5.6.5.45)
        │   └───fakeAppServer01
        │           Medidata.Core.Objects.dll
        │
        └───fakesite02.mdsol.com(5.6.5.92)
            └───fakeAppServer02
            │       Medidata.Core.Objects.dll        
            └───fakeWevServer02
                    Medidata.Core.Objects.dll
```
### The record in RavePatches table
The newly inserted record in RavePatches table is like below.
| id|	RaveVersion	|PatchNumber	|version	|Description	|DateApplied	|AppliedBy	|AppliedFrom	|Active	|AppServers	|WebServers	|Viewers	|BatchUploader	|NonSqlRun|
|---|	----------	|-----------	|-------	|------------	|------------	|-------	|-----------	|----	|--------	|-------	|-------	|-------	|-------|
| 91|	5.6.5.45	|MCC-91927	|1	|Replace Medidata.Core.Objects.dll	|2014-05-01 15:14:59.537|	|NULL|	NULL	|1	|NULL	|NULL|	NULL|	|NULL|	NULL|


