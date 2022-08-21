+++
title = 'modules.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/modules.conf"
aliases = [
    "/?page_id=75",
    "/asterisk/19/",
    "/old-content/asterisk/modules_conf/",
]
+++
```
;
; Asterisk configuration file
;
; Module Loader configuration file
;


[modules]

autoload=yes
load => res_musiconhold.so
load => res_parking.so
load => chan_zap.so
load => app_sayunixtime.so
noload => app_sayunixtime.so
noload => pbx_gtkconsole.so
noload => pbx_kdeconsole.so
noload => app_intercom.so
noload => chan_modem.so
noload => chan_modem_aopen.so
noload => chan_modem_bestdata.so
noload => chan_modem_i4l.so
noload => app_getcpeid.so
noload => app_milliwatt.so
noload => app_image.so
noload => chan_oss.so
noload => chan_agent.so
noload => chan_mgcp.so
noload => chan_local.so
noload => chan_phone.so
noload => chan_skinny.so
noload => switch_mysql.so
noload => chan_iax.so
noload => app_zapras.so
```