+++
title = 'irssi config'
date = '2005-05-25T20:55:17+01:00'
guid = 'http://www.whmcr.com/?page_id=39'
url = "/irssi-config/"
aliases = [
    "/?p=37",
    "/2005/05/25/irssi-config/",
    "/old-content/irssi-config/",
    "/irssi-config/"
]
+++

```


servers = (
  {
    address = "irc.ipv6.freenode.net";
    chatnet = "freenode";
    port = "7000";
    autoconnect = "Yes";
  },
  {
    address = "2002:d8da:ce29:1:0:0:0:1";
    chatnet = "gamesurge";
    port = "7667";
    autoconnect = "Yes";
  },
  {
    address = "irc.blitzed.org";
    chatnet = "blitzed";
    autoconnect = "Yes";
  },
  {
    address = "2001:470:1f00:975::";
    chatnet = "smirc";
    port = "7000";
    autoconnect = "Yes";
  }
);

chatnets = {
  freenode = {
    type = "IRC";
    autosendcmd = "/^msg nickserv ident superpassword;wait -freenode 2000";
    max_kicks = "4";
    max_msgs = "1";
    max_modes = "4";
    max_whois = "1";
  };
  smirc = {
    type = "IRC";
    autosendcmd = "/^msg nickserv login welby superpassword;wait -smirc 2000";
  };
  
  blitzed = {
    type = "IRC";
    autosendcmd = "/^msg nickserv ident superpassword;wait -blitzed 2000";
  };
  gamesurge = {
    type = "IRC";
    autosendcmd = "/^msg authserv@services.gamesurge.net auth welby password;wait -gamesurge 2000";
  };
};

channels = (
  { name = "#servermatrix"; chatnet = "smirc"; autojoin = "yes"; },
  { name = "#tlug"; chatnet = "blitzed"; autojoin = "Yes"; },
  { name = "#insomnia365"; chatnet = "gamesurge"; autojoin = "Yes"; },
  { name = "#irssi"; chatnet = "freenode"; autojoin = "No"; }

);

aliases = {
  J = "join";
  WJOIN = "join -window";
  WQUERY = "query -window";
  LEAVE = "part";
  BYE = "quit";
  EXIT = "quit";
  SIGNOFF = "quit";
  DESCRIBE = "action";
  DATE = "time";
  HOST = "userhost";
  LAST = "lastlog";
  SAY = "msg *";
  WI = "whois";
  WII = "whois $0 $0";
  WW = "whowas";
  W = "who";
  N = "names";
  M = "msg";
  T = "topic";
  C = "clear";
  CL = "clear";
  K = "kick";
  KB = "kickban";
  KN = "knockout";
  BANS = "ban";
  B = "ban";
  MUB = "unban *";
  UB = "unban";
  IG = "ignore";
  UNIG = "unignore";
  SB = "scrollback";
  UMODE = "mode $N";
  WC = "window close";
  WN = "window new hide";
  SV = "say Irssi $J ($V) - http://irssi.org/";
  GOTO = "sb goto";
  CHAT = "dcc chat";
  RUN = "SCRIPT LOAD";
  UPTIME = "eval exec - expr `date +%s` - \$F | awk '{print "Irssi uptime: "int(\\\$1/3600/24)"d "int(\\\$1/3600%24)"h "int(\\\$1/60%60)"m "int(\\\$1%60)"s" }'";
  CALC = "exec - if which bc &>/dev/null\; then echo '$*' | bc | awk '{print "$*="$$1}'\; else echo bc was not found\; fi";
  SBAR = "STATUSBAR";
  INVITELIST = "mode $C +I";
  Q = "QUERY";
};

statusbar = {
  # formats:
  # when using {templates}, the template is shown only if it's argument isn't
  # empty unless no argument is given. for example {sb} is printed always,
  # but {sb $T} is printed only if $T isn't empty.

  items = {
    # start/end text in statusbars
    barstart = "{sbstart}";
    barend = "{sbend}";

    topicbarstart = "{topicsbstart}";
    topicbarend = "{topicsbend}";

    # treated "normally", you could change the time/user name to whatever
    time = "{sb $Z}";
    user = "{sb {sbnickmode $cumode}$N{sbmode $usermode}{sbaway $A}}";

    # treated specially .. window is printed with non-empty windows,
    # window_empty is printed with empty windows
    window = "{sb $winref:$itemname{sbmode $M}}";
    window_empty = "{sb $winref{sbservertag $tag}}";
    prompt = "{prompt $[.15]itemname}";
    prompt_empty = "{prompt $winname}";
    topic = " $topic";
    topic_empty = " Irssi v$J - http://irssi.org/help/";

    # all of these treated specially, they're only displayed when needed
    lag = "{sb Lag: $0-}";
    act = "{sb Act: $0-}";
    more = "-- more --";
  };

  # there's two type of statusbars. root statusbars are either at the top
  # of the screen or at the bottom of the screen. window statusbars are at
  # the top/bottom of each split window in screen.
  default = {
    # the "default statusbar" to be displayed at the bottom of the window.
    # contains all the normal items.
    window = {
      disabled = "no";

      # window, root
      type = "window";
      # top, bottom
      placement = "bottom";
      # number
      position = "1";
      # active, inactive, always
      visible = "active";

      # list of items in statusbar in the display order
      items = {
        barstart = { priority = "100"; };
        time = { };
        user = { };
        window = { };
        window_empty = { };
        lag = { priority = "-1"; };
        act = { priority = "10"; };
        more = { priority = "-1"; alignment = "right"; };
        barend = { priority = "100"; alignment = "right"; };
      };
    };

    # statusbar to use in inactive split windows
    window_inact = {
      type = "window";
      placement = "bottom";
      position = "1";
      visible = "inactive";
      items = {
        barstart = { priority = "100"; };
        window = { };
        window_empty = { };
        more = { priority = "-1"; alignment = "right"; };
        barend = { priority = "100"; alignment = "right"; };
      };
    };

    # we treat input line as yet another statusbar :) It's possible to
    # add other items before or after the input line item.
    prompt = {
      type = "root";
      placement = "bottom";
      # we want to be at the bottom always
      position = "100";
      visible = "always";
      items = {
        prompt = { priority = "-1"; };
        prompt_empty = { priority = "-1"; };
        # treated specially, this is the real input line.
        input = { priority = "10"; };
      };
    };

    # topicbar
    topic = {
      type = "root";
      placement = "top";
      position = "1";
      visible = "always";
      items = {
        topicbarstart = { priority = "100"; };
        topic = { };
        topic_empty = { };
        topicbarend = { priority = "100"; alignment = "right"; };
      };
    };
  };
};
settings = {
  core = {
    real_name = "welby";
    user_name = "welby";
    nick = "welby";
    server_reconnect_time = "1min";
    hostname = "what.ever.vhost";
  };
  "irc/core" = {
    skip_motd = "yes";
    alternate_nick = "welby~";
    ctcp_version_reply = "= 6502IRC (BBC Model B/2MHz)";
    usermode = "= +iw";
  };
  proxy = {
    irssiproxy_password = "supersecretpassword";
    irssiproxy_ports = "freenode=600 smirc=601 gamesurge=602 blitzed=604";
    irssiproxy_bind = "127.0.0.1";
  };
  "fe-common/irc" = { show_away_once = "yes"; };
};

keyboard = (
  { key = "meta-e"; id = "scroll_end"; data = ""; },
  { key = "^w"; id = "lower_window"; data = ""; },
  { key = "end"; id = "scroll_end"; data = ""; },
  { key = "home"; id = "scroll_start"; data = ""; }
);
```