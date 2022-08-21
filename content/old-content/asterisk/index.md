+++
title = "Asterisk Configuration"
date = '2003-05-25T21:00:41+01:00'
aliases = [
    "/asterisk/",
    ]
url = "/asterisk/"
+++
# My Asterisk VoIP PBX Config files

Here i’ve uploaded my example asterisk VoIP configuration files, they are old copies of them, but i will upload my new (and “improved”) versions which are based on the 1.0 Releases of Asterisk.  
The configs below where used in a 1 FXO (x100p, well mercury intel 537 based chipset winmodem), 3 SIP (1 7960, 1 ATA-186) Handset enviorment, with IAX2 trunking for a few calls.

# Opened Firewall Ports

```
SIP  : UDP 5060
IAX2 : UDP 4569
RTP  : UDP 15000 -> 15015
```

# My Config files
* [asterisk.conf](/old-content/asterisk/asteriskconf)
* [cdr_mysql.conf](/old-content/asterisk/cdr_mysqlconf)
* [extensions.conf](/old-content/asterisk/extensions-conf)
* [festival.conf](/old-content/asterisk/festivalconf)
* [iax.conf](/old-content/asterisk/iaxconf)
* [indications.conf](/old-content/asterisk/indicationsconf)
* [logger.conf](/old-content/asterisk/loggerconf)
* [manager.conf](/old-content/asterisk/managerconf)
* [meetme.conf](/old-content/asterisk/meetmeconf)
* [modules.conf](/old-content/asterisk/modulesconf)
* [musiconhold.conf](/old-content/asterisk/musiconholdconf)
* [parking.conf](/old-content/asterisk/parkingconf)
* [rtp.conf](/old-content/asterisk/rtpconf)
* [sip.conf](/old-content/asterisk/sipconf)
* [voicemail.conf](/old-content/asterisk/voicemailconf)
* [zapata.conf](/old-content/asterisk/zapataconf)