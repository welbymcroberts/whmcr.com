+++
title = 'zapata.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/zapata.conf"
aliases = [
    "/?page_id=87",
    "/asterisk/25/",
    "/old-content/asterisk/zaptata_conf/",
]
+++

```
[channels]
language=en

; Default context
context=inbound-from-pstn
switchtype=national
; Signalling method (default is fxs).  Valid values:
; fxs_ls:  FXS (Loop Start)
; fxs_gs:  FXS (Ground Start)
; fxs_ks:  FXS (Kewl Start)
; fxo_ls:  FXO (Loop Start)
; fxo_gs:  FXO (Ground Start)
; fxo_ks:  FXO (Kewl Start)
; fxs_rx:  Receive audio/COR on an FXS kewlstart interface (FXO at the channel bank)
; fxs_tx:  Transmit audio/PTT on an FXS loopstart interface (FXO at the channel bank)
; fxo_rx:  Receive audio/COR on an FXO loopstart interface (FXS at the channel bank)
; fxo_tx:  Transmit audio/PTT on an FXO groundstart interface (FXS at the channel bank)

signalling=fxs_ks
;
; A variety of timing parameters can be specified as well
; Including:
;    prewink:     Pre-wink time
;    preflash:    Pre-flash time
;    wink:        Wink time
;    flash:       Flash time
;    start:       Start time
;    rxwink:      Receiver wink time
;    rxflash:     Receiver flashtime
;    debounce:    Debounce timing
;
rxwink=300		; Atlas seems to use long (250ms) winks
;
usecallerid=yes
;
; Whether or not to hide outgoing caller ID
;
hidecallerid=no
;
; Whether or not to enable call waiting on FXO lines
;
callwaiting=no
;
; Whether or not restrict outgoing caller ID (will be sent as ANI only, not available for the user)
; Mostly use with FXS ports
;
;restrictcid=no
;
; Whether or not use the caller ID presentation for the outgoing call that the calling switch is sending
;
usecallingpres=yes
;
; Support Caller*ID on Call Waiting
;
callwaitingcallerid=yes
;
; Support three-way calling
;
threewaycalling=no
;
; Support flash-hook call transfer (requires three way calling)
;
transfer=no
;
; Support call forward variable
;
cancallforward=no
;
; Whether or not to support Call Return (*69/1471,3)
;
callreturn=yes

;
; Enable echo cancellation
; Use either "yes", "no", or a power of two from 32 to 256 if you wish
; to actually set the number of taps of cancellation.
;
echocancel=yes
;
; Generally, it is not necessary (and in fact undesirable) to echo cancel
; when the circuit path is entirely TDM.  You may, however, reverse this
; behavior by enabling the echo cancel during pure TDM bridging below.
;
echocancelwhenbridged=yes
;
; In some cases, the echo canceller doesn't train quickly enough and there
; is echo at the beginning of the call.  Enabling echo training will cause
; asterisk to briefly mute the channel, send an impulse, and use the impulse
; response to pre-train the echo canceller so it can start out with a much
; closer idea of the actual echo.
;
echotraining=yes
;
; You may also set the default receive and transmit gains (in dB)
;
rxgain=0.0
txgain=0.0
;
; Logical groups can be assigned to allow outgoing rollover.  Groups
; range from 0 to 31, and multiple groups can be specified.
;
group=1
;
; Ring groups (a.k.a. call groups) and pickup groups.  If a phone is ringing
; and it is a member of a group which is one of your pickup groups, then
; you can answer it by picking up and dialing *8#.  For simple offices, just
; make these both the same
;
callgroup=9
pickupgroup=9

;
; Specify whether the channel should be answered immediately or
; if the simple switch should provide dialtone, read digits, etc.
;
immediate=yes
musiconhold=default

; On trunk interfaces (FXS) and E&M interfaces (E&M, Wink, Feature Group D
; etc, it can be useful to perform busy detection either in an effort to
; detect hangup or for detecting busies
;
busydetect=no
;
; On trunk interfaces (FXS) it can be useful to attempt to follow the progress
; of a call through RINGING, BUSY, and ANSWERING.   If turned on, call
; progress attempts to determine answer, busy, and ringing on phone lines.
; This feature is HIGHLY EXPERIMENTAL and can easily detect false answers,
; so don't count on it being very accurate.  Also, it is ONLY configured for
; standard U.S. tones.  This feature can also easily detect false hangups.
; The symptoms of this is being disconnected in the middle of a call for no
; reason.
;
callprogress=no
;


channel => 1

```