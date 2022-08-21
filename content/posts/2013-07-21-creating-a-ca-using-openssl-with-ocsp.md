+++
title = "Creating a CA using OpenSSL - with OCSP"
#description = ""
date = "2013-07-21T13:52:50+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=6",
    "/2013/07/21/creating-a-ca-using-openssl-with-ocsp/"
]
in_search_index = true


tags = [
    "infrastructure",
    "linux",
    "tls",
    "ssl",
    "ca",
    "oscp"
]
[extra]
+++

SSL Certificates are a source of huge amounts of confusion. There are two things that a SSL session will provide. The first is encryption, which can be provided with “self signed” certificates. The second, and arguably the more important is authentication of the remote server. This is managed by “Certification Authorities”. Web Browsers will have a set of known CAs that are trusted, and any certificate signed by them is therefore also trusted. Obviously if a CA has had a security breach then all bets are off.

Within an organisation it is usually preferable for **NON PUBLIC** facing sites and services to use ‘self signed’ or internal CA signed certificates. The later is usually more sensible, however it comes with the issue of more administrative time is required, and also that all clients must trust this CA.

There are various different ways of creating a CA, Windows Server 2003 and above come with their own CA software, and most UNIX/Linux distributions have OpenSSL available.

In this guide I’m going to walk through the creation of a CA using OpenSSL. I’m also going to look at enabling additional features such as OSCP (a way of clients confirming if a certificate is still valid) and go over how to create “Server Alternative Name” certificates (also known as UC or SAN certs, allowing multiple hostnames/domainnames to exist on the same cert).

One key thing to remember here is security of the CA. You must ensure that no unauthorized access is permitted to the CA, as if someone has been able to gain this, they will have access to issue certificates.

I’m also going to ensure that we setup OCSP, which is a way of clients checking to see that certificates are still valid and not revoked.

# Prerequisites

Before we start we need to have the following:

- A Linux Server with openssl installed (for this example I’m going to be using CentOS 6.4, however any linux distribution should work)
- A Domain name (in this example I’m going to use test.local)

# Configure OpenSSL

On a CentOS/RedHat system there is already a basic openssl.cnf file created, that the scripts for managing a CA already take into account. `/etc/pki/tls/openssl.cnf`

Open this up in which ever editor you like and do the following:

- Locate `countryName_default = XX` and change the `XX` to which ever country code you are in, for example the United Kingdom would be `GB`
- Locate `#stateOrProvinceName_default = Default Province` and edit this line so there is no # at the start, and that Default Province now is set to our State/Proviince/County/City
- Locate `localityName_default = Default City` and edit this to be your City
- Locate `0.organizationName_default` and edit this to be your Org name

At this point we’ve edited the config so that for any new requests you won’t have to type these in!

Whilst still in the Text Editor we need to setup the OCSP side of things.

- Locate the `[ usr_cert ]` section and add 
  ```
  authorityInfoAccess = OCSP;URI:http://URL_TO_SERVER_THAT_WILL_HOST_OCSP:8888
  ```
    
In this example I’m going to put this on the CA, but this is *NOT* a good idea from a security perspective. You want the CA to have as little (if indeed any) access from the outside.

- We also need to create the OCSP ‘extensions’ section. Add this to the end of the file 
  ```
    [ v3_OCSP ]
    basicConstraints = CA:FALSE
    keyUsage = nonRepudiation, digitalSignature, keyEncipherment
    extendedKeyUsage = OCSPSigning
    ```

# Create the CA

We’re going to use the OpenSSL CA script to do this.

- Change directory to `/etc/pki/tls/misc`
- Run the CA command: `./CA -newca`
- Whilst Running it you will be asked 
    - File name: Just hit enter here
    - PEM Passphrase:x this is the password you will use for the CA. Make sure it’s secure!
    - Country Name : Hit enter here
    - State or Province Name : Hit enter here
    - Locality Name (eg, city) : Hit enter here
    - Organization Name (eg, company) : Hit enter here
    - Organizational Unit Name : Hit enter here
    - Common Name (eg, your name or your server’s hostname): For this its generally considered best to set this to ca.domain, so in this case ca.test.local
    - Email Address: Hit enter here
    - A challenge password: Hit Enter here
    - An optional company name: Hit enter here
    - Enter pass phrase for /etc/pki/CA/private/./cakey.pem: Enter the CA password here

