+++
layout = 'post'
title = 'CrowdStrike Microsoft Outage'
date = 2024-07-25 01:52:31.730332380+05:45
draft = true
+++

# crowdstrike-microsoft-outage-BSOD
- CrowdStrike revealed that the global IT outage that impacted transport, finance, and medical industries around the world was caused by a sensor configuration update for Windows gone wrong.
- Crowdstrike Falcon Sensor for windows was updated for microsoft windows version 7.11 or above

## when was the update?
- july 19 windows update over the air
- update time from 4:09 UTC to 5:27 UTC

## what was the update for?
- Falcon sensor update
- update was designed to target newly observed, malicious named pipes being used by common C2 frameworks in cyberattacks
- the configuration update triggered a logic error that resulted in an OS crash

## what was the impact?
- 8.5 million windows devices
- US and Europe mostly 
- some corporates in india also faced the issue
- what about china? -> well china hardly uses apps from american companies for their system
- hospitals, airlines, shopping malls could not function properly
- some reports of telephone and internet outages(not so clear about this info yet)
- some reports of 911 emergency call outage in some US states

## how did it happen?
- some updates in the configuration files caused the issue
- configuration files are called channel files in here: -> C:\Windows\System32\drivers\CrowdStrike\
- "these are not kernel drivers but extensions"
- microsoft kernel during boot loads the sys files
- boot time issue with the sys files, BSOD while loading that file because it had issues

## solution?
- remove the files from the recovery process?
- if you have bitlocker enabled, first allow bitlocker to decrypt the harddrive using your password
    - then remove the sys files

## my questions!
- no mass testing deployments? canary deployments?
- did they rush the feature push to production?
- centralized internet?
