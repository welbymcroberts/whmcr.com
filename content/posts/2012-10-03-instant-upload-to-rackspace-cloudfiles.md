+++
title = "\"Instant\" Upload to Rackspace CloudFiles"
#description = ""
date = "2012-10-03T17:27:37+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=282",
    "/2012/10/03/instant-upload-to-rackspace-cloudfiles/"
]
in_search_index = true

tags = [
    "hacks",
    "bad ideas",
    "Linux"
]
[extra]
+++
Using inotify and the ‘swift’ client tools it is possible to automatically upload files to cloudfiles as they are written to disk.

This code is untested and might cause planes to drop from the sky, use it at your own risk!

```bash
#!/bin/bash
DIRECTORY='/home/welby/'
CONTAINER='testing'
USERNAME="welbycloud"
KEY="APIKEYFORCLOUD"
VERSION="2.0"
inotifywait -mr --format '%w%f' -e close_write $DIRECTORY | while read filename; do
    if [ ! -e "$1" ]; then
       sleep 2
       if [ ! -e "$1" ]; then
           continue
       fi
    fi
    swift upload  -s -A https://auth.api.rackspacecloud.com/v1.0 -U $USERNAME -K $KEY -v $VERSION $CONTAINER $filename 
done
```