The end output should look similar to

```
[root@box1 misc]# ./CA -newca
CA certificate filename (or enter to create)

Making CA certificate ...
Generating a 2048 bit RSA private key
......................................................................+++
.......+++
writing new private key to '/etc/pki/CA/private/./cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [GB]:
State or Province Name (full name) [London]:
Locality Name (eg, city) [London]:
Organization Name (eg, company) [AGrainOfSalt]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:ca.test.local
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /etc/pki/tls/openssl.cnf
Enter pass phrase for /etc/pki/CA/private/./cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            c5:07:3c:dc:c5:8a:cb:ab
        Validity
            Not Before: Jul 21 09:37:18 2013 GMT
            Not After : Jul 20 09:37:18 2016 GMT
        Subject:
            countryName               = GB
            stateOrProvinceName       = London
            organizationName          = AGrainOfSalt
            commonName                = ca.test.local
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                1F:F8:DB:5F:7B:FE:30:29:6F:E2:E2:C8:23:DE:5B:77:AE:82:2D:F8
            X509v3 Authority Key Identifier:
                keyid:1F:F8:DB:5F:7B:FE:30:29:6F:E2:E2:C8:23:DE:5B:77:AE:82:2D:F8

            X509v3 Basic Constraints:
                CA:TRUE
Certificate is to be certified until Jul 20 09:37:18 2016 GMT (1095 days)

Write out database with 1 new entries
Data Base Updated
[root@box1 misc]#
```

At this point you have a CA setup and ready to go. You will need to ensure that the CA public certificate is installed on the browsers / devices that you will be using. This can be downloaded using a SCP client from `/etc/pki/CA/cacert.pem`

### Creating a OCSP signing certificate

In order to host an OCSP server, we have to generate a OCSP signing certificate. If you’re going to have multiple OCSP servers, you may want to have multiple certificates.

We’re going to create a directory, and a request for the certificate

```
cd /etc/pki/CA
mkdir OCSP
cd OCSP
openssl req -new -nodes -out ocsp.test.local.csr -keyout ocsp.test.local.key -extensions v3_OCSP
```

At this point we now need to sign the request and make the certificate

```
openssl ca -in ocsp.test.local.csr -out ocsp.test.local.crt -extensions v3_OCSP
```

You will be asked for

- CA Passphrase
- Sign the certificate? \[y/n\]: Say yes to this
- 1 out of 1 certificate requests certified, commit? Say yes to this as well

### Start OCSP server

At this point we now also need to run the OCSP server. Be aware that this is going to run as root in this example, which you should *NOT* do. You will want to ensure permissions are done in a way that a normal user, I’m not going to cover this at the moment though. Start the server with the following

```
openssl ocsp -index /etc/pki/CA/index.txt -port 8888 -rsigner /etc/pki/CA/OCSP/ocsp.test.local.crt -rkey /etc/pki/CA/OCSP/ocsp.test.local.key -CA /etc/pki/CA/cacert.pem -text -out /etc/pki/CA/OCSP/log.txt &
```

###  Issuing a Certificate

Now that you’ve got a working CA, you can sign any certificate requests. There are multiple ways of creating these, some software will provide you a CSR, but in this example I’m going to do this all on the CA its self (don’t do this in production!)

- Change directory to `/etc/pki/tls/misc`
- Run the CA command: `./CA -newreq`

This will give a result similar to below

```
Generating a 2048 bit RSA private key
...................................+++
.........................+++
writing new private key to 'newkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [GB]:
State or Province Name (full name) []:London
Locality Name (eg, city) [London]:
Organization Name (eg, company) [AGrainOfSalt]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:web1.test.local
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Request is in newreq.pem, private key is in newkey.pem
```

