+++
title = 'QMail rrdtool graphs'
date = '2005-05-25T20:59:28+01:00'
guid = 'http://www.whmcr.com/?page_id=39'
aliases = [
    "/?p=39",
    "/2005/05/25/qmail-rrdtool-graphs/",
    "/pages/28/",
    "/old-content/qmail-rrdtool-graphs/",
    "/qmail-rrdtool-graphs/",
]
url = "/qmail-rrdtool-graphs/"
+++

This is a quick howto, thats not finished yet..

```bash
mkdir /usr/local/rrd
cd /usr/local/rrd/
```

Run the following commands to create the rrd db's
```bash

rrdtool create mess.rrd --start `date +"%s"` 
DS:mess_del:GAUGE:600:0:12500000 
DS:mess_at:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000

rrdtool create queue.rrd --start `date +"%s"` 
DS:queue_msg:GAUGE:600:0:12500000 
DS:queue_unmsg:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000

rrdtool create concurrency.rrd --start `date +"%s"` 
DS:conc_local:GAUGE:600:0:12500000 
DS:conc_remote:GAUGE:600:0:12500000 
DS:conc_smtp:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000

rrdtool create stats.rrd --start `date +"%s"` 
DS:stats_suc:GAUGE:600:0:12500000 
DS:stats_fail:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000

rrdtool create bytes.rrd --start `date +"%s"` 
DS:bytes_in:GAUGE:600:0:12500000 
DS:bytes_out:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000

rrdtool create smtp.rrd --start `date +"%s"` 
DS:smtp_allow:GAUGE:600:0:12500000 
DS:smtp_deny:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000



rrdtool create pop3conc.rrd --start `date +"%s"` 
DS:pop3conc:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000



rrdtool create pop3.rrd --start `date +"%s"` 
DS:pop3_accept:GAUGE:600:0:12500000 
DS:pop3_deny:GAUGE:600:0:12500000 
--step 300 RRA:AVERAGE:0.5:1:2000 RRA:AVERAGE:0.5:6:2000 
RRA:AVERAGE:0.5:24:2000 RRA:AVERAGE:0.5:288:2000 
RRA:MAX:0.5:1:2000 RRA:MAX:0.5:6:2000 RRA:MAX:0.5:24:2000 
RRA:MAX:0.5:288:2000
```




Add the following to update.sh
```bash
DS1="`/usr/local/bin/qmailmrtg7 m /var/log/qmail/qmail-send | head -n 1`"
DS2="`/usr/local/bin/qmailmrtg7 m /var/log/qmail/qmail-send | head -n 2 |tail -n 1`"
/usr/local/bin/rrdtool update /usr/local/rrd/mess.rrd N:$DS1:$DS2

DS1="`/usr/local/bin/qmailmrtg7 q /var/qmail/queue | head -n 1`"
DS2="`/usr/local/bin/qmailmrtg7 q /var/qmail/queue | head -n 2| tail -n 1`"
/usr/local/bin/rrdtool update /usr/local/rrd/queue.rrd N:$DS1:$DS2


DS1="`/usr/local/bin/qmailmrtg7 c /var/log/qmail/qmail-send | head -n 1`"
DS2="`/usr/local/bin/qmailmrtg7 c /var/log/qmail/qmail-send | head -n 2 |tail -n 1`"
DS3="`/usr/local/bin/qmailmrtg7 t /var/log/qmail/qmail-smtpd | head -n 1`"
/usr/local/bin/rrdtool update /usr/local/rrd/concurrency.rrd N:$DS1:$DS2:$DS3


DS1="`/usr/local/bin/qmailmrtg7 s /var/log/qmail/qmail-send | head -n 1`"
DS2="`/usr/local/bin/qmailmrtg7 s /var/log/qmail/qmail-send | head -n 2 |tail -n 1`"
/usr/local/bin/rrdtool update /usr/local/rrd/stats.rrd N:$DS1:$DS2

DS1="`/usr/local/bin/qmailmrtg7 b /var/log/qmail/qmail-send | head -n 1`"
DS2="`/usr/local/bin/qmailmrtg7 b /var/log/qmail/qmail-send | head -n 2 |tail -n 1`"
/usr/local/bin/rrdtool update /usr/local/rrd/bytes.rrd N:$DS1:$DS2

DS1="`/usr/local/bin/qmailmrtg7 a /var/log/qmail/qmail-pop3d | head -n 1`"
/usr/local/bin/rrdtool update /usr/local/rrd/pop3conc.rrd N:$DS1


DS1="`/usr/local/bin/qmailmrtg7 t /var/log/qmail/qmail-pop3d | head -n 1`"
DS2="`/usr/local/bin/qmailmrtg7 t /var/log/qmail/qmail-pop3d | head -n 2 |tail -n 1`"
/usr/local/bin/rrdtool update /usr/local/rrd/pop3.rrd N:$DS1:$DS2
```

