+++
title = 'Cisco 7960 / 7940 SIP upgrade procedure'
date = '2004-05-25T20:26:58+01:00'
url = "/cisco"
aliases = [
    "/?p=20",
    "/cisco/",
    "/2005/05/25/cisco-7960-7940-sip-upgrade-procedure/",
    "/old-content/cisco-7960-7940-sip-upgrade-procedure/",
    "/cisco-7960-7940-sip-upgrade-procedure/"
]
+++

# Intro

This is a short HOWTO type document on how to upgrade and use a cisco ip phone 7940 or 7960. These phones are shiped in CallManager (skinny) mode

# Setting up your local network.

First things first, you will need to be running a DHCP server. This is just a matter of setting up your favourite dhcpd on your favourite operating system. You need to make sure that your phone can get the Gateway/router, DNS server and TFTP server sent with the dhcp lease.

You then will also need to set up a tftpd. This is not going to be covered here at all.

# Upgrading the phone from a early call manager firmware

To start you need a “AL” firmware or above. If you don’t have this you will have to use call manager.

After this is done you will need to create your TFTP files. First file you will need is P0S30202.bin or similar. Put this file in the root of your TFTP folder. Also you will need your OS79XX.TXT.  
  
Open this file up in your favourite editor and edit it so that it contains the flowing line

```
    P0S30202
```

Save the file and go back to your TFTP root.  
You should also put the files “Ringer1.PCM” and RINGLIST.DAT.

The Contents of ringlist.dat is similar to

```
    Old Style ringer1.pcm
    Pulse Pulse1.raw
```

Where the first tsv is the name, and the second is the file name. Save this file as well.

I have recently been informed that this is not nescessary unless your adding more ringers to the phone

Next create two files  
SIPDefault.cnf and SIP.cnf (such as SIP0006D7AA2A2E.cnf)

Open SIPDefault.cnf

This file should contain the following (adjusted for your settings)

