+++
title = "On the fly content replacement using F5 Load balancers"
#description = ""
date = "2013-07-28T20:51:48+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=36",
    "/2013/07/28/on-the-fly-content-replacement-using-f5-load-balancers/"
]
in_search_index = true


tags = [
    "Hacks",
    "Infrastructure",
    "F5",
    "iRule"
]
+++

In the modern web application world, a large proportion of sites are using SSL Offloading, be this for the added security of the web servers not having the SSL private key on them (and hence if compromised the certificate is not necessarily compromised as well) or for the performance boost associated of using hardware accelerators. This however is a double-edged sword. Its more complex for developers to test their applications against this behavior, as they need to either setup two webservers (or vhosts with proxying) on the same host to emulate this, or they need to have an actual off loading device. Both of these are not always readily available options, or easy for the development team to do.

With this in mind, I have seen many times applications that “work in development” but don’t work in production. One common issue I’ve seen is developers checking the protocol that the user has connected to the server as. When off loading, this will be HTTP, rather than HTTPS. It’s also a common practice to run SSL sites on a different port, lets say port 8080, however if the developer is using the absolute URL of the server including the port number when creating URLs this can cause issues.

The result is a url like https://www.withagrainofsalt.co.uk becomes http://web123.internal:8080/. The end user is unable to get access to this (usually) and the user experience is less than idea. The correct way to fix this would be in the application its self, however this can sometimes take weeks / months, and there may not be budget allocated to fix this defect.

With this in mind, we look to other places in the infrastructure that we would be able to fix this issue. My personal preferences are

- <span style="line-height: 14px">Apache 2.2’s [mod_substitute](http://httpd.apache.org/docs/2.2/mod/mod_substitute.html) – only in 2.2.7 and above however</span>
- The [mod_replace](http://mod-replace.sourceforge.net/) filter for Apache 2.0
- [Helicon Ape](http://www.helicontech.com/ape)‘s port of mod_replace to IIS7
- A Layer 7 Load balancer

I’ve also been informed that you can do this with

- nginx’s [HttpSubsModule](http://wiki.nginx.org/HttpSubsModule) (Thanks to [@SamJSharpe](https://twitter.com/samjsharpe) for pointing this out)

In the case of my most recent requirement, the customer was unable to edit the application, there was no budget for Ape. The customer did however have an F5 in front of their web site.

### iRule, iRock

F5 Load balancers have a TCL based scripting language that can be invoked on every request that passes through them. Due to the customer needing to do some re-writes, I got to work and crafted an iRule as below. Please be aware that you must have a STREAM profile attached to the virtual server to use this!

```tcl
when RULE_INIT {
	set static::stream_replace_debug 1
}

when HTTP_REQUEST {
   # Disable Stream profile by default
   STREAM::disable
}
when HTTP_RESPONSE {  
   # If response is mime time text/* 
   if { [HTTP::header value Content-Type] contains "text" } {
		if { $static::stream_replace_debug } { 	log local0. "[IP::client_addr]:[TCP::client_port]: MIME type matches. Enabling STREAM profile"	}
		# Match on http://something.something.something.something:81/ and replace it with ""
		STREAM::expression {@http://(\S+\.?){1,}:8081/@@}
		# Enable stream profile  
		STREAM::enable  
   }
}

when STREAM_MATCHED {  
	# Replace http:// with http:s// and Strip port number
	STREAM::replace "[string map {http:// https:// :8081 ""} [STREAM::match]]"
    if {$static::stream_replace_debug} { log local0. "[IP::client_addr]:[TCP::local_port]: matched: [STREAM::match], replaced with: [string map {http:// https:// :8081 ""} [STREAM::match]]"  }
}
```

Breaking this down we have events triggering specific blocks of code

#### RULE_INIT

This is triggered when the iRule is added to a virtual server or edited

In our example we’re performing a simple set of variable, enabling debug (set this to 0 in production!)

#### HTTP_REQUEST

This is the event triggered when an HTTP request from the client hits the load balancer.

We disable the stream profile at this point

#### HTTP_RESPONSE

This code block gets fired off when the server is replying

In our code we first check to see that the Content-Type header contains text, as this would be pointless to perform on graphics! If it is we perform the next step. If debug is enabled, we log the client IP and port to the LTM log. We set the expression that the STREAM profile will use to `http://:8081/`, and then enable the stream profile. If we did not see a Content-Type header contain text, we do nothing

#### STREAM_MATCHED

We see this code block get executed when the STREAM profile’s expression is matched.

We simply perform a string replacement swapping `http://` for `https://` and `:8081` for a blank string, and log this if debug is enabled.

### Conclusion

This does have a hit to the CPU on the load balancer, so this should be used sparingly. As mentioned above you MUST have a stream profile associated with the virtual server before applying this iRule