+++
title = 'indications.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/indications.conf"
aliases = [
    "/?page_id=67",
    "/asterisk/15/",
    "/old-content/asterisk/indications_conf/",
]
+++
```
[general]
country=uk


[us]
description = United States / North America
ringcadance = 2000,4000
dial = 350+440
busy = 480+620/500,0/500
ring = 440+480/2000,0/4000
congestion = 480+620/250,0/250
callwaiting = 440/300,0/10000
dialrecall = !350+440/100,!0/100,!350+440/100,!0/100,!350+440/100,!0/100,350+440
record = 1400/500,0/15000
info = !950/330,!1400/330,!1800/330,0


[uk]
description = United Kingdom
ringcadance = 400,200,400,2000
dial = 350+440
busy = 400/375,0/375
ring = 400+450/400,0/200,400+450/400,0/2000
congestion = 400/400,0/350,400/225,0/525
callwaiting = 440/100,0/4000
dialrecall = 350+440
; XXX Not sure about the RECORDTONE
record = 1400/500,0/10000
info = 950/330,1400/330,1800/330
```