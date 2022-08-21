+++
title = "PAM and Two Factor authentication"
#description = ""
date = "2011-11-14T21:33:24+00:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=262",
    "/2011/11/14/pam-and-two-factor-authentication/"
    
]
in_search_index = true


tags = [
    "Linux",
    "Projects",
    "Software"
]
[extra]
+++

As the need for Two factor authentication is a requirement for PCI-DSS (Payment Card Industry standard) and SSH Key with password is not always deemed to be an acceptable form of Two factor authorisation there is now a surge in different forms of two factor auth, all with their own pros and cons.

For a small business or ‘Prosumer’ (professional consumers) the market incumbent (RSA) is not a viable option due to the price of the tokens and the software / appliance that is required. There are cheaper (or free!) alternatives for which two that I’ve used at Google Authenticator, and Yubikey.

Google Authenticator is an OATH-TOTP system that much like RSA generates a one time password once every 30 seconds. It’s avaiable as an App for the Big three mobile platforms (iOS, Android and Blackberry).

Yubikey is a hardware token that emulates a USB keyboard, that when the button is pressed, generates a one time password. This is supported by services such as lastpass.

Both solutions have the ability to be used with their own PAM modules. Installation of either is simple, but what happens if you want to use both, but only require one of these.

Luckily PAM makes it quite easy !

```
auth            required        pam_unix.so try_first_pass
auth            [success=1 default=ignore ]     pam_yubico.so id=1 url=https://api.yubico.com/wsapi/2.0/verify?id=%d&otp=%s
auth            required        pam_google_authenticator.so
```

In the above example the user must enter a password and then provide either their yubikey or their google_authenticator.

Should the password be incorrect the user will still be prompted for their yubikey or google authenticator, but will then fail. Should they provide a password and then their yubikey, they will not be asked for their google authenticator. Should they provide password and not a yubikey, they will be prompted for their google authenticator!