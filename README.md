# Linux Desktop Battery Notifications Based on Min/Max Threshold Levels
Concise and customizable script to send notifications to your GNU/Linux desktop if your laptop's battery level reaches a certain minimum or maximum threshold level.

In use and tested on Debian running Gnome only, but should work in other distros and desktop environments.  The script will check for its necessities and let you know what's missing, when doing an initial manual run (see instructions below.)

![Screenshot](battery-threshold.png)

**What's this script achieving?**

 I'd rather save a few pennies a year by unplugging stuff that doesn't need to be plugged in!  Likewise, my laptop manufacturer also suggests a 20/80 charge habit to keep the battery at optimum levels.

 I have experimented partial charging vs full charging and 20/80 seems to give my battery the best lifespan.

 The desktop environment I'm currently using doesn't seem to natively have the battery alert threshold capability and my laptop model doesn't have the start/stop thresholds in sysfs.

  Most existing solutions are bloated with features I don't need (or lack the capability for max/customizations).  I simply wanted a notification reminder to either plug in or remove the power when the battery charge is at my preferred levels without having to continuously look at the symbolic icons in the GUI.

## Install
Clone the repo to a directory on your PC where the script can remain accessible in userland (root/sudo is not needed)
```bash
git clone https://github.com/angela-d/battery-max.git
```

Then, do a manual run, to make sure its working (it will check to ensure your BAT path is where its expected and dependencies are installed - it'll prompt you via the terminal if something is amiss).  

Simply run the full path to `check-battery`:
```bash
/storage/battery-max/check-battery
```
(replace `/storage/battery-max` for the local path on your system, where `check-battery` will be kept)

**That's it!**  Setup a cron to automate the checks.
***
## Setup a Cron Schedule via CLI
This will trigger the script to check your battery status at the pre-defined interval you set
```bash
crontab -e
```

If you want the script to run every minute (replace `/storage/battery-max/` for the local path on your system):
```bash
* * * * * /storage/battery-max/check-battery
```

Note that if you opt to run this every minute, you may want to consider blocking cron from /var/log/syslog, else you'll see such spamming:
> Dec 28 01:50:01 debian CRON[28999]: (angela) CMD (/storage/battery-max/check-battery

*If you want to simply direct cron messages to their own log and not delete them entirely (which is what my example below will do), specify so in /etc/rsyslog.conf*

To squash this useless logging, you can run **(optional)**:
```bash
pico /etc/rsyslog.conf
```

**Find** the line:
```bash
*.*;auth,authpriv.none             -/var/log/syslog
```

**Append** `cron`, like so:
```bash
*.*;cron,auth,authpriv.none             -/var/log/syslog
```

Restart rsyslog:
```bash
service rsyslog restart
```
***
### Customizing
Open `check-battery` and you'll see a number of variables you can adjust to suit your needs:
```bash
# modify the following to suit
BAT_PATH=/sys/class/power_supply/BAT0/
AVAIL=$(cat $BAT_PATH"capacity")
MIN=70
MAX=80
NOTIFY_TITLE="Battery Level Threshold Met"
NOTIFY_MSG="Battery level is at "
ICON="battery"
FREQ=15
```

Explanation of variables:
- **BAT_PATH** = This path may be different depending on your model of laptop
- **AVAIL** = Chances are you won't need to modify this particular variable
- **MIN** = The minimum battery percentage you want to be notified for
- **MAX** = The maximum battery percentage you want to be notified for
- **NOTIFY_TITLE** = The top-most part of the alert notification
- **NOTIFY_MSG** = The message body of the alert
- **ICON** = Don't specify a file extension *or* absolute path - filename only, this can be taken from `/usr/share/icons/[current theme name]` or `~/.local/share/icons/[current theme name]`
- **FREQ** = The threshold in which a notification will repeat.  If you have a fabulous battery that maintains its charge at 80% for more than 15 minutes, you'd want to increase this to something greater - and vice-versa if it loses its charge rapidly
***
### Alternate Approaches
If you prefer to leave the battery plugged in 24/7, this script is of no use to you and [utilizing the threshold capabilities of TLP](https://linrunner.de/en/tlp/docs/tlp-linux-advanced-power-management.html) might be better suited, if you're on an IBM or Thinkpad laptop.

Additionally, you can modify `/etc/UPower/UPower.conf` and adjust the values that will send a system notification, as well (`PercentageLow` being the rate in which a critical notification is sent):
```bash
PercentageLow=10
PercentageCritical=3
PercentageAction=2
```
***
**No notifications?**

Add the following before the crons that interact with your GUI in `crontab -e` to set your display environment, which cron does not natively have access to:

```bash
DISPLAY=":0.0"
XAUTHORITY="/home/your_username/.Xauthority"
XDG_RUNTIME_DIR="/run/user/1000"
```
(all 3 the above variables may/may not be required by your distro - this formulation is what worked for me.)

I didn't add this to the script, as I have other custom utilities that send notifications to my desktop and don't feel it necessary to add this in multiple locations.