We now need to sign the certificate

- Run the CA command `./CA -sign`


This will give a result similar to

```
[root@box1 misc]# ./CA -sign
Using configuration from /etc/pki/tls/openssl.cnf
Enter pass phrase for /etc/pki/CA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            c5:07:3c:dc:c5:8a:cb:ad
        Validity
            Not Before: Jul 21 13:02:27 2013 GMT
            Not After : Jul 21 13:02:27 2014 GMT
        Subject:
            countryName               = GB
            stateOrProvinceName       = London
            localityName              = London
            organizationName          = AGrainOfSalt
            commonName                = web1.test.local
        X509v3 extensions:
            Authority Information Access:
                OCSP - URI:http://162.13.47.187:8888

            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                60:06:C1:47:5B:DE:6F:0C:64:00:DA:A9:77:05:67:AA:8F:39:C9:AF
            X509v3 Authority Key Identifier:
                keyid:1F:F8:DB:5F:7B:FE:30:29:6F:E2:E2:C8:23:DE:5B:77:AE:82:2D:F8

Certificate is to be certified until Jul 21 13:02:27 2014 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            c5:07:3c:dc:c5:8a:cb:ad
        Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=GB, ST=London, O=AGrainOfSalt, CN=ca.test.local
        Validity
            Not Before: Jul 21 13:02:27 2013 GMT
            Not After : Jul 21 13:02:27 2014 GMT
        Subject: C=GB, ST=London, L=London, O=AGrainOfSalt, CN=web1.test.local
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:97:d0:a4:0e:c3:61:c0:b0:8e:af:5b:2a:96:85:
                    fc:e8:e9:23:3c:97:f0:2f:15:5e:a3:00:49:0c:b6:
                    2d:5a:2c:f0:ff:ca:2c:49:f1:ee:d8:3a:0b:f2:ab:
                    9b:96:f8:a2:cf:a2:6a:82:63:3b:7a:9b:7c:b1:4c:
                    4a:65:c8:70:cc:7c:90:1b:7b:b3:a0:6c:91:5d:1e:
                    12:93:31:d0:68:bb:33:6e:e7:54:91:fc:f8:e1:b3:
                    3e:26:33:4c:d0:d7:ea:fd:6f:1f:b6:a4:cf:1b:82:
                    03:41:58:d5:47:4d:f6:a3:50:a5:4e:92:74:96:c6:
                    1f:b2:3d:33:00:25:62:35:2b:89:6e:60:37:d0:44:
                    d4:11:89:0d:21:ed:3f:d3:54:db:c5:21:5f:43:3f:
                    bd:2b:e6:a9:48:f1:dd:11:0f:a2:f2:d9:7a:2f:0b:
                    78:8c:98:b2:3a:4b:23:fb:16:41:9e:b8:69:ee:e5:
                    22:bc:67:49:40:fe:eb:13:d6:45:50:3b:cc:14:b3:
                    1b:ba:e1:5d:89:33:ed:8e:6a:05:36:0e:c2:bb:21:
                    9f:6f:6b:17:99:a2:53:cc:69:24:03:95:db:89:79:
                    46:8d:05:ed:1d:b6:c3:22:89:9a:43:e5:ff:c1:b7:
                    58:f1:40:ab:b1:e5:ca:c1:ec:64:59:a7:fb:53:44:
                    8e:27
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            Authority Information Access:
                OCSP - URI:http://162.13.47.187:8888

            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                60:06:C1:47:5B:DE:6F:0C:64:00:DA:A9:77:05:67:AA:8F:39:C9:AF
            X509v3 Authority Key Identifier:
                keyid:1F:F8:DB:5F:7B:FE:30:29:6F:E2:E2:C8:23:DE:5B:77:AE:82:2D:F8

    Signature Algorithm: sha1WithRSAEncryption
        88:05:46:ab:a5:50:08:56:50:ca:4b:15:af:1e:84:ab:b1:d0:
        b0:b9:81:52:0b:f5:e6:28:51:71:a6:11:64:46:04:42:84:eb:
        84:5f:d1:87:64:18:60:1b:31:7a:13:b2:9d:10:bd:56:7a:7f:
        2e:88:23:55:52:a4:a2:9d:d0:8c:70:b7:0a:69:50:96:fa:54:
        be:6b:bd:24:25:9d:59:52:30:33:92:cc:63:3e:5c:47:87:e3:
        ca:d0:55:09:c3:2a:4b:fc:f0:b8:34:3f:1a:d3:b9:3a:66:7f:
        86:d1:8c:08:4c:cf:19:3d:c5:c7:3f:b1:73:7b:bd:54:73:c6:
        65:74:8e:8a:17:5a:e9:d9:bd:91:39:8b:ae:46:10:52:d9:03:
        db:51:3a:14:41:2b:96:6c:ea:db:c7:20:89:48:ae:32:fe:05:
        66:2b:5c:48:19:7b:b8:45:99:7a:b1:45:b0:66:06:31:3d:86:
        c5:c3:7b:99:d5:cf:1f:9d:64:69:bf:60:1a:d8:03:7e:75:e2:
        44:b7:41:36:aa:b8:c0:df:9c:24:74:eb:ab:b6:b3:1b:74:be:
        73:bf:52:bd:b7:de:31:81:ff:df:b6:f4:b9:8f:9f:22:93:3b:
        12:5c:a4:6a:3f:aa:8e:8a:7c:69:65:e9:65:f7:98:44:1b:59:
        7d:e4:ca:e0
-----BEGIN CERTIFICATE-----
MIID5DCCAsygAwIBAgIJAMUHPNzFisutMA0GCSqGSIb3DQEBBQUAME0xCzAJBgNV
BAYTAkdCMQ8wDQYDVQQIDAZMb25kb24xFTATBgNVBAoMDEFHcmFpbk9mU2FsdDEW
MBQGA1UEAwwNY2EudGVzdC5sb2NhbDAeFw0xMzA3MjExMzAyMjdaFw0xNDA3MjEx
MzAyMjdaMGAxCzAJBgNVBAYTAkdCMQ8wDQYDVQQIDAZMb25kb24xDzANBgNVBAcM
BkxvbmRvbjEVMBMGA1UECgwMQUdyYWluT2ZTYWx0MRgwFgYDVQQDDA93ZWIxLnRl
c3QubG9jYWwwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCX0KQOw2HA
sI6vWyqWhfzo6SM8l/AvFV6jAEkMti1aLPD/yixJ8e7YOgvyq5uW+KLPomqCYzt6
m3yxTEplyHDMfJAbe7OgbJFdHhKTMdBouzNu51SR/Pjhsz4mM0zQ1+r9bx+2pM8b
ggNBWNVHTfajUKVOknSWxh+yPTMAJWI1K4luYDfQRNQRiQ0h7T/TVNvFIV9DP70r
5qlI8d0RD6Ly2XovC3iMmLI6SyP7FkGeuGnu5SK8Z0lA/usT1kVQO8wUsxu64V2J
M+2OagU2DsK7IZ9vaxeZolPMaSQDlduJeUaNBe0dtsMiiZpD5f/Bt1jxQKux5crB
7GRZp/tTRI4nAgMBAAGjgbMwgbAwNQYIKwYBBQUHAQEEKTAnMCUGCCsGAQUFBzAB
hhlodHRwOi8vMTYyLjEzLjQ3LjE4Nzo4ODg4MAkGA1UdEwQCMAAwLAYJYIZIAYb4
QgENBB8WHU9wZW5TU0wgR2VuZXJhdGVkIENlcnRpZmljYXRlMB0GA1UdDgQWBBRg
BsFHW95vDGQA2ql3BWeqjznJrzAfBgNVHSMEGDAWgBQf+Ntfe/4wKW/i4sgj3lt3
roIt+DANBgkqhkiG9w0BAQUFAAOCAQEAiAVGq6VQCFZQyksVrx6Eq7HQsLmBUgv1
5ihRcaYRZEYEQoTrhF/Rh2QYYBsxehOynRC9Vnp/LogjVVKkop3QjHC3CmlQlvpU
vmu9JCWdWVIwM5LMYz5cR4fjytBVCcMqS/zwuDQ/GtO5OmZ/htGMCEzPGT3Fxz+x
c3u9VHPGZXSOihda6dm9kTmLrkYQUtkD21E6FEErlmzq28cgiUiuMv4FZitcSBl7
uEWZerFFsGYGMT2GxcN7mdXPH51kab9gGtgDfnXiRLdBNqq4wN+cJHTrq7azG3S+
c79SvbfeMYH/37b0uY+fIpM7Elykaj+qjop8aWXpZfeYRBtZfeTK4A==
-----END CERTIFICATE-----
Signed certificate is in newcert.pem
```

