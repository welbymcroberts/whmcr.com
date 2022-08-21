+++
title = "Sky on a HTPC"
#description = ""
date = "2008-09-16T23:05:27+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=148",
    "/?p=3",
    "/2008/09/16/sky-on-a-htpc/"
]
in_search_index = true


tags = [
    "sky",
    "CAM",
    "DTV",
    "DVB",
    "htpc",
    "media",
    "'media centre'",
    "Hardware",
    "media",
    "Projects",
    "sky",
    "TV"
]
[extra]
+++

Recently I’ve become more and more annoyed with my SKY-HD’s disk spinning up and down, and then the power appearing to be cut to the drive, meaning that theres a rather loud click comes from it. Not a problem if you’re watching TV, as this only occurs when the box is in standby. Very annoying if you’re having problems sleeping and the thing is going clunk ever 30 minutes or so. I’ve been told that I can change a disk spin down somewhere on the box, however this doesn’t appear to have made any difference. Another issue that is compounding the annoyance is that the SKY-HD box is almost impossible to use with a single tuner.

I decided to resurrect my HTPC and attempt to get SKY going into that. There where 4 major requirements for this

1. Has to be able to play content – I pay a silly amount a month just for 3 HD channels (BBC, Discovery and History) :: This meant that a DVB-S2 receiver was required

2. Has to be able to decode pay for channels – I pay a subscription to them, I’ll be damned if I don’t get my channels! :: This meant that either a SoftCAM or a CI slot and CAM were required

3. Has to be local to the machine, I want a raw MEPG2/h264 stream going to the media pc, and not any additional transcoding, also one less set of CPUs is a good thing ™ – This isnt a poke at a specific Linux Based satellite receiver at all :: This meant internal cards or locally attached devices (USB2/Firewire)

4. The HTPC must be running software that can play my videos – I don’t want to have my popcorn hour AND a HTPC to do my video. :: This meant using a Media Centre type application, this does however exclude Microsoft’s Windows Media Centre, as it doesn’t play MKVs/OGMs etc

Relatively small requests one would think, but apparently not! I was left with a few choices for Card, however the one that seemed to come out tops was the Digital-Everywhere [FloppyDTV/S2](http://www.kustompcs.co.uk/acatalog/info_4348.html). This meets requirements 1 & 3, by being able to decode DVB-S2 signals, and also is sending data via the Firewire bus.

In order to meet requirement 2 I opted for the “Dragon Cam” (or specifically the T-Rex 4.1). This is a Conditional Access Module, which along with a valid SKY viewing card, preforms the VideoCrypt (NDS) decryption. This does have one, annoying, caveat. The smartcard must go into a SKY box every 4->6 weeks to have a “new installation” done, as the CAM will not rewrite the new decryption codes to the card.

The Shopping list at the end of all of this was as follows:

* Digital Everywhere FireDTV S2 External @ £160 (External was chosen for various reasons, including the IR remote support)
* Firewire PCI Card @ £10
* T-Rex Cam @ £60
* Infinity Unlimited USB Card Programmer @ £60 – This was required to do the initial loading of the T-Rex CAM, however can be returned / resold / similar as its a once off requirement

So all in all £220 to view/record on a media pc. This is for a single tuner only, as I don’t have access to multiple drops from the buildings satellite distribution system (which is rather amusing, as these are "high end" flats, built in the last 3 years, and yet all flats only have a single drop for satellite). Multi Drops can be done using a SoftCAM, where the CAM is replaced with a USB Smartcard programmer, but only one is required, meaning that the first channel would be £220, but each after that would be £160 (or £130 if an internal was to be used). Of course the Legality of using a SoftCAM is extremely questionable, where as a non official sky receiver is only marginally.