+++
title = "F5 LoadBalancing on a per app mountpoint"
#description = ""
date = "2013-03-19T13:47:00+00:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=289",
    "/2013/03/19/f5-loadbalancing-on-a-per-app-mountpoint/"
    
]
in_search_index = true

tags = [
    "F5",
    "iRule",
    "Linux"
]
[extra]
+++


With some customers solutions I’ve seen a common requirement to do loadbalancing decision based on the actual application server serving the content, this obviosuly introduces a few issues if you’re using a single base URL for this

If we take the example below

```
www.example.com/ -> Web Servers
www.example.com/app1 -> App1 Servers
www.example.com/app2 -> App1 Servers
```

With this in mind, its not possible to use traditional Layer 3 / Layer 4 load balancers, and would require a L7 Load balancer, such as a F5 LTM or Riverbed Stingray (ZTM/ZXTM). I’m going to concentrate on the F5 in this example.

On the F5 you have the abbility to use a iRule to preform load balacning actions. On a Virtal Server that has the “http” profile enabled, you would be able to add a iRule similar to below.

```tcl
# Name        : Application Loadbalacning Split
# Date        : 19/03/2013
# Purpose     : Split loadbalancing based on application
# Methodology : Change pool based on url

# Set the poolname prefix based on the virtual servers name.
# Pool will always be POOL_$SERVERPOOL
when CLIENT_ACCEPTED {
    if { [virtual] contains "testing" } {
        set serverpool "testing"
    } else {
        set serverpool "liveserver"
    }
}

# Preform a load balancing decision based on the endpoint
# Split the HTTP::path out on '/' and return only the first /<something>/
# This doesn't convert from HEX / Encoded urls, but sends to default pool
when HTTP_REQUEST {
    switch [ lindex [split [string tolower [HTTP::path] ] "/" ]  1 ] {
        "app1" {
            set NEWPOOL "APP1_$serverpool"
        }
        "app2" {
            set NEWPOOL "APP2_$serverpool"
        }
        "default" {
            set NEWPOOL "default_$serverpool"
        }
    }
    pool $NEWPOOL

}

# We add a HTTP Header to identify the app pool that we're going to
when HTTP_RESPONSE {
    HTTP::header insert "X-AP" $NEWPOOL
}
```

There are multiple events that this will trigger.

1. CLIENT_ACCEPTED
 This event is triggered whenever a new connection is made to the Load balancer. In our case the code will check to see if the virtual servers name contains “testing”, which if it does sets the serverpool variable to contain “testing”, otherwise it will set it to “liveserver”

3. HTTP_REQUEST
 This event is triggered on any new HTTP Request. In our case this preforms a ‘switch’ (a multiple if/else statement) on the URL. We do however preform two “transformations” on the URL, the first is we convert it to lower case. the second is that we only take the URL between the first two /’s. So for the URL http://www.example.com/app1/test we would use app1 for teh switch statement.  
 Based on the path, we will then set the NEWPOOL variable, and then set the pool to NEWPOOL

5. HTTP_RESPONSE
 This event is triggered when the server send a response to a HTTP Request. We add the “X-AP” header to the response, and set this to the NEWPOOL variable.