+++
title = "Yubikey and server authentication"
#description = ""
date = "2011-12-06T16:48:05+00:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=267",
    "/2011/12/06/yubikey-and-server-authentication/"
]
in_search_index = true


tags = [
    "Linux",
    "Projects",
    "Software"
]
[extra]
+++
 After starting to use the Yubikey for LastPass and various other online servers I’ve started also using my yubikey for SSH access to my server(s).

I’ve touched on google_authenticator and pam_yubico for authentication in a previous post however I will be going into this in a bit more detail.

Taking a machine at home as an example. My requirements are simple

1. NO SSH Key access to be allowed – as there is no way to require a second factor with an SSH Key (Passphrases can be removed or a new key generated)
2. Access from Local machines to be allowed without Two Factor being enabled
3. Yubikey to be the Primary TFA
4. Fall back to google authenticator should either the Yubico servers be down, an issue with my keys or I just don’t have a USB port available (IE I’m on a phone or whatever)

In order to meet these requirements I’m going to need the following
1. [yubico-pam Yubikey PAM](https://github.com/Yubico/yubico-pam)
2. [Google Authenticator PAM](http://code.google.com/p/google-authenticator/)
3. pam_access
The server is running Archlinux, and luckily all of these are within AUR – and as such I’m not going to cover the install of the modules.

In order to restrict SSHd access as above I need the following auth lines in `/etc/pam.d/sshd`

```
# Check unix password
auth            required        pam_unix.so try_first_pass
# check to see if the User/IP combo is on the skip list - if so, skip the next two lines
auth            [success=2 default=ignore] pam_access.so accessfile=/etc/security/access_yubico.conf
# Check /etc/yubikey for the users yubikey and skip the next line if it all works
auth            [success=1 default=ignore ]     pam_yubico.so id=1 url=https://api.yubico.com/wsapi/2.0/verify?id=%d&otp=%s authfile=/etc/yubikey
# Check against google authenticator
auth            required        pam_google_authenticator.so
auth            required        pam_env.so
```

The next step is ensure that the relevant users and IP are listed in `/etc/security/access_yubico.conf`

```
# Allow welby from 1.2.3.4
+ : welby : 1.2.3.4
# Deny all others
- : ALL : ALL
```

After this is setup we will also need to setup the yubikey file `/etc/yubikey`

```
welby:ccccccdddddd:cccccccccccc
```

I’m not going to cover configuration of google authenticator with the google-authenticator command

The final changes are to the `/etc/ssh/sshd_config` ensuring that the following are set

```
PasswordAuthentication no
PubkeyAuthentication no
PermitRootLogin no
ChallengeResponseAuthentication yes
UsePAM yes
```