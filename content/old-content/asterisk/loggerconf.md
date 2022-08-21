+++
title = 'logger.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/logger.conf"
aliases = [
    "/?page_id=69",
    "/asterisk/16/",
    "/old-content/asterisk/logger_conf/",
]
+++
```

[logfiles]
;
; Format is "filename" and then "levels" of debugging to be included:
;    debug
;    notice
;    warning
;    error
;
; Special filename "console" represents the system console
;

;debug => debug
console => notice,warning,error
;console => notice,warning,error,debug
messages => notice,warning,error

;syslog  wheeee lets have this print to the dotmatrix
syslog.local0 => notice,warning,error
```