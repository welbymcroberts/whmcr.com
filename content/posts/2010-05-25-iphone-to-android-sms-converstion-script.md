+++
title = "iPhone to Android SMS Converstion Script"
#description = ""
date = "2010-05-25T11:56:29+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=134",
    "/2010/05/25/iphone-to-android-sms-converstion-script/"
]
in_search_index = true

tags = [
    "Android",
    "Telephony",
    "Apple",
    "iPhone",
    "Hardware"
]
[extra]
+++

Here’s a copy of my iPhone to Android script. Just a quick and dirty python script that reads in a backup from itunes and converts it to a bit of XML able to be read in by SMS Backup and Restore on the android platform

```python
from sqlite3 import *
from sqlite3 import *
from xml.sax.saxutils import escape
import codecs
import re
f = codecs.open('sms.xml','w','utf-8')
f.write ('''

''')
# This is 31bb7ba8914766d4ba40d6dfb6113c8b614be442.mddata or 31bb7ba8914766d4ba40d6dfb6113c8b614be442.mdbackup usally
c = connect('sms.db')
curs = c.cursor()
curs.execute('''SELECT address,date,text,flags FROM message WHERE flags <5 ORDER BY date asc''')
for row in curs:
        a= escape(unicode(row[0]))
        d = escape(unicode(row[1]))
        t = row[3]-1
        t = str(t)
        b = re.sub('"',"'",escape(unicode(row[2])))

        f.write( ''+"n")
f.write(
'''''' )
```