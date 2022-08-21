+++
title = 'voicemail.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/voicemail.conf"
aliases = [
    "/?page_id=85",
    "/asterisk/24/",
    "/old-content/asterisk/voicemail_conf/",
]
+++

```

[general]
; Default formats for writing Voicemail

format=wav49|gsm|wav
; Who the e-mail notification should appear to come from
serveremail=voicemail@bordem.net
; Should the email contain the voicemail as an attachment
attach=no
; Maximum length of a voicemail message
maxmessage=180
; Maximum length of greetings
maxgreet=60
; How many miliseconds to skip forward/back when rew/ff in message playback
skipms=3000
; How many seconds of silence before we end the recording
maxsilence=10
; Silence threshold (what we consider silence, the lower, the more sensitive)
silencethreshold=128
; Max number of failed login attempts
maxlogins=3

; Skip the "[PBX]:" string from the message title
;pbxskip=yes
; Change the From: string
fromstring=The 62.bordem.net Voicemail System
; Change the email body, variables: VM_NAME, VM_DUR, VM_MSGNUM, VM_MAILBOX, VM_CALLERID, VM_DATE
emailbody=Dear ${VM_NAME}:nnt you were just left a ${VM_DUR} long message (number ${VM_MSGNUM})nin mailbox ${VM_MAILBOX} from ${VM_CALLERID}, on ${VM_DATE} n


;
; Each mailbox is listed in the form =,,,,
; if the e-mail is specified, a message will be sent when a message is
; received, to the given mailbox. If pager is specified, a message will be sent there as well.
;
[default]
6001 => 1234,Welby McRoberts,voicemail@localhost,,attach=yes
6002 => 1234,Robert Gordon,voicemail@cgi-bin.org.uk,,attach=no
6003 => 1234,Ewan Fisher,ewan@ewanfisher.co.uk,,attach=no

[other]
;1234 => 5678,Company2 User,root@localhost


```