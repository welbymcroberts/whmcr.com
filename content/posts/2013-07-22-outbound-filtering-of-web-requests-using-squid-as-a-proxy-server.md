+++
title = "Outbound filtering of Web requests using Squid as a Proxy server"
#description = ""
date = "2013-07-22T07:00:16+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=19",
    "/2013/07/22/outbound-filtering-of-web-requests-using-squid-as-a-proxy-server/"
]
in_search_index = true


tags = [
    "hacks",
    "infrastructure",
    "linux",
    "proxy",
    "squid",
    "tls",
    "ssl"
]
[extra]
+++

Frequently in my line of work I’ll be asked about filtering of outbound traffic from application servers. There are two schools of thought here, one is that an app server can have unfiltered access to the internet, and the other that the app server should have as little access to any resources (both inside and outside of the solution) as needed to preform its role.

This generally isn’t an issue if site to site VPNs, static IPs or similar are being used on the destination side. But what happens if your application requires access to something like Youtube, Facebook or Flickr. As these cloud services are not managed by the customer, we have no idea if they are on static IP addresses (and in the case of flickr, they do seem to change moderately frequently).

With this in mind a traditional Layer3/Layer4 firewall is only going to be able to handle this if it supports DNS resolution in its access-list set, and unfortunately (but for good reason) this is not a common feature. Cisco did [introduce](https://supportforums.cisco.com/docs/DOC-17014) this to the ASA firewalls in 8.4, however I personally have not used this, so at the moment its still a bit of an unknown and I can’t recommend it to a customer.

There is however another way of doing this, whilst it might not be a perfect situation, it does at least allow you to filter outbound traffic.

The [Squid](http://www.squid-cache.org/) proxy server has been around for quite some time and is quite a stable product, both in the forward (outbound) and reverse (inbound) HTTP proxy space. We’re going to use this to preform our outbound proxying. It is possible to use commercial products like a BlueCoat proxy, however I’m going to concentrate on the FOSS solution here.


# Prerequisites

Before we start we need to have the following:

- A Linux Server (for this example I’m going to be using CentOS 6.4, however any linux distribution should work)

# Installing Squid

This is a really simple task on most linux distributions, as not only has squid been since the early 90’s, it’s also really popular! You can use the package manager to install squid on most distributions

```
yum install -y squid
```

You should get a response similar to below:

```
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package squid.x86_64 7:3.1.10-18.el6_4 will be installed
--> Processing Dependency: perl(DBI) for package: 7:squid-3.1.10-18.el6_4.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: 7:squid-3.1.10-18.el6_4.x86_64
--> Running transaction check
---> Package libtool-ltdl.x86_64 0:2.2.6-15.5.el6 will be installed
---> Package perl-DBI.x86_64 0:1.609-4.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================================================================================================================================
 Package                                          Arch                                       Version                                               Repository                                   Size
=====================================================================================================================================================================================================
Installing:
 squid                                            x86_64                                     7:3.1.10-18.el6_4                                     updates                                     1.7 M
Installing for dependencies:
 libtool-ltdl                                     x86_64                                     2.2.6-15.5.el6                                        base                                         44 k
 perl-DBI                                         x86_64                                     1.609-4.el6                                           base                                        705 k

Transaction Summary
=====================================================================================================================================================================================================
Install       3 Package(s)

Total download size: 2.5 M
Installed size: 7.5 M
Downloading Packages:
(1/3): libtool-ltdl-2.2.6-15.5.el6.x86_64.rpm                                                                                                                                 |  44 kB     00:00
(2/3): perl-DBI-1.609-4.el6.x86_64.rpm                                                                                                                                        | 705 kB     00:00
(3/3): squid-3.1.10-18.el6_4.x86_64.rpm                                                                                                                                       | 1.7 MB     00:01
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                942 kB/s | 2.5 MB     00:02
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : perl-DBI-1.609-4.el6.x86_64                                                                                                                                                       1/3
  Installing : libtool-ltdl-2.2.6-15.5.el6.x86_64                                                                                                                                                2/3
  Installing : 7:squid-3.1.10-18.el6_4.x86_64                                                                                                                                                    3/3
  Verifying  : 7:squid-3.1.10-18.el6_4.x86_64                                                                                                                                                    1/3
  Verifying  : libtool-ltdl-2.2.6-15.5.el6.x86_64                                                                                                                                                2/3
  Verifying  : perl-DBI-1.609-4.el6.x86_64                                                                                                                                                       3/3

Installed:
  squid.x86_64 7:3.1.10-18.el6_4

Dependency Installed:
  libtool-ltdl.x86_64 0:2.2.6-15.5.el6                                                                 perl-DBI.x86_64 0:1.609-4.el6

Complete!
```

We now would need to configure squid to start on boot

```
chkconfig squid on
```

###  SSL Proxying

Squid has a rather nice feature called [SSLBump](http://wiki.squid-cache.org/Features/SslBump) which allows us to preform a Man In the Middle SSL Proxy. Privacy issues aside on this feature (after all we’re using it for servers not for end users) this is going to work for us from the server side of things. One key thing to note is we have to trust the CA, that we’re going to generate, on all applications / servers. I’m not going to cover how to do this in this post.

Normally when we create an SSL certificate we’d do this with a specific domain, however as we’re going to be proxying for all domains we’re going to use a wildcard certificate. For the "Common Name" or Server name, we need to chose "*" as the value.

In order to create the CA you can follow the [following post](http://withagrainofsalt.co.uk/2013/07/21/creating-a-ca-using-openssl-with-ocsp/ "Creating a CA using OpenSSL – with OCSP"). One point of note is to ensure that you do not do this on the Squid server, as this would mean that should the server be compromised, the CA (which is trusted on multiple servers) is now trusted as well.

We need to create the certificate using the CA script as per the above post. ```CA -newreq``` This will look similar to

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
State or Province Name (full name) [London]:
Locality Name (eg, city) [London]:
Organization Name (eg, company) [AGrainOfSalt]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:*
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Request is in newreq.pem, private key is in newkey.pem
```

Once this is completed you’ll need to sign this with the ```CA -sign``` command

```
Using configuration from /etc/pki/tls/openssl.cnf
Enter pass phrase for /etc/pki/CA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number:
            c5:07:3c:dc:c5:8a:cb:ae
        Validity
            Not Before: Jul 21 16:45:48 2013 GMT
            Not After : Jul 21 16:45:48 2014 GMT
        Subject:
            countryName               = GB
            localityName              = London
            organizationName          = AGrainOfSalt
            commonName                = *
        X509v3 extensions:
            Authority Information Access:
                OCSP - URI:http://162.13.47.187:8888

            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                E8:30:79:47:75:8D:06:0C:CB:9E:84:47:65:61:D4:27:8D:61:52:D2
            X509v3 Authority Key Identifier:
                keyid:1F:F8:DB:5F:7B:FE:30:29:6F:E2:E2:C8:23:DE:5B:77:AE:82:2D:F8

Certificate is to be certified until Jul 21 16:45:48 2014 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            c5:07:3c:dc:c5:8a:cb:ae
        Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=GB, ST=London, O=AGrainOfSalt, CN=ca.test.local
        Validity
            Not Before: Jul 21 16:45:48 2013 GMT
            Not After : Jul 21 16:45:48 2014 GMT
        Subject: C=GB, L=London, O=AGrainOfSalt, CN=*
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:e0:e7:17:43:65:55:7c:da:56:46:4c:44:11:2a:
                    96:8b:b9:c0:d0:55:10:d7:f7:c7:ca:29:3b:2d:1d:
                    08:46:00:db:23:78:58:04:36:35:79:ca:5a:4f:a3:
                    81:31:c7:4c:ec:a9:07:46:af:60:98:9d:ff:06:1c:
                    58:8f:16:53:97:1b:f1:b0:17:b5:9a:5c:eb:eb:7c:
                    c5:a9:3a:93:e2:8b:23:ad:d9:54:1e:c3:99:2c:8f:
                    24:1e:b0:0b:d3:3a:2f:b3:72:79:f5:71:d9:3b:52:
                    de:78:18:c4:41:8e:dc:5c:4c:96:da:90:75:1f:21:
                    f5:83:91:30:64:11:fe:af:b1:e1:fb:4d:4a:1f:06:
                    a4:7a:b0:bf:91:bc:74:5b:27:88:e3:0d:2e:1c:3c:
                    3b:e8:6c:7b:32:90:60:c8:4c:2a:db:84:fc:c2:53:
                    7a:6c:4b:ae:45:4a:86:4a:c6:a4:88:24:33:a6:f4:
                    de:8b:56:3b:59:3f:de:92:c4:9d:f2:d4:2b:53:da:
                    62:05:94:7e:bc:f9:f8:45:53:88:56:13:30:67:ed:
                    6c:e7:0c:32:f8:53:03:49:3e:c3:c2:b6:06:83:e0:
                    d1:80:51:f4:86:f7:52:b0:7d:05:34:39:df:3b:46:
                    29:62:24:43:9a:b9:fe:ac:10:30:17:3a:e3:9a:d5:
                    ab:69
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            Authority Information Access:
                OCSP - URI:http://162.13.47.187:8888

            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                E8:30:79:47:75:8D:06:0C:CB:9E:84:47:65:61:D4:27:8D:61:52:D2
            X509v3 Authority Key Identifier:
                keyid:1F:F8:DB:5F:7B:FE:30:29:6F:E2:E2:C8:23:DE:5B:77:AE:82:2D:F8

    Signature Algorithm: sha1WithRSAEncryption
        83:2c:00:e4:58:1f:db:02:aa:a8:ff:52:45:d3:63:4f:8f:47:
        8f:65:1e:21:c7:d7:c2:76:df:03:af:64:c1:e0:2c:d5:92:44:
        35:a4:c3:02:78:0a:43:0f:ed:91:03:2a:f8:00:5c:97:f7:fc:
        6f:81:69:96:3d:c3:ce:80:f2:d2:0d:de:5c:2d:f0:27:ca:ba:
        1c:b4:09:8a:cc:b6:76:06:9f:a9:ad:a5:bf:9c:7d:9d:c5:f1:
        32:d4:d6:30:bf:bc:57:19:76:06:51:ee:e8:8b:f2:a1:4c:f7:
        69:ef:d4:96:58:4d:5a:de:98:c5:f4:17:af:b5:d2:cc:26:f0:
        69:43:72:77:4a:e7:cf:79:62:b6:a4:47:75:4b:29:dc:2f:6f:
        e6:c6:d5:1a:79:e2:1d:bf:f8:82:04:fe:d0:21:7a:8b:4e:1e:
        93:10:f4:81:d6:9d:41:9a:70:02:e7:3f:22:1a:d8:a6:2e:21:
        8a:b1:34:03:1c:83:ca:8c:19:59:1b:d6:85:f7:eb:e2:a7:32:
        d9:61:5c:e6:68:b3:ef:ba:27:4a:3f:ff:5a:6e:d2:60:36:bb:
        a2:0a:ba:aa:4f:d9:22:7e:ab:7e:78:80:87:6d:51:92:44:61:
        e6:aa:63:fa:e5:13:88:c9:f9:de:90:31:9f:28:78:ca:79:74:
        ff:0e:a4:0b
-----BEGIN CERTIFICATE-----
MIIDxTCCAq2gAwIBAgIJAMUHPNzFisuuMA0GCSqGSIb3DQEBBQUAME0xCzAJBgNV
BAYTAkdCMQ8wDQYDVQQIDAZMb25kb24xFTATBgNVBAoMDEFHcmFpbk9mU2FsdDEW
MBQGA1UEAwwNY2EudGVzdC5sb2NhbDAeFw0xMzA3MjExNjQ1NDhaFw0xNDA3MjEx
NjQ1NDhaMEExCzAJBgNVBAYTAkdCMQ8wDQYDVQQHDAZMb25kb24xFTATBgNVBAoM
DEFHcmFpbk9mU2FsdDEKMAgGA1UEAwwBKjCCASIwDQYJKoZIhvcNAQEBBQADggEP
ADCCAQoCggEBAODnF0NlVXzaVkZMRBEqlou5wNBVENf3x8opOy0dCEYA2yN4WAQ2
NXnKWk+jgTHHTOypB0avYJid/wYcWI8WU5cb8bAXtZpc6+t8xak6k+KLI63ZVB7D
mSyPJB6wC9M6L7NyefVx2TtS3ngYxEGO3FxMltqQdR8h9YORMGQR/q+x4ftNSh8G
pHqwv5G8dFsniOMNLhw8O+hsezKQYMhMKtuE/MJTemxLrkVKhkrGpIgkM6b03otW
O1k/3pLEnfLUK1PaYgWUfrz5+EVTiFYTMGftbOcMMvhTA0k+w8K2BoPg0YBR9Ib3
UrB9BTQ53ztGKWIkQ5q5/qwQMBc645rVq2kCAwEAAaOBszCBsDA1BggrBgEFBQcB
AQQpMCcwJQYIKwYBBQUHMAGGGWh0dHA6Ly8xNjIuMTMuNDcuMTg3Ojg4ODgwCQYD
VR0TBAIwADAsBglghkgBhvhCAQ0EHxYdT3BlblNTTCBHZW5lcmF0ZWQgQ2VydGlm
aWNhdGUwHQYDVR0OBBYEFOgweUd1jQYMy56ER2Vh1CeNYVLSMB8GA1UdIwQYMBaA
FB/42197/jApb+LiyCPeW3eugi34MA0GCSqGSIb3DQEBBQUAA4IBAQCDLADkWB/b
Aqqo/1JF02NPj0ePZR4hx9fCdt8Dr2TB4CzVkkQ1pMMCeApDD+2RAyr4AFyX9/xv
gWmWPcPOgPLSDd5cLfAnyroctAmKzLZ2Bp+praW/nH2dxfEy1NYwv7xXGXYGUe7o
i/KhTPdp79SWWE1a3pjF9BevtdLMJvBpQ3J3SufPeWK2pEd1SyncL2/mxtUaeeId
v/iCBP7QIXqLTh6TEPSB1p1BmnAC5z8iGtimLiGKsTQDHIPKjBlZG9aF9+vipzLZ
YVzmaLPvuidKP/9abtJgNruiCrqqT9kifqt+eICHbVGSRGHmqmP65ROIyfnekDGf
KHjKeXT/DqQL
-----END CERTIFICATE-----
Signed certificate is in newcert.pem
```

Once this is completed, ensure that newcert.pem and newkey.pem are copied to the squid server. You will then also need to remove the passphrase from the key.

```
openssl rsa -in newkey.pem -out sslbump.pem
```

Once this is done, you’ll need to then also copy the cert into the same file

```
cat newcert.pem >> sslbump.pem
```

### Configuring Squid

We’re going to make a very simple squid config, allowing access from the App servers to youtube.com, but no other hosts. Replace ```/etc/squid/squid.conf``` with the following

```
acl manager proto cache_object
acl localhost src 127.0.0.1/32 ::1
acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 443         # https
acl CONNECT method CONNECT
acl youtube dstdomain www.youtube.com
acl app_server src 192.168.0.3/32

http_access deny manager
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports

ssl_bump allow app_server youtube
http_access allow app_server youtube
http_access deny all
ssl_bump deny all

hierarchy_stoplist cgi-bin ?
coredump_dir /var/spool/squid

# Listen on 3128 and do ssl-bump
http_port 3128 ssl-bump cert=/etc/squid/sslbump.pem
# Don't cache / refresh anything
refresh_pattern .               0       0%      0
```

# Testing Squid

We’re going to use the curl command to test that the ACLs are working

First lets test google, this should fail. We specify the proxy with the -x flag

```
[root@box1 squid]# curl -I www.google.com -x 192.168.0.2:3128
HTTP/1.0 403 Forbidden
Server: squid/3.1.10
Mime-Version: 1.0
Date: Sun, 21 Jul 2013 17:23:36 GMT
Content-Type: text/html
Content-Length: 3274
X-Squid-Error: ERR_ACCESS_DENIED 0
Vary: Accept-Language
Content-Language: en
X-Cache: MISS from box1.agrainofsalt.com
X-Cache-Lookup: NONE from box1.agrainofsalt.com:3128
Via: 1.0 box1.agrainofsalt.com (squid/3.1.10)
Connection: keep-alive
```

As you can see we get a 403 on this from Squid

Lets now try http access to youtube.com

```
[root@box1 squid]# curl -I www.youtube.com -x 192.168.0.2:3128
HTTP/1.0 200 OK
Date: Sun, 21 Jul 2013 17:25:37 GMT
Server: gwiseguy/2.0
X-Frame-Options: SAMEORIGIN
X-YouTube-Other-Cookies: VISITOR_INFO1_LIVE=A4yQlrbOatM;PREF=f1=50000000
P3P: CP="This is not a P3P policy! See http://support.google.com/accounts/bin/answer.py?answer=151657&hl=en-GB for more info."
X-Content-Type-Options: nosniff
Cache-Control: no-cache
Expires: Tue, 27 Apr 1971 19:44:06 EST
Set-Cookie: YSC=zqSQpd08t-o; path=/; domain=.youtube.com; httponly
Set-Cookie: PREF=f1=50000000; path=/; domain=.youtube.com; expires=Sat, 22-Mar-2014 05:18:37 GMT
Set-Cookie: VISITOR_INFO1_LIVE=A4yQlrbOatM; path=/; domain=.youtube.com; expires=Sat, 22-Mar-2014 05:18:36 GMT
Content-Type: text/html; charset=utf-8
X-XSS-Protection: 1; mode=block
X-Cache: MISS from box1.agrainofsalt.com
X-Cache-Lookup: MISS from box1.agrainofsalt.com:3128
Via: 1.0 box1.agrainofsalt.com (squid/3.1.10)
Connection: keep-alive
```

This works as expected. Lets try https to youtube.com now!

```
[root@box1 squid]# curl -I https://www.youtube.com/ -x 127.0.0.1:3128
HTTP/1.0 200 Connection established

curl: (60) Peer certificate cannot be authenticated with known CA certificates
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```

This has failed as we’ve not got the CA certificate in the bundle that curl uses, lets get curl to ignore the SSL certificate

```
[root@box1 squid]# curl -Ik https://www.youtube.com/ -x 127.0.0.1:3128
HTTP/1.0 200 Connection established

HTTP/1.0 503 Service Unavailable
Server: squid/3.1.10
Mime-Version: 1.0
Date: Sun, 21 Jul 2013 18:36:00 GMT
Content-Type: text/html
Content-Length: 3408
X-Squid-Error: ERR_CANNOT_FORWARD 0
Vary: Accept-Language
Content-Language: en
X-Cache: MISS from box1.agrainofsalt.com
X-Cache-Lookup: MISS from box1.agrainofsalt.com:3128
Via: 1.0 box1.agrainofsalt.com (squid/3.1.10)
Connection: keep-alive
```

Now lets just make sure that other https sites don’t work.

```
[root@box1 squid]# curl -Ik https://www.gmail.com/ -x 127.0.0.1:3128
HTTP/1.0 403 Forbidden
Server: squid/3.1.10
Mime-Version: 1.0
Date: Sun, 21 Jul 2013 18:36:28 GMT
Content-Type: text/html
Content-Length: 3261
X-Squid-Error: ERR_ACCESS_DENIED 0
Vary: Accept-Language
Content-Language: en
X-Cache: MISS from box1.agrainofsalt.com
X-Cache-Lookup: NONE from box1.agrainofsalt.com:3128
Via: 1.0 box1.agrainofsalt.com (squid/3.1.10)
Connection: keep-alive

curl: (56) Received HTTP code 403 from proxy after CONNECT
```

###  Forwarding all traffic via the Proxy server

Now the way that this is done depends on the firewall or router in use. What we need to achieve is to either D-NAT or redirect all traffic on port 80 / 443 outbound to the Squid server.

For a Cisco ASA there is a guide on how to do this with [WCCP](https://supportforums.cisco.com/docs/DOC-12623)

For a Linux based device you would want to have a IPTables rule similar to

```
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to 192.168.0.2:3128 -s ! 192.168.0.2
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 443 -j DNAT --to 192.168.0.2:3128 -s ! 192.168.0.2
```