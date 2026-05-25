# Zynthian Command Reference

## SSH Access

```bash
ssh root@zynthian.local          # default hostname
ssh root@192.168.x.x             # by IP if .local doesn't resolve
# password: opensynth
```

## Service Management

```bash
systemctl status zynthian --no-pager
systemctl status zynthian-webconf --no-pager
systemctl status jack2 --no-pager
systemctl status bluetooth --no-pager

systemctl restart zynthian
systemctl restart zynthian-webconf
systemctl restart jack2

journalctl -u zynthian -n 50 --no-pager
journalctl -u zynthian-webconf -n 30 --no-pager
journalctl -u jack2 -n 20 --no-pager
```

## Audio

```bash
aplay -l                          # list playback devices
arecord -l                        # list capture devices
speaker-test -D hw:S2 -c 2 -r 44100 -t sine   # test audio output
cat /proc/asound/card*/stream0    # device capabilities
vcgencmd get_throttled            # check undervoltage (0x0 = healthy)
```

## MIDI

```bash
aconnect -l                       # list MIDI ports
amidi -l                          # list raw MIDI devices
```

## Bluetooth

```bash
bluetoothctl power on
bluetoothctl scan on              # start scanning
bluetoothctl pair XX:XX:XX:XX:XX:XX
bluetoothctl trust XX:XX:XX:XX:XX:XX
bluetoothctl connect XX:XX:XX:XX:XX:XX
```

## System

```bash
dmesg | grep -i "error\|usb\|mmc" | tail -20
cat /root/first_boot.log          # first boot log
vcgencmd measure_temp             # CPU temperature
free -h                           # memory
df -h                             # disk space
```

## Configuration

```bash
cat /boot/firmware/config.txt     # Pi boot config (Pi 4/5)
cat /etc/zynthian_envars.sh       # active environment variables
ls /etc/systemd/system/zynth*     # Zynthian systemd services
```

## Documentation Regeneration

```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
git add htmldoku/ docs/
git commit -m "docs: <what changed>"
```

## Git

```bash
# zynthian-sys
git -C ~/zynth/zynthian-sys log --oneline -10
git -C ~/zynth/zynthian-sys status

# zynthian-ui
git -C ~/zynth/zynthian-ui log --oneline -10

# zynthian-webconf
git -C ~/zynth/zynthian-webconf log --oneline -10
```
