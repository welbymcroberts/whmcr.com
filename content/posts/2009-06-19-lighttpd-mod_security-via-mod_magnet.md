+++
title = "Lighttpd: mod_security via mod_magnet"
#description = ""
date = "2009-06-19T12:06:33+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=106",
    "/2009/06/19/lighttpd-mod_security-via-mod_magnet/"
]
in_search_index = true

tags = [
    "Lighttpd",
    "Security",
    "Linux",
    "Webserver"
]
[extra]
+++

In most large enterprises there is a requirement to comply with various standards. The hot potato in the Ecommerce space at the moment (and has been for a few years!) is PCI-DSS.

At $WORK we have to comply with PCI-DSS with the full audit and similar occurring due to the number of transactions we perform. Recently we’ve deployed lighttpd for one of our platforms, which has caused an issue for our Information Security Officers and Compliance staff.

PCI-DSS 6.6 requires EITHER a Code review to be preformed, which whilst this may seem to be an easy task, when you’re talking about complex enterprise applications following a very……… agile development process it’s not always an option. The other option is to use a WAF (Web Application Firewall). Now there are multiple products available that sit upstream and perform this task. There is however an issue if you use SSL for your traffic. Most WAF will not do the SSL decryption / reencryption between the client and server (effectively becoming a Man in the Middle). There are however a few products which do this, F5 networks’ ASM being one that springs to mind. Unfortunately this isn’t always an option due to licensing fees and similar. An alternative is to run a WAF on the server its self. A common module for this is Mod_Security for Apache. Unfortunately, a similar module does not exist for Lighttpd.

In response to $WORKs requirement for this I’ve used mod_magnet to run a small lua script to emulate the functionality of mod_security (to an extent at least!). Please note that mod_magent is blocking, so will cause any requests to be blocked until the mod_magnet script has completed, so be very careful with the script, and ensure that it’s not causing any lag in a test environment, prior to deploying into live!

Below is a copy of an early version of the script (most of the mod_security rules that we have are specific to work, so are not being included for various reasons), however I’ll post updates to this soon.

`/etc/lighttpd/mod_sec.lua`

```lua
-- mod_security alike in LUA for mod_magnet
LOG = true
DROP = true

function returnError(e)
        if (lighty.env["request.remote-ip"]) then
                remoteip = lighty.env["request.remote-ip"]
        else
                remoteip = "UNKNOWN_IP"
        end
        if (LOG == true) then
                print ( remoteip .. " blocked due to ".. e .. " --- " ..
                                lighty.env["request.method"] .. " " .. lighty.request["Host"] .. " " .. lighty.env["request.uri"])
        end
        if (DROP == true) then
                return 405
        end
end

function SQLInjection(content)
        if (string.find(content, "UNION")) then
                return returnError('UNION in uri')
        end
end

function UserAgent(UA)
        UA = UA:gsub("%a", string.lower, 1)
        if (string.find(UA, "libwhisker")) then
                return returnError('UserAgent - libwhisker')
        elseif (string.find(UA, "paros")) then
                return returnError('UserAgent - paros')
        elseif (string.find(UA, "wget")) then
                return returnError('UserAgent - wget')
        elseif (string.find(UA, "libwww")) then
                return returnError('UserAgent - libwww')
        elseif (string.find(UA, "perl")) then
                return returnError('UserAgent - perl')
        elseif (string.find(UA, "java")) then
                return returnError('UserAgent - java')
        end
end

-- URI = lighty.env["request.uri"]
-- POST = lighty.request
if ( SQLInjection(lighty.env["request.uri"]) == 405) then
       ret = 405
end
if ( UserAgent(lighty.request["User-Agent"]) == 405) then
       ret = 405
end
return ret
```

The following needs to be added to lighttpd.conf to attach this LUA script via mod magnet

```
server.modules += ( "mod_magnet" )
magnet.attract-physical-path-to = ( "/etc/lighttpd/mod_sec.lua")
```

* Update – 23 Aug 09 - Updated to return code even if one test passes

Comments or suggestions are appreciated!