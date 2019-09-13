# Flexget Build
This recipe is for setting up Flexget.

Network Prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`

Other Prerequisites are:
- [x] Synology NAS, or linux variant of a NAS, is fully configured as per [SYNOBUILD](https://github.com/ahuacate/synobuild#synobuild).
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building).
- [x] pfSense is fully configured as per [HAProxy in pfSense](https://github.com/ahuacate/proxmox-reverseproxy/blob/master/README.md#haproxy-in-pfsense)
- [x] Deluge LXC with Deluge SW installed as per [Deluge LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#40-deluge-lxc---ubuntu-1804).
- [x] Flexget LXC with Flexget SW installed as per [Flexget LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#60-flexget-lxc---ubuntu-1804)

Tasks to be performed are:
- [ ] 1.0 Setup Jellyfin and perform base configuration
- [ ] 00.00 Patches & Fixes

## 1.00 Setting Up Flexget
FlexGet is a multipurpose automation tool for all of your RSS media. Support for torrents, nzbs, podcasts, comics, TV, movies, RSS, HTML, CSV, and more. WE use Flexget in conjunction with Filebot which is used to post-process and rename all Flexget downloaded media.

Flexget is a command line based application. Flexget uses YAML for configuration. A prebuilt YAML configuration showld be 

## 2.00 Download the FileBot deluge-postprocess.sh script for Deluge
Filebot renames and moves all your Flexget downloads ready for viewing on your NAS. This action is done by running a shell script called `deluge-postprocess.sh`. Deluge uses the Execute Plugin to execute `deluge-postprocess.sh` whenever it completes a torrent download.

This script (`deluge-postprocess.sh`) is for Deluge only. It would've been installed when you completed the Deluge installation guide [HERE](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#400-deluge-lxc---ubuntu-1804).

In the event you want to upgrade or overwrite your `deluge-postprocess.sh` you can with these instructions. 

**Option (1):** Easy Way

With the Proxmox web interface go to `typhoon-01` > `113 (deluge)` > `>_ Shell` and type the following:

```
wget  https://raw.githubusercontent.com/ahuacate/deluge/master/deluge-postprocess.sh -P /home/media/.config/deluge &&
chmod +rx /home/media/.config/deluge/deluge-postprocess.sh &&
chown 1005:1005 /home/media/.config/deluge/deluge-postprocess.sh
```
**Option (2):** Nano Way
With the Proxmox web interface go to `typhoon-01` > `113 (deluge)` > `>_ Shell` and type the following:
```
nano /home/media/.config/deluge/deluge-postprocess.sh
```
Now cut & paste the following into the terminal window:
```
#!/bin/sh -xu

# Input Parameters & Folder Configuration
SERIES_INPUT="/mnt/downloads/deluge/complete/flexget/series"
SERIES_OUTPUT="/mnt/video/documentary/series"
MOVIE_INPUT="/mnt/downloads/deluge/complete/flexget/movies"
MOVIE_OUTPUT="/mnt/video/documentary/movies"
UNSORTED_OUTPUT="/mnt/video/documentary/unsorted"

filebot -script fn:amc --output "$SERIES_OUTPUT" --def "ut_label=series" --action copy --conflict override -non-strict --def artwork=n --def unsorted=y --def unsortedFormat="$MOVIE_INPUT/{fn}.{ext}" --def clean=y "ut_dir=$SERIES_INPUT" "ut_kind=multi" --def "seriesFormat=/mnt/video/documentary/series/{n}/{'S'+s.pad(2)}/{n.replaceAll(/[!?.]+$/).space('.')}.{'s'+s.pad(2)}e{e.pad(2)}.{vf}.{source}.{vc}.{ac}" --def excludeList="/home/media/.filebot/amc.txt" -no-xattr --log-file "/home/media/.filebot/amc.log" --def reportError=y > /home/media/.filebot/output.txt 2>&1


filebot -script fn:amc --output "$MOVIE_OUTPUT" --def "ut_label=movies" --action move --conflict override --def artwork=n --def clean=y "ut_dir=$MOVIE_INPUT" "ut_kind=multi" --def "movieFormat=/mnt/video/documentary/movies/{ny}/{n.upperInitial().replaceAll(/[!?.]+$/).space('.')}.{y}.{vf}.{source}.{vc}.{ac}" --def unsorted=y --def unsortedFormat="$UNSORTED_OUTPUT/{fn}.{ext}" --def excludeList="/home/media/.filebot/amc.txt" -no-xattr --log-file "/home/media/.filebot/amc.log" --def reportError=y > /home/media/.filebot/output.txt 2>&1" > /home/media/.config/deluge/deluge-postprocess.sh
```
Note: After pasting your key (copy & paste the license key code with your mouse buttons) into the terminal, it's `CTRL O` (thats a capital letter O, not numerical 0) to prompt a save, `Enter` to save the file and `CTRL X` to exit nano.

Completed.
## 1.0 Get your config.yml from GitHub
In your web browser type `http://192.168.30.113:8112/` and login with the default password. 
