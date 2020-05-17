# Run a script when logging into desktop

On ubuntu 20.04 my s/pdif audio keeps turning off / becoming unavailable when resuming from sleep / unlock.
So one solution I came up with was to catch dbus messages when logging into the desktop.

In `~/.config/autostart/`, create a .desktop file which starts a bash script:

```
[Desktop Entry]
Categories=System;Audio;
Comment=Monitor dbus for unlock signals and restarts pulseaudio
Exec=/home/seven/env/linux/scripts/restart-pulseaudio-on-unlock.sh
Name=Restart pulseaudio monitor
Type=Application
```

Then a `restart-pulseaudio-on-unlock.sh` script:

```
#!/bin/bash

dbus-monitor --session "type=signal,interface=com.canonical.Unity.Session" --profile \
| while read dbusmsg; do
    if [[ "$dbusmsg" =~ Unlocked$ || "$dbusmsg" =~ NameAcquired$ ]] ; then
        sleep 5
        pulseaudio -k
        notify-send "$(basename $0)" "Restarted pulseaudio"
    fi
done
```

When logging in there's a "NameAcquired" signal when dbus monitor starts.