```
    # Image Version
    image_version: P0S30202
    # Proxy Server
    proxy1_address: "fwd.pulver.com" ; Can be dotted IP or FQDN
    proxy1_port: 5060
    # Proxy Registration (0-disable (default), 1-enable)
    proxy_register: 1
    # Phone Registration Expiration [1-3932100 sec] (Default - 3600)
    timer_register_expires: 3600
    # Codec for media stream (g711ulaw (default), g711alaw, g729a)
    preferred_codec: g711ulaw
    # TOS bits in media stream [0-5] (Default - 5)
    tos_media: 5
    # Inband DTMF Settings (0-disable, 1-enable (default))
    dtmf_inband: 1
    # Out of band DTMF Settings (none-disable, avt-avt enable (default), avt_always - always avt )
    dtmf_outofband: avt
    # DTMF dB Level Settings (1-6dB down, 2-3db down, 3-nominal (default), 4-3db up, 5-6dB up)
    dtmf_db_level: 3
    # SIP Timers
    timer_t1: 500 ; Default 500 msec
    timer_t2: 4000 ; Default 4 sec
    sip_retx: 10 ; Default 10
    sip_invite_retx: 6 ; Default 6
    timer_invite_expires: 180 ; Default 180 sec
    # Dialplan template (.xml format file relative to the TFTP root directory)
    dial_template: dialplan
    # TFTP Phone Specific Configuration File Directory
    tftp_cfg_dir: "" ; Example: ./sip_phone/
    # Time Server (There are multiple values and configurations refer to Admin Guide for Specifics)
    sntp_server: "194.81.227.227" ; SNTP Server IP Address (this is ntp1.ja.net)
    sntp_mode: anycast ; unicast, multicast, anycast, or directedbroadcast (default)
    time_zone: GMT ; Time Zone Phone is in
    dst_offset: 1 ; Offset from Phone's time when DST is in effect
    dst_start_month: April ; Month in which DST starts
    dst_start_day: "21" ; Day of month in which DST starts
    dst_start_day_of_week: Sun ; Day of week in which DST starts
    dst_start_week_of_month: 1 ; Week of month in which DST starts
    dst_start_time: 02 ; Time of day in which DST starts
    dst_stop_month: Oct ; Month in which DST stops
    dst_stop_day: "20" ; Day of month in which DST stops
    dst_stop_day_of_week: Sunday ; Day of week in which DST stops
    dst_stop_week_of_month: 8 ; Week of month in which DST stops 8=last week of month
    dst_stop_time: 2 ; Time of day in which DST stops
    dst_auto_adjust: 1 ; Enable(1-Default)/Disable(0) DST automatic adjustment
    time_format_24hr: 1 ; Enable(1 - 24Hr Default)/Disable(0 - 12Hr)
    dnd_control: 0 ; Default 0 (Do Not Disturb feature is off)
    callerid_blocking: 0 ; Default 0 (Disable sending all calls as anonymous)
    anonymous_call_block: 0 ; Default 0 (Disable blocking of anonymous calls)
    dtmf_avt_payload: 101 ; Default 101
    # Sync value of the phone used for remote reset
    sync: 1 ; Default 1
    proxy_backup: "" ; Dotted IP of Backup Proxy
    proxy_backup_port: 5060 ; Backup Proxy port (default is 5060)
    proxy_emergency: "" ; Dotted IP of Emergency Proxy
    proxy_emergency_port: 5060 ; Emergency Proxy port (default is 5060)
    # Configurable VAD option
    enable_vad: 0 ; VAD setting 0-disable (Default), 1-enable
    nat_enable: 1 ; 0-Disabled (default), 1-Enabled
    nat_address: "" ; WAN IP address of NAT box (dotted IP or DNS A record only)
    voip_control_port: 5060 ; UDP port used for SIP messages (default - 5060)
    start_media_port: 16384 ; Start RTP range for media (default - 16384)
    end_media_port: 32766 ; End RTP range for media (default - 32766)
    nat_received_processing: 0 ; 0-Disabled (default), 1-Enabled
    outbound_proxy: "192.246.69.247" ; restricted to dotted IP or DNS A record only (this is fwdnat.pulver.com)
    outbound_proxy_port: 5082 ; default is 5060
    # Allow for the bridge on a 3way call to join remaining parties upon hangup
    cnf_join_enable : 1 ; 0-Disabled, 1-Enabled (default)
    # Allow Transfer to be completed while target phone is still ringing
    semi_attended_transfer: 1 ; 0-Disabled, 1-Enabled (default)
    # Telnet Level (enable or disable the ability to telnet into the phone)
    telnet_level: 2 ; 0-Disabled (default), 1-Enabled, 2-Privileged
    # XML URLs
    ;services_url: "http://your.site/services.xml" ; URL for external Phone Services
    services_url: "http://193.113.58.136/bt/" ;bt services
    directory_url: "http://your.site/directory.xml" ; URL for external Directory location
    logo_url: "http://your.site/logo.bmp" ; URL for branding logo to be used on phone display
    # HTTP Proxy Support
    http_proxy_addr: "" ; Address of HTTP Proxy server
    http_proxy_port: 80 ; Port of HTTP Proxy Server (80-default)
    # Dynamic DNS/TFTP Support
    dyn_dns_addr_1: "" ; restricted to dotted IP
    dyn_dns_addr_2: "" ; restricted to dotted IP
    dyn_tftp_addr: "" ; restricted to dotted IP
    # Remote Party ID
    remote_party_id: 0 ; 0-Disabled (default), 1-Enabled
    # Call Hold Ringback (0-disabled, 1-enabled, 2-disabled no user control, 3-enabled no user control)
    call_hold_ringback: 0 ; Default 0 (Disable ringback of held call)
```

Save this file and open SIP.cnf  
This file should contain