The certificate now exists and can be seen in newcert.pem (and the key in newkey.pem)

### Checking OCSP status

We can now check to see if the above certificate is valid via OCSP:

`openssl ocsp -CAfile /etc/pki/CA/cacert.pem -issuer /etc/pki/CA/cacert.pem -cert newcert.pem -url http://127.0.0.1:8888 -resp_text`

This will return an address similar to below:

```
[root@box1 misc]# openssl ocsp -CAfile /etc/pki/CA/cacert.pem -issuer /etc/pki/CA/cacert.pem -cert newcert.pem -url http://127.0.0.1:8888 -resp_text
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: C = GB, ST = London, O = AGrainOfSalt, CN = ocsp.test.local
    Produced At: Jul 21 13:06:37 2013 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: CD84AAE91120F3595F8B572F367D17BBD2B7D51E
      Issuer Key Hash: 1FF8DB5F7BFE30296FE2E2C823DE5B77AE822DF8
      Serial Number: C5073CDCC58ACBAD
    Cert Status: good
    This Update: Jul 21 13:06:37 2013 GMT

    Response Extensions:
        OCSP Nonce:
            04108CFBA6175ED9F3E58C50C39889E2B484
    Signature Algorithm: sha1WithRSAEncryption
        ba:2e:74:c5:3a:57:88:f1:7a:0f:d4:e9:01:f0:7c:31:2e:a3:
        31:cb:25:c4:8a:91:e6:05:88:39:3a:18:ef:89:eb:02:6a:46:
        12:4e:12:90:85:55:3d:22:67:aa:68:ab:11:08:d9:89:29:20:
        b6:36:78:89:3d:5d:c5:9b:7a:94:1a:e4:4f:48:b2:45:53:0b:
        86:65:fb:64:cc:e0:15:a6:32:7a:2d:00:c1:a6:c7:25:c1:a0:
        f8:4b:67:42:77:ad:cf:ab:01:8a:0c:3a:65:18:3f:d7:85:26:
        6c:a4:d0:ab:8c:40:2e:2f:f7:3f:0f:9d:f7:6c:80:d8:52:0e:
        8b:21:5d:3c:cb:d9:6b:c3:87:05:eb:00:4d:6a:b8:74:d5:28:
        fb:08:63:4a:b4:4b:c8:67:c9:01:f7:51:75:6e:50:15:bd:db:
        a9:cc:83:26:63:67:ce:2b:96:d6:e2:e8:df:01:82:36:75:23:
        44:30:d9:8e:32:a9:74:ce:69:4c:f3:79:80:a5:73:75:b3:bf:
        c2:b4:66:e8:70:b7:4c:9c:4a:3d:2a:ff:98:25:d2:19:86:3b:
        70:78:e0:7c:72:82:69:e0:08:6e:1d:a5:e8:d2:6f:4d:e7:a5:
        a2:15:05:5f:98:b5:1e:20:13:54:e2:e2:62:4d:38:5d:e8:b1:
        56:15:97:00
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            c5:07:3c:dc:c5:8a:cb:ac
        Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=GB, ST=London, O=AGrainOfSalt, CN=ca.test.local
        Validity
            Not Before: Jul 21 10:16:20 2013 GMT
            Not After : Jul 21 10:16:20 2014 GMT
        Subject: C=GB, ST=London, O=AGrainOfSalt, CN=ocsp.test.local
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:f2:1e:67:ef:42:45:2c:3f:ed:09:c6:d2:11:ca:
                    28:ce:16:ac:2b:15:7a:25:34:dc:35:2d:d9:6e:60:
                    b6:06:ed:ea:fc:ec:0e:37:0f:44:bc:25:02:b6:41:
                    a2:89:c8:58:e3:cc:0d:c4:b6:b1:e3:08:b0:2d:6c:
                    85:52:0a:3a:4c:ae:ad:1b:8a:d7:0b:fd:da:f7:85:
                    94:66:e9:25:48:a3:d6:07:27:e6:51:ee:03:96:db:
                    80:ec:60:00:27:ac:8f:93:63:e6:8a:22:d9:45:da:
                    8a:93:67:aa:d8:6a:00:32:0c:8c:84:87:47:30:a2:
                    96:21:44:e1:cd:19:a9:bb:0e:9a:70:5b:fc:4e:a0:
                    79:ca:27:b0:5f:c0:7c:3d:23:c7:ae:26:c2:20:86:
                    97:89:f6:a9:32:0e:e9:10:c1:c0:f3:51:4e:61:a7:
                    6c:ea:84:d2:d2:7d:ec:6d:58:f8:5c:c8:4e:37:f1:
                    05:03:52:07:fc:96:dd:69:8b:6e:30:d9:75:0a:4c:
                    17:74:89:4c:bc:06:b0:d9:d3:03:a3:bd:75:c5:9a:
                    2c:7b:75:f8:6f:e4:44:a9:ef:08:49:aa:88:49:1a:
                    41:d7:98:8e:8f:ac:be:bb:e1:66:e0:33:05:2d:64:
                    7e:90:fc:62:eb:bc:18:45:7d:cb:23:bb:af:3c:2c:
                    0b:a1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Key Usage:
                Digital Signature, Non Repudiation, Key Encipherment
            X509v3 Extended Key Usage:
                OCSP Signing
    Signature Algorithm: sha1WithRSAEncryption
        92:e0:49:92:23:63:fe:04:8d:5d:cf:79:fb:60:b6:3d:63:6c:
        43:8d:5a:95:09:ae:c0:75:ea:c3:08:9e:f1:1e:f3:bf:34:c4:
        f2:d7:93:58:55:b4:c5:3a:16:48:76:d8:04:b3:dc:69:67:ce:
        e3:16:6a:e9:47:06:33:16:9e:aa:e2:99:49:74:9b:2c:22:99:
        3e:5b:50:57:2b:46:da:25:d6:e7:5b:4e:36:bd:82:ac:3a:d0:
        f2:73:c0:c4:1e:27:57:63:c5:fb:0c:19:86:2b:45:dc:cb:f1:
        b5:9c:fa:22:da:d9:0c:e4:e0:5a:53:87:e1:6e:d9:7b:d9:7d:
        cf:33:90:66:07:b3:9a:38:81:63:6c:c8:7f:a4:d2:8c:15:23:
        68:18:4a:ee:e4:61:b4:2d:29:43:75:8d:67:8a:08:55:30:e9:
        09:77:cc:24:71:1c:66:fe:77:25:28:89:f5:80:a2:7f:2d:be:
        81:22:51:33:c4:67:b9:67:1e:51:81:93:da:d7:f5:b4:80:fb:
        07:8d:c8:64:a2:98:9a:4c:55:97:88:6a:51:30:2c:2d:ba:ed:
        90:ec:bf:24:18:b0:17:c9:ae:95:85:8d:22:6b:b3:5f:9b:57:
        b7:96:38:ab:0f:75:6b:47:57:56:24:d5:01:59:87:4f:76:e6:
        f5:72:97:12
-----BEGIN CERTIFICATE-----
MIIDTjCCAjagAwIBAgIJAMUHPNzFisusMA0GCSqGSIb3DQEBBQUAME0xCzAJBgNV
BAYTAkdCMQ8wDQYDVQQIDAZMb25kb24xFTATBgNVBAoMDEFHcmFpbk9mU2FsdDEW
MBQGA1UEAwwNY2EudGVzdC5sb2NhbDAeFw0xMzA3MjExMDE2MjBaFw0xNDA3MjEx
MDE2MjBaME8xCzAJBgNVBAYTAkdCMQ8wDQYDVQQIDAZMb25kb24xFTATBgNVBAoM
DEFHcmFpbk9mU2FsdDEYMBYGA1UEAwwPb2NzcC50ZXN0LmxvY2FsMIIBIjANBgkq
hkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA8h5n70JFLD/tCcbSEcoozhasKxV6JTTc
NS3ZbmC2Bu3q/OwONw9EvCUCtkGiichY48wNxLax4wiwLWyFUgo6TK6tG4rXC/3a
94WUZuklSKPWByfmUe4DltuA7GAAJ6yPk2PmiiLZRdqKk2eq2GoAMgyMhIdHMKKW
IUThzRmpuw6acFv8TqB5yiewX8B8PSPHribCIIaXifapMg7pEMHA81FOYads6oTS
0n3sbVj4XMhON/EFA1IH/JbdaYtuMNl1CkwXdIlMvAaw2dMDo711xZose3X4b+RE
qe8ISaqISRpB15iOj6y+u+Fm4DMFLWR+kPxi67wYRX3LI7uvPCwLoQIDAQABoy8w
LTAJBgNVHRMEAjAAMAsGA1UdDwQEAwIF4DATBgNVHSUEDDAKBggrBgEFBQcDCTAN
BgkqhkiG9w0BAQUFAAOCAQEAkuBJkiNj/gSNXc95+2C2PWNsQ41alQmuwHXqwwie
8R7zvzTE8teTWFW0xToWSHbYBLPcaWfO4xZq6UcGMxaequKZSXSbLCKZPltQVytG
2iXW51tONr2CrDrQ8nPAxB4nV2PF+wwZhitF3MvxtZz6ItrZDOTgWlOH4W7Ze9l9
zzOQZgezmjiBY2zIf6TSjBUjaBhK7uRhtC0pQ3WNZ4oIVTDpCXfMJHEcZv53JSiJ
9YCify2+gSJRM8RnuWceUYGT2tf1tID7B43IZKKYmkxVl4hqUTAsLbrtkOy/JBiw
F8mulYWNImuzX5tXt5Y4qw91a0dXViTVAVmHT3bm9XKXEg==
-----END CERTIFICATE-----
Response verify OK
newcert.pem: good
        This Update: Jul 21 13:06:37 2013 GMT
```

#  Revoking a certificate.

Oh no! The certificate above has been compromised. We need to revoke it. This isn’t as difficult as you may think. We have a copy of all of the certificates on the CA. If we look at the certificate serial number (c5:07:3c:dc:c5:8a:cb:ad in this case) this file should exist in `/etc/pki/CA/newcerts/` To revoke you need to

- Revoke the certificate 
  ```
  openssl ca -revoke /etc/pki/CA/newcerts/C5073CDCC58ACBAD.pem
  ```
- Verify that the certifcate is revoked 
  ```
  openssl ocsp -CAfile /etc/pki/CA/cacert.pem -issuer /etc/pki/CA/cacert.pem -cert /etc/pki/CA/newcerts/C5073CDCC58ACBAD.pem -url http://127.0.0.1:8888 -resp_text
  ```