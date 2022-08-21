+++
title = "IRSSI Prowl Notifications"
#description = ""
date = "2009-07-09T17:39:38+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=112",
    "/2009/07/09/irssi-prowl-notifications/"
]
in_search_index = true


tags = [
    "irc", "irssi",
    "Apple",
    "iPhone",
    "Linux"
]
[extra]
+++


A quick script to send notifications from IRSSI for privmessages and also for highlights, I’ll put more commentary on later, but for now..

```perl

use strict;
use vars qw($VERSION %IRSSI);
use Irssi;
use LWP::UserAgent;

$VERSION = '0.1';

%IRSSI = (
        authors => 'Welby McRoberts',
        contact => 'irssi@whmcr.com',
        name => 'irssi_prowler',
        description => 'Sends a notification to Prowl to alert an iPhone of a new highlighted message',
        url => 'http://www.whmcr.com/2009/07/irssi-prowl-notifications',
        changes => 'Friday, 10 Jun 2009'
);

######## Config
my($PRIV_PRI, $PRIV_EVENT, $HI_PRI, $HI_EVENT, $APP, $UA, $APIKEY);
$PRIV_PRI = 2;
$PRIV_EVENT = 'Private Message';
$HI_PRI = 1;
$HI_EVENT = 'Highlight';
$APP = 'irssi';
$UA = 'irssi_prowler';
$APIKEY='7b5d817bd95911b4c049e3034dcf7a96dfa3fb53';
########

####### Highlights

sub highlight {
        my ($dest, $text, $stripped) = @_;
        if ($dest->{level} & MSGLEVEL_HILIGHT) {
                print "prowl($HI_PRI, $APP, $HI_EVENT, $text)";
                prowl($HI_PRI, $APP, $HI_EVENT, $text);
        }
}

####### Private Messages

sub priv {
        my ($server, $text, $nick, $host, $channel) = @_;
        print "prowl($PRIV_PRI, $APP, $PRIV_EVENT, $text)";
        prowl($PRIV_PRI, $APP, $PRIV_EVENT, $text);
}

####### Prowl call

sub prowl {
        my ($priority, $application, $event, $description) = @_;
        my ($request, $response, $url, $lwp);
        print 'pri: '.$priority;
        print 'app: '.$application;
        print 'event: '.$event;
        print 'description: '.$description;

        ######## Setting up the LWP
        $lwp = LWP::UserAgent->new;
        $lwp->agent($UA);
        # URL Encode
        $application =~ s/([^A-Za-z0-9])/sprintf("%%%02X", ord($1))/seg;
    $event =~ s/([^A-Za-z0-9])/sprintf("%%%02X", ord($1))/seg;
    $description =~ s/([^A-Za-z0-9])/sprintf("%%%02X", ord($1))/seg;
        # Setup the url
        $url = sprintf("https://prowl.weks.net/publicapi/add?apikey=%s&priority=%d&application=%s&event=%s&description=%s&",
                                        $APIKEY,
                                        $priority,
                                        $application,
                                        $event,
                                        $description
                                        );
        print $url;
        $request = HTTP::Request->new(GET => $url);
        $response = $lwp->request($request);
        print $response;
}

####### Bind "message private" to priv()
Irssi::signal_add_last("message private", "priv");
####### Bind "print text" to highlights()
Irssi::signal_add_last("print text", "highlight");
```