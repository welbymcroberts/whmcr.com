+++
title = 'procmailrc and .qmail file'
date = '2005-05-25T20:55:17+01:00'
guid = 'http://www.whmcr.com/?page_id=39'
aliases = [
    "?p=31",
    "2005/05/25/procmailrc-and-qmail-file/",
    "pages/26/",
    "old-content/procmailrc-and-qmail-file/",
    "/procmailrc-and-qmail-file/",
]
url = "/procmailrc-and-qmail-file/"
+++
My procmail .conf and .qmail-user are below

```bash
cd ~vpopmail/domains/wheely-bin.co.uk
cat .qmail-welby
| /usr/local/bin/procmail -m -t ./welby/Maildir/procmailrc


cat ~vpopmail/domains/wheely-bin/welby/Maildir/procmailrc



VERBOSE=off
LOGABSTRACT=yes
LOGFILE=./welby/proc.log
COMSAT=no
DIR="./welby/Maildir/"
SPAM=${DIR}.SPAM/

### Spam? ok, send to the Spam Folder... sorted
:0
* ^Subject:.:SPAM:
{
LOG ="SPAM"
:0
${DIR}.SPAM/
}

:0
* ^X-Spam-Status: YES
{
LOG="SPAM-2"
:0
${DIR}.SPAM/
}

#No message id? its most likely junk, lets bin it
:0
* !^Message-Id
{
	LOG = "No ID "
	:0
	/dev/null
}

# no to header ... ummm AYE bin it
:0
* !^To:
{
	LOG = "No To: "
	:0
	/dev/null
}

# Unfortuantly i'm not brilliant at kanji, or infact any far east style language:
:0
* [Bb][Ii][Gg]5
{
        LOG = "Big5 "
        :0
        /dev/null
}

####
# I don't deal with .br, .ar or .fr, lets send them to null, or france ... wait a min!
:0
* ^(From|Received).*.(com|net).(br|ar|fr)
{
        LOG = "BR/AR/FR "
        :0
        /dev/null
}

####	forgeing IP addresses HA (except for morons using IMS,
#	a Microsoft product which breaks an otherwise valid spam-signature
#	test).
:0
* ^Received:.*((|[)(([0-9][0-9][0-9][0-9]+|[03-9][0-9][0-9]|2[6-9][0-9]|25[6-9]|0[0-9]).[0-9]+.[0-9]+.[0-9]+|
               [0-9]+.([0-9][0-9][0-9][0-9]+|[03-9][0-9][0-9]|2[6-9][0-9]|25[6-9]|0[0-9]).[0-9]+.[0-9]+|
               [0-9]+.[0-9]+.([0-9][0-9][0-9][0-9]+|[03-9][0-9][0-9]|2[6-9][0-9]|25[6-9]|0[0-9]).[0-9]+|
               [0-9]+.[0-9]+.[0-9]+.([0-9][0-9][0-9][0-9]+|[03-9][0-9][0-9]|2[6-9][0-9]|25[6-9]|0[0-9]))()|])
* !^Received:.*Internet Mail Service
{
	LOG="ip "
	:0
        ${SPAM}
}

####	More bogus IP addresses
:0
* ^Received: .*[(0)+.(0)+.(0)+.(0)+].*
{
	LOG="ip0 "
	:0
        ${SPAM}
}

## Invalid message-id format - apparantly can cause problems with people sending with arcahich
#versions of exchange, so lets spam it rather than bin it
:0
* !^Message-Id:[ ]*
{
	LOG="id "
	:0
        ${SPAM}
}

####	fscked urls, so obvisoly spam
:0 B
* http://[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]
{
	LOG="url as 10 digits "
	:0
	$HOME/spam
}

####	had lots of spam from this lot ... buh bye
:0 B
* -500^0
*  500^0	(england|india|japan|china|france|belgium|arabia).com
{
        LOG = "$country.com "
        :0
	/dev/null
}

####	$insert french national anthem
:0
* ^(From|Received).*wanadoo.fr
{
	LOG="wanado fr "
	:0
        ${SPAM}
}
# i HATE ecards... bintime!
:0
* ^Subject: .*you have an E-Card from
{
	LOG="e-card "
	:0
	/dev/null
}


# the next few rules are from someones site,  checks for forged headers from hotmail yahoo etc
# hotmail-specific
:0
* ^(From|Return-Path):.+@hotmail.com
{
	:0
	*       ^From: ".+" 
	*       ^X-OriginalArrivalTime:
	*       ^X-Originating-IP: [[0-9]+.[0-9]+.[0-9]+.[0-9]+]
 	*       ^Received: from hotmail.com (/...
 	* $     ^Message-ID: 
	{ }
	:0 Efhw
	| formail -A "X-Spammers: fake hotmail"
}

# yahoo-specific
#:0
#* ^(From|Return-Path):.+@yahoo.[a-z]+
#{
#	:0
#	*       ^Message-ID: < ([0-9.]+.qmail|[0-9]+.[0-9A-Z]+)@/[a-z0-9-]+. yahoo.[a-z.]+
#	* $     ^Received: from .+by $MATCH
#	{ }
#	:0 Efhw
#	| formail -A "X-Spammers: fake yahoo"
#}

# netscape-specific
:0
* ^(From|Return-Path):.+@netscape.
{
	:0
	*       ^X-Mailer: Atlas
	*       ^Received: from +netscape.*MAILIN
	*       ^Return-Path: </[a-z0-9_.-]+@netscape.[a-z.]+
	* $     ^From:.*$MATCH
	* $     ^Received: from $MATCH.*by [a-z0-9.-]+.aol.com
	*       ^Message-ID: <[a-z0-9]+.[a-z0-9]+.[a-z0-9]+@netscape.[a-z.]+
	{ }
	:0 Efhw
	| formail -A "X-Spammers: fake netscape"
}

#yet again, from the 'net, 419'ers

:0 B
* -500^0
*  499^2        [DM][R].[ ][A-Z]*
*  499^0        (LAGOS|NIGERIA|AFRICA)
*  150^2        [Pp][Rr][Oo][Pp][Oo][Ss][Aa][Ll]
*  150^2        [M]illion [D]ollars
*  200^2        [U]nited [S]tates
*  100^2        strictly private
*  200^2        unclaimed
*  200^2        offshore
*  100^2        funds
*  200^2        [P]rince
*  200^2        Minist(er|ry)
*  200^2        confidential
*  100^2        confidence
*  100^2        trustworthy
*   50^2        personal
*   50^2        recommend
*   50^2        invoiced
{
        LOG = "419 "
        :0
        /dev/null
}
# "419" is the section of the Nigerian penal code that covers these
# scammers.


##	Bounces...  For Bounces
:0
* ^X-Loop: You have toomany shoes
	${DIR}/

#allow everything else
:0:
*
${DIR}/


```