```


    # Line 1 appearance
    line1_name: line1
    # Line 1 Registration Authentication
    line1_authname: "authname"

    # Line 1 Registration Password
    line1_password: "password"
    # Line 2 appearance
    line2_name: line2
    # Line 2 Registration Authentication
    line2_authname: "authname"
    # Line 2 Registration Password
    line2_password: "UNPROVISIONED"


    ####### New Parameters added in Release 2.0 #######
    # All user_parameters have been removed
    # Phone Label (Text desired to be displayed in upper right corner)
    phone_label: "Phone using " ; Has no effect on SIP messaging
    # Line 1 Display Name (Display name to use for SIP messaging)
    line1_displayname: " line 1"
    # Line 2 Display Name (Display name to use for SIP messaging)
    line2_displayname: " line2"


    ####### New Parameters added in Release 3.0 ######
    # Phone Prompt (The prompt that will be displayed on console and telnet)
    phone_prompt: "SIP Phone" ; Limited to 15 characters (Default - SIP Phone)
    # Phone Password (Password to be used for console or telnet login)
    phone_password: "cisco" ; Limited to 31 characters (Default - cisco)
    # User classifcation used when Registering [ none(default), phone, ip ]
    user_info: none
```

Save this file. And create a file called syncinfo.xml.

This file needs to contain

```


     <SYNCINFO>

     <IMAGE VERSION="*" SYNC="1"/>
     </SYNCINFO>
```

Save this file and create dialplan.xml  
This file should contain.

```


     <TEMPLATE MATCH="*" Timeout="5"/>>
     </DIALTEMPLATE>
```

Save this file. And boot the phone.

It should say on the screen.

```


    Configuring VLAN
    Configuiring IP
    Requesting Configuration
    Upgrading Software
```

And then it should reboot. (after a seiers of light flashing on the headset/speaker and mute buttons.)

This is you phone now upgraded to SIP v2.2 firmware. You can now proceded to the next step.

# Upgrading to SIP v3.x firmware

To do this you need to edit the OS79XX.TXT file to contain P0S3-03-0-00.  
Also you then need to change the line in SIPDefault.cnf so that

```
    Image_version: P0S3-03-0-00 
```

Is the line at the top.

After you have done this make sure that the P0S3-03-0-00.bin file is in the tftp root. And then send the phone a CHECK-SYNC message.

I have been informed that this should NOT be nesscessary for local updates, however for some reason my phone required it, so i shall leave it here just incase

This can be done using the following perl script

```perl


    #!/usr/bin/perl
    use Socket; reboot("192.168.0.123"); # this is the phones ip
    sub reboot{
    $phone_ip = shift;

    $MESG="NOTIFY sip:sip@$phone_ip:5060 SIP/2.0
    From: 
    To: 
    Event: check-sync
    Date: Mon, 10 Jul 2000 16:28:53 -0700
    Call-ID: test@192.168.0.1
    CSeq: 1300 NOTIFY
    Contact: 
    Content-Length: 17806";


    $proto = getprotobyname('udp');
    socket(SOCKET, PF_INET, SOCK_DGRAM, $proto) ;
    $iaddr = inet_aton("66.123.187.80");
    $paddr = sockaddr_in(5060, $iaddr);
    bind(SOCKET, $paddr) ;
    $port=5060;
    $hisiaddr = inet_aton($phone_ip) ;
    $hispaddr = sockaddr_in($port, $hisiaddr);
    send(SOCKET, $MESG, 0,$hispaddr ) || warn "send $host $!n";
    }

```

Run this script and then that should make the phone reboot and upgrade the firmware. After this is done you should be running a v3 SIP firmware.

# Upgrading to V4.

CAUTION v4.4 is the only SIP firmware that will work with FWD and most other SIP services. This is apparently fixed in v4.4 and I seem to be having no problems using.

To do this you need to edit the OS79XX.TXT file to contain P0S3-04-4-00.

Also you then need to change the line in SIPDefault.cnf so that

```
    Image_version: P0S3-04-4-00
```

Is the line at the top.

After making sure that your firmware file is in the root of the TFTP server. Reboot the phone (power cycle) and it should automatically upgrade your firmware.

Once this is done you should be running v4.4 of SIP firmware.

I Have recently got the 5.3 Firmware, and upgradeing is much the same as 4.4