Create the crontab
```bash
chmod +x update.sh
crontab -e

#add the following to your crontab
* * * * * /usr/local/rrd/update.sh
```


```bash
mkdir /usr/local/rrd/graph

#add the following to graph.sh

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/messages.png -a PNG -h 125 -s -129600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/mess.rrd:mess_del:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/mess.rrd:mess_at:AVERAGE' 
    'LINE1:ds1#00FF00:Message Deliveries'
    'LINE2:ds2#0000FF:Message Del Attemptsj' 
    -t "Messages"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/queue.png -a PNG -h 125 -s -129600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/queue.rrd:queue_msg:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/queue.rrd:queue_unmsg:AVERAGE' 
    'LINE1:ds1#00FF00:Queue Msgs '
    'LINE2:ds2#0000FF:Queue Unprocessed Msgs  j' 
    -t "Queue Size"

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/stats.png -a PNG -h 125 -s -129600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/stats.rrd:stats_suc:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/stats.rrd:stats_fail:AVERAGE' 
    'LINE1:ds1#00FF00:Succcessful Deliveries '
    'LINE2:ds2#0000FF:Failed Deliveries  j' 
    -t "Message Status"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/bytes.png -a PNG -h 125 -s -129600 
-v "bytes" 
    'DEF:ds1=/usr/local/rrd/bytes.rrd:bytes_in:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/bytes.rrd:bytes_out:AVERAGE' 
    'LINE1:ds1#00FF00:bytes In '
    'LINE2:ds2#0000FF:bytes out  j' 
    -t "Transfer (smtpd)"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/conc.png -a PNG -h 125 -s -129600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/concurrency.rrd:conc_local:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/concurrency.rrd:conc_remote:AVERAGE' 
        'DEF:ds3=/usr/local/rrd/concurrency.rrd:conc_smtp:AVERAGE' 
    'LINE1:ds1#00FF00:Local '
    'LINE2:ds2#0000FF:Remote  '
    'LINE3:ds3#00AAAA:SMTP j' 
    -t "Concurrency (smtpd/send)"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/smtp.png -a PNG -h 125 -s -129600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/smtp.rrd:smtp_allow:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/smtp.rrd:smtp_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted SMTP'
    'LINE2:ds2#0000FF:Denied SMTP j' 
    -t "SMTP"

    /usr/local/bin/rrdtool graph /usr/local/rrd/graph/pop3.png -a PNG -h 125 -s -129600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3.rrd:pop3_accept:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/pop3.rrd:pop3_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted pop3'
    'LINE2:ds2#0000FF:Denied pop3 j' 
    -t "pop3"

  /usr/local/bin/rrdtool graph /usr/local/rrd/graph/popconc.png -a PNG -h 125 -s -129600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3conc.rrd:pop3conc:AVERAGE' 
    'LINE1:ds1#00FF00:pop3 '
    -t "pop3 concurrency"



/usr/local/bin/rrdtool graph /usr/local/rrd/graph/messages.month.png -a PNG -h 125 -s -2592000 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/mess.rrd:mess_del:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/mess.rrd:mess_at:AVERAGE' 
    'LINE1:ds1#00FF00:Message Deliveries'
    'LINE2:ds2#0000FF:Message Del Attemptsj' 
    -t "Messages"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/queue.month.png -a PNG -h 125 -s -2592000 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/queue.rrd:queue_msg:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/queue.rrd:queue_unmsg:AVERAGE' 
    'LINE1:ds1#00FF00:Queue Msgs '
    'LINE2:ds2#0000FF:Queue Unprocessed Msgs  j' 
    -t "Queue Size"

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/stats.month.png -a PNG -h 125 -s -2592000 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/stats.rrd:stats_suc:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/stats.rrd:stats_fail:AVERAGE' 
    'LINE1:ds1#00FF00:Succcessful Deliveries '
    'LINE2:ds2#0000FF:Failed Deliveries  j' 
    -t "Message Status"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/bytes.month.png -a PNG -h 125 -s -2592000 
-v "bytes" 
    'DEF:ds1=/usr/local/rrd/bytes.rrd:bytes_in:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/bytes.rrd:bytes_out:AVERAGE' 
    'LINE1:ds1#00FF00:bytes In '
    'LINE2:ds2#0000FF:bytes out  j' 
    -t "Transfer (smtpd)"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/conc.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/concurrency.rrd:conc_local:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/concurrency.rrd:conc_remote:AVERAGE' 
        'DEF:ds3=/usr/local/rrd/concurrency.rrd:conc_smtp:AVERAGE' 
    'LINE1:ds1#00FF00:Local '
    'LINE2:ds2#0000FF:Remote  '
    'LINE3:ds3#00AAAA:SMTP j' 
    -t "Concurrency (smtpd/send)"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/smtp.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/smtp.rrd:smtp_allow:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/smtp.rrd:smtp_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted SMTP'
    'LINE2:ds2#0000FF:Denied SMTP j' 
    -t "SMTP"

    /usr/local/bin/rrdtool graph /usr/local/rrd/graph/pop3.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3.rrd:pop3_accept:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/pop3.rrd:pop3_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted pop3'
    'LINE2:ds2#0000FF:Denied pop3 j' 
    -t "pop3"

  /usr/local/bin/rrdtool graph /usr/local/rrd/graph/popconc.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3conc.rrd:pop3conc:AVERAGE' 
    'LINE1:ds1#00FF00:pop3 '
    -t "pop3 concurrency"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/messages.month.png -a PNG -h 125 -s -2592000 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/mess.rrd:mess_del:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/mess.rrd:mess_at:AVERAGE' 
    'LINE1:ds1#00FF00:Message Deliveries'
    'LINE2:ds2#0000FF:Message Del Attemptsj' 
    -t "Messages"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/queue.month.png -a PNG -h 125 -s -2592000 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/queue.rrd:queue_msg:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/queue.rrd:queue_unmsg:AVERAGE' 
    'LINE1:ds1#00FF00:Queue Msgs '
    'LINE2:ds2#0000FF:Queue Unprocessed Msgs  j' 
    -t "Queue Size"

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/stats.month.png -a PNG -h 125 -s -2592000 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/stats.rrd:stats_suc:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/stats.rrd:stats_fail:AVERAGE' 
    'LINE1:ds1#00FF00:Succcessful Deliveries '
    'LINE2:ds2#0000FF:Failed Deliveries  j' 
    -t "Message Status"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/bytes.month.png -a PNG -h 125 -s -2592000 
-v "bytes" 
    'DEF:ds1=/usr/local/rrd/bytes.rrd:bytes_in:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/bytes.rrd:bytes_out:AVERAGE' 
    'LINE1:ds1#00FF00:bytes In '
    'LINE2:ds2#0000FF:bytes out  j' 
    -t "Transfer (smtpd)"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/conc.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/concurrency.rrd:conc_local:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/concurrency.rrd:conc_remote:AVERAGE' 
        'DEF:ds3=/usr/local/rrd/concurrency.rrd:conc_smtp:AVERAGE' 
    'LINE1:ds1#00FF00:Local '
    'LINE2:ds2#0000FF:Remote  '
    'LINE3:ds3#00AAAA:SMTP j' 
    -t "Concurrency (smtpd/send)"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/smtp.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/smtp.rrd:smtp_allow:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/smtp.rrd:smtp_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted SMTP'
    'LINE2:ds2#0000FF:Denied SMTP j' 
    -t "SMTP"

    /usr/local/bin/rrdtool graph /usr/local/rrd/graph/pop3.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3.rrd:pop3_accept:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/pop3.rrd:pop3_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted pop3'
    'LINE2:ds2#0000FF:Denied pop3 j' 
    -t "pop3"

  /usr/local/bin/rrdtool graph /usr/local/rrd/graph/popconc.month.png -a PNG -h 125 -s -2592000 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3conc.rrd:pop3conc:AVERAGE' 
    'LINE1:ds1#00FF00:pop3 '
    -t "pop3 concurrency"





/usr/local/bin/rrdtool graph /usr/local/rrd/graph/messages.week.png -a PNG -h 125 -s -604800 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/mess.rrd:mess_del:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/mess.rrd:mess_at:AVERAGE' 
    'LINE1:ds1#00FF00:Message Deliveries'
    'LINE2:ds2#0000FF:Message Del Attemptsj' 
    -t "Messages"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/queue.week.png -a PNG -h 125 -s -604800 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/queue.rrd:queue_msg:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/queue.rrd:queue_unmsg:AVERAGE' 
    'LINE1:ds1#00FF00:Queue Msgs '
    'LINE2:ds2#0000FF:Queue Unprocessed Msgs  j' 
    -t "Queue Size"

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/stats.week.png -a PNG -h 125 -s -604800 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/stats.rrd:stats_suc:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/stats.rrd:stats_fail:AVERAGE' 
    'LINE1:ds1#00FF00:Succcessful Deliveries '
    'LINE2:ds2#0000FF:Failed Deliveries  j' 
    -t "Message Status"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/bytes.week.png -a PNG -h 125 -s -604800 
-v "bytes" 
    'DEF:ds1=/usr/local/rrd/bytes.rrd:bytes_in:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/bytes.rrd:bytes_out:AVERAGE' 
    'LINE1:ds1#00FF00:bytes In '
    'LINE2:ds2#0000FF:bytes out  j' 
    -t "Transfer (smtpd)"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/conc.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/concurrency.rrd:conc_local:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/concurrency.rrd:conc_remote:AVERAGE' 
        'DEF:ds3=/usr/local/rrd/concurrency.rrd:conc_smtp:AVERAGE' 
    'LINE1:ds1#00FF00:Local '
    'LINE2:ds2#0000FF:Remote  '
    'LINE3:ds3#00AAAA:SMTP j' 
    -t "Concurrency (smtpd/send)"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/smtp.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/smtp.rrd:smtp_allow:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/smtp.rrd:smtp_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted SMTP'
    'LINE2:ds2#0000FF:Denied SMTP j' 
    -t "SMTP"

    /usr/local/bin/rrdtool graph /usr/local/rrd/graph/pop3.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3.rrd:pop3_accept:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/pop3.rrd:pop3_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted pop3'
    'LINE2:ds2#0000FF:Denied pop3 j' 
    -t "pop3"

  /usr/local/bin/rrdtool graph /usr/local/rrd/graph/popconc.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3conc.rrd:pop3conc:AVERAGE' 
    'LINE1:ds1#00FF00:pop3 '
    -t "pop3 concurrency"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/messages.week.png -a PNG -h 125 -s -604800 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/mess.rrd:mess_del:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/mess.rrd:mess_at:AVERAGE' 
    'LINE1:ds1#00FF00:Message Deliveries'
    'LINE2:ds2#0000FF:Message Del Attemptsj' 
    -t "Messages"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/queue.week.png -a PNG -h 125 -s -604800 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/queue.rrd:queue_msg:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/queue.rrd:queue_unmsg:AVERAGE' 
    'LINE1:ds1#00FF00:Queue Msgs '
    'LINE2:ds2#0000FF:Queue Unprocessed Msgs  j' 
    -t "Queue Size"

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/stats.week.png -a PNG -h 125 -s -604800 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/stats.rrd:stats_suc:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/stats.rrd:stats_fail:AVERAGE' 
    'LINE1:ds1#00FF00:Succcessful Deliveries '
    'LINE2:ds2#0000FF:Failed Deliveries  j' 
    -t "Message Status"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/bytes.week.png -a PNG -h 125 -s -604800 
-v "bytes" 
    'DEF:ds1=/usr/local/rrd/bytes.rrd:bytes_in:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/bytes.rrd:bytes_out:AVERAGE' 
    'LINE1:ds1#00FF00:bytes In '
    'LINE2:ds2#0000FF:bytes out  j' 
    -t "Transfer (smtpd)"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/conc.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/concurrency.rrd:conc_local:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/concurrency.rrd:conc_remote:AVERAGE' 
        'DEF:ds3=/usr/local/rrd/concurrency.rrd:conc_smtp:AVERAGE' 
    'LINE1:ds1#00FF00:Local '
    'LINE2:ds2#0000FF:Remote  '
    'LINE3:ds3#00AAAA:SMTP j' 
    -t "Concurrency (smtpd/send)"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/smtp.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/smtp.rrd:smtp_allow:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/smtp.rrd:smtp_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted SMTP'
    'LINE2:ds2#0000FF:Denied SMTP j' 
    -t "SMTP"

    /usr/local/bin/rrdtool graph /usr/local/rrd/graph/pop3.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3.rrd:pop3_accept:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/pop3.rrd:pop3_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted pop3'
    'LINE2:ds2#0000FF:Denied pop3 j' 
    -t "pop3"

  /usr/local/bin/rrdtool graph /usr/local/rrd/graph/popconc.week.png -a PNG -h 125 -s -604800 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3conc.rrd:pop3conc:AVERAGE' 
    'LINE1:ds1#00FF00:pop3 '
    -t "pop3 concurrency"






/usr/local/bin/rrdtool graph /usr/local/rrd/graph/messages.year.png -a PNG -h 125 -s -31449600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/mess.rrd:mess_del:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/mess.rrd:mess_at:AVERAGE' 
    'LINE1:ds1#00FF00:Message Deliveries'
    'LINE2:ds2#0000FF:Message Del Attemptsj' 
    -t "Messages"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/queue.year.png -a PNG -h 125 -s -31449600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/queue.rrd:queue_msg:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/queue.rrd:queue_unmsg:AVERAGE' 
    'LINE1:ds1#00FF00:Queue Msgs '
    'LINE2:ds2#0000FF:Queue Unprocessed Msgs  j' 
    -t "Queue Size"

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/stats.year.png -a PNG -h 125 -s -31449600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/stats.rrd:stats_suc:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/stats.rrd:stats_fail:AVERAGE' 
    'LINE1:ds1#00FF00:Succcessful Deliveries '
    'LINE2:ds2#0000FF:Failed Deliveries  j' 
    -t "Message Status"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/bytes.year.png -a PNG -h 125 -s -31449600 
-v "bytes" 
    'DEF:ds1=/usr/local/rrd/bytes.rrd:bytes_in:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/bytes.rrd:bytes_out:AVERAGE' 
    'LINE1:ds1#00FF00:bytes In '
    'LINE2:ds2#0000FF:bytes out  j' 
    -t "Transfer (smtpd)"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/conc.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/concurrency.rrd:conc_local:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/concurrency.rrd:conc_remote:AVERAGE' 
        'DEF:ds3=/usr/local/rrd/concurrency.rrd:conc_smtp:AVERAGE' 
    'LINE1:ds1#00FF00:Local '
    'LINE2:ds2#0000FF:Remote  '
    'LINE3:ds3#00AAAA:SMTP j' 
    -t "Concurrency (smtpd/send)"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/smtp.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/smtp.rrd:smtp_allow:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/smtp.rrd:smtp_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted SMTP'
    'LINE2:ds2#0000FF:Denied SMTP j' 
    -t "SMTP"

    /usr/local/bin/rrdtool graph /usr/local/rrd/graph/pop3.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3.rrd:pop3_accept:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/pop3.rrd:pop3_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted pop3'
    'LINE2:ds2#0000FF:Denied pop3 j' 
    -t "pop3"

  /usr/local/bin/rrdtool graph /usr/local/rrd/graph/popconc.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3conc.rrd:pop3conc:AVERAGE' 
    'LINE1:ds1#00FF00:pop3 '
    -t "pop3 concurrency"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/messages.year.png -a PNG -h 125 -s -31449600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/mess.rrd:mess_del:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/mess.rrd:mess_at:AVERAGE' 
    'LINE1:ds1#00FF00:Message Deliveries'
    'LINE2:ds2#0000FF:Message Del Attemptsj' 
    -t "Messages"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/queue.year.png -a PNG -h 125 -s -31449600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/queue.rrd:queue_msg:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/queue.rrd:queue_unmsg:AVERAGE' 
    'LINE1:ds1#00FF00:Queue Msgs '
    'LINE2:ds2#0000FF:Queue Unprocessed Msgs  j' 
    -t "Queue Size"

/usr/local/bin/rrdtool graph /usr/local/rrd/graph/stats.year.png -a PNG -h 125 -s -31449600 
-v "Messages" 
    'DEF:ds1=/usr/local/rrd/stats.rrd:stats_suc:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/stats.rrd:stats_fail:AVERAGE' 
    'LINE1:ds1#00FF00:Succcessful Deliveries '
    'LINE2:ds2#0000FF:Failed Deliveries  j' 
    -t "Message Status"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/bytes.year.png -a PNG -h 125 -s -31449600 
-v "bytes" 
    'DEF:ds1=/usr/local/rrd/bytes.rrd:bytes_in:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/bytes.rrd:bytes_out:AVERAGE' 
    'LINE1:ds1#00FF00:bytes In '
    'LINE2:ds2#0000FF:bytes out  j' 
    -t "Transfer (smtpd)"




/usr/local/bin/rrdtool graph /usr/local/rrd/graph/conc.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/concurrency.rrd:conc_local:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/concurrency.rrd:conc_remote:AVERAGE' 
        'DEF:ds3=/usr/local/rrd/concurrency.rrd:conc_smtp:AVERAGE' 
    'LINE1:ds1#00FF00:Local '
    'LINE2:ds2#0000FF:Remote  '
    'LINE3:ds3#00AAAA:SMTP j' 
    -t "Concurrency (smtpd/send)"


/usr/local/bin/rrdtool graph /usr/local/rrd/graph/smtp.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/smtp.rrd:smtp_allow:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/smtp.rrd:smtp_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted SMTP'
    'LINE2:ds2#0000FF:Denied SMTP j' 
    -t "SMTP"

    /usr/local/bin/rrdtool graph /usr/local/rrd/graph/pop3.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3.rrd:pop3_accept:AVERAGE' 
    'DEF:ds2=/usr/local/rrd/pop3.rrd:pop3_deny:AVERAGE' 
    'LINE1:ds1#00FF00:Accepted pop3'
    'LINE2:ds2#0000FF:Denied pop3 j' 
    -t "pop3"

  /usr/local/bin/rrdtool graph /usr/local/rrd/graph/popconc.year.png -a PNG -h 125 -s -31449600 
-v "Sessions" 
    'DEF:ds1=/usr/local/rrd/pop3conc.rrd:pop3conc:AVERAGE' 
    'LINE1:ds1#00FF00:pop3 '
    -t "pop3 concurrency"
```

Set graphs to be generated
```bash
chmod +x graph.sh

#add the folowing to your crontab as above

0-59/5 * * * * /usr/local/rrd/graph.sh 1>/dev/null 2>/dev/null

#create a symbolic link to somewhere in your web root to the graph directory

ln -s /usr/local/rrd/graph /usr/local/www/data/graph
```