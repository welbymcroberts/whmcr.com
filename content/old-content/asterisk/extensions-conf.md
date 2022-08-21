+++
title = 'extensions.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/extensions.conf"
aliases = [
    "/?page_id=47",
    "/asterisk/12/",
    "/old-content/asterisk/extensions-conf/",
]
+++

```
[general]
static=yes
writeprotect=no


[globals]
;Using this for outbound calls, atm only one FXO card, but this will use any in group 1
OUTBOUND=Zap/1
; My extension
WELBY=SIP/6001
; person2's extension
person2=SIP/6002
; person3's extension
person3=SIP/6003
; our conumal Free World Dialup User ID
FWDUSERID=57052
;all extensions
EVERYONE=${WELBY}&${person2}&${person3}




[default]
;default context
; here just incase i've forgot to edit anything
include => mainmenu



[unrecognised]
;not used at the moment


[international]
;context for international calls
;at the current price of international
;calls on telewest, they are instantly blocked

;block when someone dials 00
exten => _00.,1,Congestion
;block when someone tries to be sneaky and 141 the number
exten => _14100.,1,Congestion


[nongeographicalnumbers]
;only certain non geogrpahical numbers are allowed
;these are the standard 0845,087* numbers

exten => _0845.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _0845.,2,Congestion
exten => _087.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _087.,2,Congestion

;same as above, but for people who are 141'ing number
exten => _141087.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _141087.,2,Congestion
exten => _1410845.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _1410845.,2,Congestion


[premium]
;this the generic premium rate number context
;i refuse to pay > 10p / min for a normal call
;so i block all 09** and
;089* numbers(no longer in use afaik)
;and of course silly 070* personal redirects
;at like 40p / min
;and of course all 118's are they are
;ludicrously inacurate and expensive....
; and we have BT's EDQ if needed

exten => _09.,1,Congestion
exten => _118.,1,Congestion
exten => _089.,1,Congestion
exten => _070.,1,Congestion
;same as above, but for people who are 141'ing number
exten => _14109.,1,Congestion
exten => _141118.,1,Congestion
exten => _141089.,1,Congestion
exten => _141070.,1,Congestion

[nationalcalls]
;this is the context for all 01 and 02 numbers
;which of course, are the only two geographical
;national number codes
exten => _01.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _01.,2,Congestion
exten => _02.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _02.,2,Congestion
;same as above, but for people who are 141'ing number
exten => _14101.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _14101.,2,Congestion
exten => _14102.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _14102.,2,Congestion


[freephone]
;we don't mind freephones so we allow all
;080*, 0500's
exten => _080.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _080.,2,Congestion
exten => _0500.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _0500.,2,Congestion
;same as above, but for people who are 141'ing number
exten => _141080.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _141080.,2,Congestion
exten => _1410500.,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _1410500.,2,Congestion


[localcalls]
;we are in edinburgh, so our local code is 0131
;so we have the other 7 numbers to play with
;this will get edited soon so that cable numbers
; are in a seperate context
; so at night no billing is done for the free
; cable to cable calls
;
; if your in an area code thats 5 digits (like 01750)
;remove one of the X's from each line


exten => _XXXXXXX,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _XXXXXXX,2,Congestion
;same as above, but for people who are 141'ing number
exten => _141XXXXXXX,1,Dial(${OUTBOUND}/${EXTEN},tT)
exten => _141XXXXXXX,2,Congestion

[mobiles]
;I personally don't like calling mobiles from the landline
;but one of the flatmates wants to be able to
;so heres the context for 07 numbers (not 070*'s though)
exten => _07.,1,Dial(${OUTBOUND}/${EXTEN})
exten => _07.,2,Congestion
;same as above, but for people who are 141'ing number
exten => _14107.,1,Dial(${OUTBOUND}/${EXTEN})
exten => _14107.,2,Congestion


[pstnservices]
;there are a few services on the phone line we still want
;1571 for instance as telewest STILL won't remove it fully
;150 to phone them to rant
;and 1471 which isnt really of anyuse now
exten => _91571,1,Dial(${OUTBOUND}/1571)
exten => _91571,2,Congestion
exten => _150,1,Dial(${OUTBOUND}/150,tT)
exten => _150,2,Congestion
exten => _91471,1,Dial(${OUTBOUND}/1471)
exten => _91471,2,Congestion

[prankcalls]
;theres a few numbers we continually prank call, so when we call them
; we are automagically 141'ing the number (removed them for obvios reasons)

exten => 1111111111,1,Dial(${OUTBOUND}/141011111111)
exten => 1111111111,2,Congestion

[longdistance]
;empty, not aplicable for telewest
[local]
;empty, not aplicable for telewest

[pstnoutbound]
;this is so i only have to include one context
;for outgoing calls on the pstn line
;includes all the relevant contexts
include => prankcalls
include => unrecognised
include => international
include => premium
include => nationalcalls
include => freephone
include => localcalls
include => mobiles
include => pstnservices
include => nongeographicalnumbers

[outbound]
;this is so i only have to include one context
;for outgoing calls includes all the relevant contexts
include => pstnoutbound
include => emergency
include => person4_iax
include => fwd-out
include => nexthopproject
include => iaxtel-out



[inbound-from-local]
exten => i,1,Playback(invalid)
exten => i,2,Hangup
include => outbound
include => internalextensions
include => voicemail
include => conference
include => sillystuff
include => onefoursevenone
include => parkedcalls


[inbound-from-pstn]
;lets bung everyone into the default context
include => default

[inbound-from-sip]
;lets bung everyone into the default context
include => default

[inbound-from-iax]
;lets bung everyone into the default context
include => default
;unless they dialed a 600x extension, in which case
exten => _600X,1,Goto(internalextensions,${EXTEN},1)
;transfer tehm to the internalextensions/extensionnumber priority 1

;anything --> mainmenu
exten => _.,1,Goto(mainmenu,s,1)
exten => _.,2,Hangup

;these are here but are redundant now
;invalid --> main menu
exten => i,1,Goto(mainmenu,s,1)


[inbound-from-parcelfarce]
;anything thats inbound from parcelfarce (my external FWD connected box)
;lets bung everyone into the default context
include => default

;invalid extension?
;ok set caller id to 7callerid
exten => i,1,SetCIDNum(7${CALLERIDNUM})
;then go to the main menu
exten => i,2,Goto(mainmenu,s,1)
;make sure they hangup
exten => i,3,Hangup

;called anything ?
;ok set caller id to 7callerid
exten => _.,1,SetCIDNum(7${CALLERIDNUM})
;then go to the main menu
exten => _.,2,Goto(mainmenu,s,1)
;make sure they hangup
exten => _.,3,Hangup


[internalextensions]
;internal extensions, call us using the macro stdexten
exten => 6001,1,Macro(stdexten,6001,${WELBY})
exten => welby,1,Goto(6001|1)
exten => 6002,1,Macro(stdexten,6002,${ROBERT})
exten => person2,1,Goto(6002|1)
exten => 6003,1,Macro(stdexten,6003,${EWAN})
exten => person3,1,Goto(6003|1)


[nexthopproject]
;some random free american call thing
exten => 1810,1,SetCIDName(${CLIDNAME})
exten => 1810,2,Dial(IAX2/password:password@iaxtel.com/17009452475@iaxtel)

[iaxtel-out]
;make calls to iaxtel via this context

exten => _17XXNXXXXXX,1,Dial(IAX2/password:password@iaxtel.com/${EXTEN}@iaxtel)
exten => _1888NXXXXXX,1,Dial(IAX2/password:password@iaxtel.com/${EXTEN}@iaxtel)
exten => _1877NXXXXXX,1,Dial(IAX2/password:password@iaxtel.com/${EXTEN}@iaxtel)
exten => _1866NXXXXXX,1,Dial(IAX2/password:password@iaxtel.com/${EXTEN}@iaxtel)
exten => _1800NXXXXXX,1,Dial(IAX2/password:password@iaxtel.com/${EXTEN}@iaxtel)

[fwd-out]
;to make calls to fwd, we prefix it with a 7
;set our callerid number to FWDUSERID
exten => _7.,1,SetCallerID(${FWDUSERID})
;call it over iax to parcelfarce
exten => _7.,2,Dial(IAX2/dr@parcelfarce/${EXTEN})
;make sure it hangs up
exten => _7.,3,Hangup

[voicemail]
;voicemail, speaks for its self really
exten => 86,1,VoicemailMain
exten => 86,2,Hangup
exten => 1571,1,VoicemailMain
exten => 1571,2,Hangup

[conference]
;our converence rooms
exten => 6101,1,meetme,1
exten => 6102,1,meetme,2
exten => 6103,1,meetme,3
exten => 6104,1,meetme,4
exten => 6105,1,meetme,5

[emergency]
;snazy 999 script, hopefully will never need to be used
;checks to see if theres an outbound zapata channel availbe
exten => 999,1,ChanIsAvail(Zap/1)
;if there is, dial 999
exten => 999,2,Dial(${AVAILCHAN}/999)
;then hangup when finished
exten => 999,3,Hangup()
;or..... just hang up the call
exten => 999,102,SoftHangup(Zap/1-1)
;wait a second
exten => 999,103,Wait(1)
;start again
exten => 999,104,Goto(1)

[person4_iax]
;used to call person4
;exten 1919 dials him over iax

exten => 1919,1,Dial(IAX2/username@ll/1919,tT)
exten => 1919,2,Hangup


[onefoursevenone]
;this script, is well, hell really
;i spent an hour or two on it, and it gives me nightmares
;its semi documented
; just incase you have no idea what 1471 is
; its like *69, except it tells you the number
; then asks you to press 3 to return the call
exten => 1471,1,Answer
exten => 1471,2,Wait(1)						;replace with you where called
exten => 1471,3,Dbget(lastcaller=CALLLOG/${CALLERIDNUM})
exten => 1471,4,Dbget(month=CALLLOG/${CALLERIDNUM}-month)
exten => 1471,5,Dbget(day=CALLLOG/${CALLERIDNUM}-day)
exten => 1471,6,Dbget(hour=CALLLOG/${CALLERIDNUM}-hour)
exten => 1471,7,Dbget(minute=CALLLOG/${CALLERIDNUM}-minute)
exten => 1471,8,Dbget(second=CALLLOG/${CALLERIDNUM}-second)
exten => 1471,9,GotoIf($[${month} = ${TIMESTAMP:4:2}]?10:13)    ;same month? goto11 for true
exten => 1471,10,GotoIf($[${day} = ${TIMESTAMP:6:2}]?11:13)    ;same day? goto12 for true
exten => 1471,11,Wait(1)					; replace with, today
exten => 1471,12,Goto(66)					; goto "at"
exten => 1471,13,Wait(1)					; replace with "on"
exten => 1471,14,GotoIf($[${day:0:1} = 0]?15:17)	; is it a day beginging 0 ? yes ?
exten => 1471,15,Playback(digits/h-${day:1})			; say h-day
exten => 1471,16,Goto(29)					; goto month processing
exten => 1471,17,GotoIf($[${day:0:1} = 2]?18:83)	; does it begin with 2x ?
exten => 1471,18,Playback(20)				; say 20
exten => 1471,19,Playback(digits/h-${day:1})			; say h-day
exten => 1471,21,Goto(29)					; goto month processing

; shut up,  i made a numbering mistake
exten => 1471,83,GotoIf($[${day:0:1} = 3]?22:24)	; does it begin with 3x ?


exten => 1471,22,Playback(30)				; say 30
exten => 1471,23,Playback(digits/h-${day:1})			; say h-day
exten => 1471,24,Goto(29)					; goto month processing
exten => 1471,25,GotoIf($[${day:0:1} = 1]26?:97)	; does it begin with 1x ?
exten => 1471,26,Playback(digits/h-${day})			; say h-day
exten => 1471,27,Goto(66)					; goto month processing
exten => 1471,29,Wait(1)					; replace with "of"
exten => 1471,30,GotoIf($[${month} = 01]?31:33)		; is it jan
exten => 1471,31,Playback(digits/mon-0)
exten => 1471,32,Goto(66)
exten => 1471,33,GotoIf($[${month} = 02]?34:36)		; is it feb
exten => 1471,34,Playback(digits/mon-1)
exten => 1471,35,Goto(66)
exten => 1471,36,GotoIf($[${month} = 03]?37:39)		; is it mar
exten => 1471,37,Playback(digits/mon-2)
exten => 1471,38,Goto(66)
exten => 1471,39,GotoIf($[${month} = 04]?40:42)		; is it apr
exten => 1471,40,Playback(digits/mon-3)
exten => 1471,41,Goto(66)
exten => 1471,42,GotoIf($[${month} = 05]?43:45)		; is it may
exten => 1471,43,Playback(digits/mon-4)
exten => 1471,44,Goto(66)
exten => 1471,45,GotoIf($[${month} = 06]?46:48)		; is it jun
exten => 1471,46,Playback(digits/mon-5)
exten => 1471,47,Goto(66)
exten => 1471,48,GotoIf($[${month} = 07]?49:51)		; is it jul
exten => 1471,49,Playback(digits/mon-6)
exten => 1471,50,Goto(66)
exten => 1471,51,GotoIf($[${month} = 08]?52:54)		; is it aug
exten => 1471,52,Playback(digits/mon-7)
exten => 1471,53,Goto(66)
exten => 1471,54,GotoIf($[${month} = 09]?55:57)		; is it sep
exten => 1471,55,Playback(digits/mon-8)
exten => 1471,56,Goto(66)
exten => 1471,57,GotoIf($[${month} = 10]?58:60)		; is it oct
exten => 1471,58,Playback(digits/mon-9)
exten => 1471,59,Goto(66)
exten => 1471,60,GotoIf($[${month} = 11]?61:63)		; is it nov
exten => 1471,61,Playback(digits/mon-10)
exten => 1471,62,Goto(66)
exten => 1471,63,GotoIf($[${month} = 12]?64:97)		; is it dec
exten => 1471,64,Playback(digits/mon-11)
exten => 1471,65,Goto(66)
exten => 1471,66,Wait(1)					; replace with "at"
exten => 1471,67,SayNumber(${hour})
exten => 1471,68,NoOp					; exten => 1471,68,Playback(vm-and)
 ; say "and" . Didnt like it saying "and"
exten => 1471,69,SayNumber(${minute})			; say minutes
exten => 1471,70,Playback(vm-and)				; say "and"
exten => 1471,71,SayNumber(${second})			; say seconds
exten => 1471,72,Wait(1)					; replace with "seconds"
exten => 1471,73,GotoIf($[${lastcaller}]?74:76)
exten => 1471,74,Wait(1)					;replace with "by caller"
exten => 1471,75,Goto(78)
exten => 1471,76,Playback(privacy-incorrect)		;sorry wrong number
;(well "I'm sorry, that number is not valid.") replace with
;"We do not have the callers number to return the call"
exten => 1471,77,Hangup
exten => 1471,78,SayDigits(${lastcaller})
exten => 1471,79,Background(pressthreetocallback)
exten => 1471,80,Timeout(7)
exten => 1471,81,NoOp
exten => 1471,82,Hangup
;83 in use above due to me fucking up

exten => 1471,97,Playback(tt-somethingwrong)		;umm you shouldnt be here
exten => 1471,98,Playback(tt-weasels)			;
exten => 1471,99,Hangup					; bye


exten => 3,1,Goto(inbound-from-local,${lastcaller},1);
exten => 3,2,Playback(vm-goodbye)
exten => i,2,Hangup
exten => t,1,Playback(vm-goodbye)
exten => t,2,Hangup



[sillystuff]
;this context is full of silly things, still usefull
exten => 500,1,Playback(demo-echotest)	; Let them know what's going on
exten => 500,2,Echo			; Do the echo test
exten => 500,3,Playback(demo-echodone)	; Let them know it's over
exten => 500,4,Goto(s,6)		; Start over
;lets play the main menu
exten => 501,1,Goto(mainmenu,s,1)
exten => 501,2,Hangup()
;123 is the uk speaking clock
;i will edit this eventually to mimic it
;at the third stroke the time sponsored by acurist will be
exten => 123,1,Answer
exten => 123,2,Wait(1)
exten => 123,3,SayUnixTime( | | k)
exten => 123,4,SayUnixTime( | | M)
exten => 123,5,Playback(vm-and)
exten => 123,6,SayUnixTime( | | S)
exten => 123,7,Hangup
;borkan ping agi
exten => 502,1,agi,tts-ping.agi
exten => 502,2,Hangup
;17070 is bt's line test facility, reads out your number
; we don't need line test, but this is hear to say what extension is calling
; pointless but nice
exten => 17070,1,Answer
exten => 17070,2,Wait(1)
exten => 17070,3,SayDigits(${CALLERIDNUM})
exten => 17070,4,Hangup

;124 is a slightly different 123
exten => 124,1,Answer
exten => 124,2,SayUnixTime()
exten => 124,3,Hangup
;random di.fm stream, can't remeber why i added it
exten => 401,1,Answer
exten => 401,2,MP3Player,http://64.236.34.97:80/stream/1003

;ah the hallowed 890

exten => 890,1,Answer()
exten => 890,2,Wait,1
exten => 890,3,mp3player(/var/lib/asterisk/music/890_hold.mp3)
exten => 890,4,Goto(890,1)

;ah ha the new 890
exten => 891,1,Answer
exten => 891,2,Wait,1
exten => 891,3,mp3player(/var/lib/asterisk/music/new890.mp3)
exten => 891,4,Goto(891,1)

;my random "what does that sound say" extension
exten => 503,1,Playback(digits/h-1)
exten => 503,2,Wait(1)
exten => 503,3,Playback(digits/mon-1)
exten => 503,4,Wait(1)
exten => 503,5,Playback(digits/oh)
exten => 503,6,Hangup


[macro-stdexten];
;
; Standard extension macro:
;   ${ARG1} - Extension  (we could have used ${MACRO_EXTEN} here as well
;   ${ARG2} - Device(s) to ring
;

;is the caller thats beeing called 6004 ?
exten => s,1,GotoIf($[${ARG1} = 6004]?30:2)
;yes go to 30, no go to 2

;save all the caller id info (for 1471) in CALLOG/EXTEN
exten => s,2,DBput(CALLLOG/${ARG1}=${CALLERIDNUM})
exten => s,3,DBput(CALLLOG/${ARG1}-month=${TIMESTAMP:4:2})
exten => s,4,DBput(CALLLOG/${ARG1}-day=${TIMESTAMP:6:2})
exten => s,5,DBput(CALLLOG/${ARG1}-hour=${TIMESTAMP:9:2})
exten => s,6,DBput(CALLLOG/${ARG1}-minute=${TIMESTAMP:-4:2})
exten => s,7,DBput(CALLLOG/${ARG1}-second=${TIMESTAMP:-2:2})
;play transfer (please hold whilst i try to conenct you)
exten => s,8,Playback(transfer)
; Ring ring, call arg2 (but put caller on a Music on hold processe)
exten => s,9,Dial(${ARG2},25,m)
; If unavailable, send to voicemail w/ unavail announce
exten => s,10,Voicemail(u${ARG1})
exten => s,11,Hangup()


exten => s,102,Voicemail(b${ARG1})				; If busy, send to voicemail w/ busy announce
exten => s,103,Hangup()

;this adds the callid stuff to EVERYONES extension
exten => s,30,DBput(CALLLOG/6001=${CALLERIDNUM})
exten => s,31,DBput(CALLLOG/6001-month=${TIMESTAMP:4:2})
exten => s,32,DBput(CALLLOG/6001-day=${TIMESTAMP:6:2})
exten => s,33,DBput(CALLLOG/6001-hour=${TIMESTAMP:9:2})
exten => s,34,DBput(CALLLOG/6001-minute=${TIMESTAMP:-4:2})
exten => s,35,DBput(CALLLOG/6001-second=${TIMESTAMP:-2:2})
exten => s,36,DBput(CALLLOG/6002=${CALLERIDNUM})
exten => s,37,DBput(CALLLOG/6002-month=${TIMESTAMP:4:2})
exten => s,38,DBput(CALLLOG/6002-day=${TIMESTAMP:6:2})
exten => s,39,DBput(CALLLOG/6002-hour=${TIMESTAMP:9:2})
exten => s,40,DBput(CALLLOG/6002-minute=${TIMESTAMP:-4:2})
exten => s,41,DBput(CALLLOG/6002-second=${TIMESTAMP:-2:2})
exten => s,42,DBput(CALLLOG/6003=${CALLERIDNUM})
exten => s,43,DBput(CALLLOG/6003-month=${TIMESTAMP:4:2})
exten => s,44,DBput(CALLLOG/6003-day=${TIMESTAMP:6:2})
exten => s,45,DBput(CALLLOG/6003-hour=${TIMESTAMP:9:2})
exten => s,46,DBput(CALLLOG/6003-minute=${TIMESTAMP:-4:2})
exten => s,47,DBput(CALLLOG/6003-second=${TIMESTAMP:-2:2})
;go to 8
exten => s,48,Goto(8)





[mainmenu]
;this is our main menu
;wait two seconds (to let the 2nd ring come through (for bellcore clid))
exten => s,1,Wait,1
exten => s,2,Wait,1
;answer it
;its for yooooooooooou
exten => s,3,Answer
;play hello.gsm
exten => s,4,Background(hello)
;play end.gsm
exten => s,5,Background(end)
;play the short 890 (20sec)
exten => s,6,Background(890)
;play hello again
exten => s,7,Background(hello)
;and end again
exten => s,8,Background(end)
;play 5 minute 890
exten => s,9,background(890long)
;call them insanse for listening to it for 5minutes
exten => s,10,Background(realend)
;hangup
exten => s,11,Hangup
;press one for welby
exten => 1,1,Macro(stdexten,6001,${WELBY})
exten => welby,1,Goto(6001|1)
;press two for person2
exten => 2,1,Macro(stdexten,6002,${ROBERT})
exten => person2,1,Goto(6002|1)
;press three for person3
exten => 3,1,Macro(stdexten,6003,${EWAN})
exten => person3,1,Goto(6003|1)
;press four to anyone
exten => 4,1,Macro(stdexten,6004,${EVERYONE})
;i'm sorry you cant type on a phone
exten => i,1,Playback(invalid)




;the bellow was taken from sprackket or somewhere, i'm not 100% sure
;thanks however

; Any category other than "General" and "Globals" represent
; extension contexts, which are collections of extensions.
;
; Extension names may be numbers, letters, or combinations
; thereof. If an extension name is prefixed by a '_'
; character, it is interpreted as a pattern rather than a
; literal.  In patterns, some characters have special meanings:
;
;   X - any digit from 0-9
;   N - any digit from 2-9
;   [1235-9] - any digit in the brackets (in this example, 1,2,3,5,6,7,8,9)
;   . - wildcard, matches anything remaining (e.g. _9011. matches anything starting with 9011 including 9011)
;
; For example the extenion _NXXXXXX would match normal 7 digit dialings, while
; _1NXXNXXXXXX would represent an area code plus phone number
; preceeded by a one.
;
; Contexts contain several lines, one for each step of each
; extension, which can take one of two forms as listed below,
; with the first form being preferred.  One may include another
; context in the current one as well, optionally with a
; date and time.  Included contexts are included in the order
; they are listed.
;


; ############################################################################
; Magic Extensions to be used within a context
; ############################################################################

; s = start
; t = This gets jumped to on timeout waiting for keypress
; i = This gets jumped to on an invalid extension
; h = This gets jumped to when a call in a given context is hung up
; fax = This gets jumped to on fax tone detection

; ############################################################################
; Dial Options
; ############################################################################

; Dial jumps to priority n+101 on busy if n+101 exists
; 't' -- Allow the user to dial # then an extension to transfer
; 'T' -- to allow the calling user to transfer the call.
; 'r' -- indicate ringing to the calling party, pass no audio until answer.
; 'm' -- provide hold music to the calling party until answered.
; 'd' -- data-quality (modem) call (minimum delay).
; 'c' -- clear-channel data call (PRI-PRI only).
; 'H' -- allow caller to hang up by hitting *.
; 'C' -- reset call detail record for this call.
; 'P[(x)]' -- privacy mode, using 'x' as database if provided.
; 'g' -- goes on in context if the destination channel hangs up

; ############################################################################
; Zaptel Acceptable Dial Strings
; ############################################################################
;
; Digits  0 - 9
; Symbols # *
; Wait    W
```