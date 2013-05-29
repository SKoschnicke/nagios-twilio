notify-twilio
===

**twilio-sms** is [Perl](http://www.perl.org) command line script which is able to send SMS messages
using [Twilio](http://www.twilio.com) SMS service.

Even though this script can be used for any kind of sms messaging need, it's
primary use is [Nagios](http://www.nagios.org)/[Icinga](http://www.icinga.org)
alerting.

Requirements
==

[Perl](http://www.perl.org) interpreter and the following modules:

  * [JSON](https://metacpan.org/module/JSON)
  * [MIME::Base64](https://metacpan.org/module/MIME::Base64)
  * [URI::Escape](https://metacpan.org/module/URI::Escape)
  * [LWP/libwww-perl](https://metacpan.org/module/LWP)

Setup
==

  * install this package and put the script in $PATH
  * Create default configuration file

twilio-sms --default-config > ~/.twilio-sms.conf

  * Edit configuration file to suit your twilio account settings

```
from          = "<your twilio telephone number>"
user_id       = "<your twilio user_id>"
auth_token    = "<your twilio auth token>"
timeout       = 5
verbose       = 0
split_big_msg = 1   # you probably want to send sms messages larger than 160 chars
rewrite       = 0
```

  * Try to send a message :)

```
echo "Hello World" | twillio-sms -- +1987654321 +1987654322
```

Nagios/Icinga setup
===
  * add the following new commands to your Nagios/Icinga configuration:

```
define command {
    command_name    notify-service-by-twilio
    command_line    /usr/bin/printf "%b" "Service: $SERVICEDESC$\nHost: $HOSTALIAS$\nState: $SERVICESTATE$\n\n$SERVICEOUTPUT$" | /etc/icinga/plugins/twilio-sms -- $CONTACTPAGER$
}

define command {
        command_name    notify-host-by-twilio
    command_line    /usr/bin/printf "%b" "Host: $HOSTNAME$\nState: $HOSTSTATE$\n\n$HOSTOUTPUT$" | /etc/icinga/plugins/twilio-sms -- $CONTACTPAGER$
}
```

  * define new template and update contacts

```
# contact template
define contact {
    contact_name                    twilio-template
    name                            twilio-template
    alias                           Twilio contact template
    
    service_notification_period     24x7
    host_notification_period        24x7
    service_notification_options    w,u,c,r
    host_notification_options       d,r
    service_notification_commands   notify-service-by-twilio
    host_notification_commands      notify-host-by-twilio

    register                        0
}
    
# REAL contacts, based on twilio-template
define contact {
    use                             twilio-template
    contact_name                    joe_average
    pager                           +1234567890
}

# you probably want to use contact groups in your
# host/service definitions
define contactgroup {
    contactgroup_name       poor_bastards_on_call
    members                 joe_average
}
```

  * update service definitions to use new contactgroups

```
define service {
  name                            some_service
  # other boring stuff
  contact_groups                  poor_bastards_on_call
}
```

License
===
[The BSD 3-Clause License](http://opensource.org/licenses/BSD-3-Clause)