+++
title = "Google Authenticator F5 IRule"
#description = ""
date = "2012-09-02T17:04:10+01:00"
#updated = ""
#slug = "a-brief-overview-of-openhab"
weight = 0
draft = false
#path = 
aliases = [
    "/?p=277",
    "/2012/09/02/google-authenticator-f5-irule/"
]
in_search_index = true

[taxonomies]
tags = [
    "Hardware",
    "Infrastructure",
    "F5",
    "iRule"
]
[extra]
+++

Two Factor authentication is rather hit and miss in terms of support from web apps.

A quick look around the web turns up an [article on DevCentral](https://devcentral.f5.com/Tutorials/TechTips/tabid/63/articleType/ArticleView/articleId/1086517/Two-Factor-Authentication-With-Google-Authenticator-And-LDAP.aspx) for a solution to implement google authentication with ldap. As I don’t run a LDAP server at home I needed to hack up the script a bit. This iRule implements the two factor side of things from the above article, but skips the LDAP side of things, as it’s not needed!

```tcl
when RULE_INIT {
  # auth parameters
  set static::auth_cookie "bigip_virtual_auth"
  set static::auth_cookie_aes_key "AES 128 abcdef0123456789abcdef0123456789"
  set static::auth_timeout 86400
  set static::auth_lifetime 86400

  # name of datagroup that holds AD user to Google Authenticator mappings
  set static::user_to_google_auth_class "user_to_google_auth"

  # lock the user out after x attempts for a period of x seconds
  set static::lockout_attempts 3
  set static::lockout_period 30

  # 0 - logging off
  # 1 - log only successes, failures, and lockouts
  # 2 - log every attempt to access virtual as well as authentication process details
  set static::debug 1

  # HTML for login page
  set static::login_page { 
  
    <div align="center">
      <div align="center" style="border:1px solid;width:300px">
        <h2>Authorization Required</h2>
        <form method="POST">
          user: <br></br>
          Google Authenticator code:
          
          
        </form>
      </div>
    </div>
  
 }
}

when CLIENT_ACCEPTED {
  # per virtual status tables for lockouts and users' auth_status
  set lockout_state_table "[virtual name]_lockout_status"
  set auth_status_table "[virtual name]_auth_status"
  set authid_to_user_table "[virtual name]_authid_to_user"

  # record client IP, [IP::client_addr] not available in AUTH_RESULT
  set user_ip [IP::client_addr]

  # set initial values for auth_id and auth_status
  set auth_id [md5 [expr rand()]]
  set auth_status 2
  set auth_req 1  
}

when HTTP_REQUEST {
	
  if { $auth_req == 1 } {
  # track original URI user requested prior to login redirect
  set orig_uri [b64encode [HTTP::uri]]

  if { [HTTP::cookie exists $static::auth_cookie] && !([HTTP::path] starts_with "/google/auth/login")} {
    set auth_id_current [AES::decrypt $static::auth_cookie_aes_key [b64decode [HTTP::cookie value $static::auth_cookie]]]
    set auth_status [table lookup -notouch -subtable $auth_status_table $auth_id_current]
    set user [table lookup -notouch -subtable $authid_to_user_table $auth_id_current]

    if { $auth_status == 0 } {
      if { $static::debug >= 2 } { log local0. "$user ($user_ip): Found valid auth cookie (auth_id=$auth_id_current), passing request through" }
    } else {
      if { $static::debug >= 2 } { log local0. "Found invalid auth cookie (auth_id=$auth_id_current), redirecting to login"}
      HTTP::redirect "/google/auth/login?orig_uri=$orig_uri"
    }
  } elseif { ([HTTP::path] starts_with "/google/auth/login") && ([HTTP::method] eq "GET") } {
    HTTP::respond 200 content $static::login_page
  } elseif { ([HTTP::path] starts_with "/google/auth/login") && ([HTTP::method] eq "POST") } {
    set orig_uri [b64decode [URI::query [HTTP::uri] "orig_uri"]] 
    HTTP::collect [HTTP::header Content-Length]
  } else {
    if { $static::debug >= 2 } { log local0. "Request for [HTTP::uri] from unauthenticated client ($user_ip), redirecting to login" }
    HTTP::redirect "/google/auth/login?orig_uri=$orig_uri"
  }
  }  
}

when HTTP_REQUEST_DATA {
  if { $auth_req == 1} {
  set user ""
  set ga_code ""

  foreach param [split [HTTP::payload] &] {
    set [lindex [split $param =] 0] [lindex [split $param =] 1]
  }
  
  if { ($user ne "") && ([string length $ga_code] == 6) } {
    set ga_code_b32 [class lookup $user $static::user_to_google_auth_class]

    set prev_attempts [table incr -notouch -subtable $lockout_state_table $user]
    table timeout -subtable $lockout_state_table $user $static::lockout_period

    if { $prev_attempts = 2 } { log local0. "$user ($user_ip): Starting authentication sequence, attempt #$prev_attempts" }

        # begin - Base32 decode to binary

        # Base32 alphabet (see RFC 4648)
        array set static::b32_alphabet {
          A 0  B 1  C 2  D 3
          E 4  F 5  G 6  H 7
          I 8  J 9  K 10 L 11
          M 12 N 13 O 14 P 15
          Q 16 R 17 S 18 T 19
          U 20 V 21 W 22 X 23
          Y 24 Z 25 2 26 3 27
          4 28 5 29 6 30 7 31
        }

        set l [string length $ga_code_b32]
        set n 0
        set j 0
        set ga_code_bin ""

        for { set i 0 } { $i < $l } { incr i } {
          set n [expr $n <= 8 } {
            set j [incr j -8]
            append ga_code_bin [format %c [expr ($n & (0xFF <> $j]]
          }
        }

        # end - Base32 decode to binary

        # begin - HMAC-SHA1 calculation of Google Auth token 
    
        set time [binary format W* [expr [clock seconds] / 30]]
  
        set ipad ""
        set opad ""
  
        for { set j 0 } { $j < [string length $ga_code_bin] } { incr j } {
          binary scan $ga_code_bin @${j}H2 k
          set o [expr 0x$k ^ 0x5C]
          set i [expr 0x$k ^ 0x36]
          append ipad [format %c $i]
          append opad [format %c $o]
        }
        while { $j < 64 } {
          append ipad 6
          append opad \
          incr j
        }
        binary scan [sha1 $opad[sha1 ${ipad}${time}]] H* token

        # end - HMAC-SHA1 calculation of Google Auth hex token 

        # begin - extract code from Google Auth hex token
        set offset [expr ([scan [string index $token end] %x] & 0x0F) <= 2 } { log local0. "$user ($user_ip): Google Authenticator TOTP token matched" }
          set auth_status 0
          set auth_id_aes [b64encode [AES::encrypt $static::auth_cookie_aes_key $auth_id]]
          table add -subtable $auth_status_table $auth_id $auth_status $static::auth_timeout $static::auth_lifetime
          table add -subtable $authid_to_user_table $auth_id $user $static::auth_timeout $static::auth_lifetime
          if { $static::debug >= 1 } { log local0. "$user ($user_ip): authentication successful (auth_id=$auth_id), redirecting to $orig_uri" }
		  HTTP::respond 302 "Location" $orig_uri "Set-Cookie" "$static::auth_cookie=$auth_id_aes;"
          HTTP::collect
        } else {
          if { $static::debug >= 1 } { log local0. "$user ($user_ip): authentication failed - Google Authenticator TOTP token not matched" }
          HTTP::respond 200 content $static::login_page
        }
      } else {
        if { $static::debug >= 1 } { log local0. "$user ($user_ip): could not find valid Google Authenticator secret for $user" }
          HTTP::respond 200 content $static::login_page
      }
    } else {
      if { $static::debug >= 1 } { log local0. "$user ($user_ip): attempting authentication too frequently, locking out for ${static::lockout_period}s" }
      HTTP::respond 200 content "You've made too many attempts too quickly. Please wait $static::lockout_period seconds and try again."
    }
  } else {
    HTTP::respond 200 content $static::login_page
  }
 }  
}
```