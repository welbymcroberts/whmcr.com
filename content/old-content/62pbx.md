+++
title = '6-2 Calder View Phone system'
date = '2003-05-25T21:00:41+01:00'
url = "/62pbx"
aliases = [
    "/?p=41",
    "/62pbx/",
    "/6-2-pbx/",
    "/2003/05/25/62pbx/",
    "/old-content/62pbx/",
]
+++

# Making Calls

To make a call simply pick up the handset. ou should hear a dialtone. Simply just dial your number, there may be a slight pause whilst it connects you. You may skip part of this pause by pressing the # key.

# Reciving Calls

You will be able to recive calls exactly as you would recive a normal phone call, just pick up the handset and you will be connected

# Internal Extensions

All internal phones have a “6000” extension.

For example this may be 6001 for the first phone setup, 6002 for second etcetcetc

The Current Extensions are

```
    * 6001 : Welby
    * 6002 : person2
    * 6003 : person3
```

# PSTN Calls

To call out using the Telewest phone line simply place a call dialing the normal phone number. For instance 01234 567 890

For local calls simply dial the number. For instance 421 3456

Only calls of the Following types are allowed

```
    * 01/02 Numbers
    * 0845
    * 087*
    * 080*
    * 0500
    * Mobiles (07* but not 070* which are Personal Redirect numbers at many p/min)
    * 999
    * 150
```

# Voicemail

If you have a voicemail you should have a stuttering dialtone. To retrive your messages dial 1571. Your mailbox is your extension number, and the default pin is 1234. Please change this by pressing 0 in the main menu

## Voicemail menu

```

    * 1 Read voicemail messages
          o 5 Repeat current message
          o 6 Play next message
          o 7 Delete current message
          o 8 Forward message to another mailbox
          o 9 Save message in a folder
          o * Help
          o # Exit
    * 2 Change folders
    * 0 Mailbox options
          o 1 Record your unavailable message
          o 2 Record your busy message
          o 3 Record your name
          o 4 Change your password
          o * Return to the main menu
    * * Help
    * # Exit

```

# 141’ing your number

To withhold your number on an outgoing call simply dial 141 before the number. For instance 14101213335671.

Prank Calls to certain well known college numbers and Helldesks are automatically 141’ed

# IAXTEL

We are connected to the IAXTEL network so we can make calls to american 1700/1888/1877/1866/1800 numbers. This may, or may not include some canadian numbers. To make these calls, just dial the number

# Free World Dialup

We are connected to the Free World Dialup network via IAX on parcelfarce.bordem.net. To dial any FWD extension simply prefix the number with a 7. For instance 757052. Our inbound FWD number is 57052. All incoming FWD numbers should automatically be prefixed with a 7.

# Conference Calls

We do not subscribe to the Telewest threeway calling service anymore, so only one incoming call can be had on the landline, and in reality two or three incoming on the VoIP side of things.

However we do have an internal conference system inplace. To access this dial 6101 for the first conference room, 6102 for the second etc until 6105 which is the last conference room. There are no PIN’s on these conference rooms.

You may also transfer calls to the conferences. So if theres an incoming call, transfer them to 6101.

# Calling a.n.otherperson

This can be done by dialing 1919. This goes over an IAX connection straight to his asterisk server.

# 1471

Dial 1471 for the number of the last person to call you, press 3 to return the call

# Transfering calls

To “blind transfer” a call, press recall then #90 number # then hang up

So for instance to transfer the call to welby, press recall #906001# and hang up

To transfer the call normaly, press recall, dial the number, wait untill the person answer and then hang up

# Caller ID

Caller ID is enabled on all phone. On the ATA’s caller id is using the BELLCORE standard. Make sure that the phone is “cable” caller id ready

# Call Divert

To divert all calls to another number, dial \*21\*number#

To divert calls when your busy to another number, dial \*67\*number#

To cancel all diverts call #21#

# Call waiting

Call waiting should be on for all calls. To switch between callers press recall

To switch it off for the next call dial #43#

# Call Parking

If your wanting to move phones press # whilst in a call and dial 6200#. You will be given an number like 6201. Go to the other phone and dial 6201 and you will be connected to your old call

# Other Silly things

```
    * 123 - The Talking Clock
    * 124 - Another talking clock
    * 401 - di.fm stream
    * 500 - The echo test
    * 501 - The Main Menu (as inbound callers get)
    * 502 - PING AGI script (requires festival to be fixed)
    * 503 - Some random sounds
    * 890 - Older Telewest 890 music (Primal Scream - Movin' on up)
    * 891 - Old Telewest 890 music (Unkown)
    * 892 - Not in use yet (will be "new" 890 music)
    * 17070 - What Number is this?
```