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
FlexGet is a multipurpose automation tool for all of your RSS media. Support for torrents, nzbs, podcasts, comics, TV, movies, RSS, HTML, CSV, and more. I use Flexget in conjunction with Filebot for post-processing and renaming all Flexget downloaded media.

Flexget is a command line based application. Flexget uses YAML for configuration. A prebuilt YAML configuration file is available which is written to seek out documentaries and factual TV shows like Panoroma, Frontline and other informative News broadcasting. It consists of 3x files:

*  **config.yml**: This is where the smart stuff happens;
*  **serial.yml**: Add your TV series naming aliases and regexp entries;
*  **secrets.yml**: This is where you must enter your usernames and passwords or api keys to sites like trakt, showrrss etc.

You should have these three files already installed if you followed the instructions when installing Flexget [HERE](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#60-flexget-lxc---ubuntu-1804). If you want you can 
download the Flexget YAML configuration files again from GitHub. Go to the Proxmox web interface typhoon-01 > 114 (flexget) > >_ Shell and type the following:
```
wget https://raw.githubusercontent.com/ahuacate/flexget/master/config.yml -P /home/media/flexget &&
wget https://raw.githubusercontent.com/ahuacate/flexget/master/secrets.yml -P /home/media/flexget &&
wget https://raw.githubusercontent.com/ahuacate/flexget/master/serial.yml -P /home/media/flexget
```

## 2.00 Get your `secrets.yml` in order
The first step is to create user accounts at the following websites (all free).

*  Showrss - https://showrss.info
*  MVGroup - https://mvgroup.org
*  tvDb - https://www.thetvdb.com

### 2.10 Setup ShowRSS
Login to your Showrss account [HERE](https://showrss.info). Then go to `Change Settings` and set as follows:

| Edit your Settings | Value
| :---  | :---:
| Use Magnets | `☑`
| Date format | Default
| Timezone | `UTC`
| Add new shows as | `Any HD quality`

Then setup up your custom RSS feed address. Click on `My Feeds` and configure as follows:

| Your personal timeline feed address | Value
| :---  | :---
| Link type | `Use magnets in feed (recommended)`
| Feed namespaces | `Include namespaces (required for catch`
| Episode name style | `Raw episode name (torrent client)`
| Quality | `Force any HD quality (override per-show setting)`
| PROPER/REPACK | `Per-show settings (recommended)`
| **The Output feed should resemble**
| Your custom RSS feed address: | `http://showrss.info/user/XXXXXX.rss?magnets=true&namespaces=true&name=null&quality=anyhd&re=null`

It is this custom RSS feed address which needs to inserted into your `secrets.yml` file like the following:
```
### RSS Feeds
rssfeeds:
  showrss01: http://showrss.info/user/XXXXX.rss?magnets=true&namespaces=true&name=null&quality=anyhd&re=null
  mvgroup01: https://username:password@forums.mvgroup.org/rss.php?torrentsonly=1
```
Last add some series to `My Shows`. For news and factual TV I recommend adding the following series:
*  60 Minutes (US)
*  Frontline
*  Horizon (UK)
*  Panorama

### 2.11 Setup MVGroup


Then enter your username, password, Api or RSS feed url into your `secrets.yml` fields as required. Go to the Proxmox web interface typhoon-01 > 114 (flexget) > >_ Shell and type the following:

```
nano /home/media/flexget/secrets.yml
```
The `secrets.yml` field codes requiring your credentials are shown as follows:
```
### Flexget Secrets File ###

### Deluge Torrent Downloader ###
### To find your deluge user and password type "cat /home/media/.config/deluge/auth" in your LXC hosting Deluge SW
### Do Not Edit
deluge:
  username: flexget
  password: 9c67cf728b8c079c2e0065ee11cb3a9a6771420a
  host: 192.168.30.113
  port: 58846

### Folder Mount Points ###
storage:
  documentary: /mnt/video/documentary
  downloads: /mnt/downloads/deluge

### Trackt Account ###
trakt:
  account: type_here
  username: type_here
  
### TVdB List Account ###
thetvdb:
  username: type_here
  account_id: type_here
  api_key: type_here
  
### MV Group Account ###
mvgroup:
  url: https://forums.mvgroup.org/rss.php?torrentsonly=1
  username: type_here
  password: type_here

### RSS Feeds ###
#### Example. 'showrss01: http://showrss.info/user/your_id.rss?magnets=true&namespaces=true&name=null&quality=anyhd&re=null'
rssfeeds:
  showrss01: type_here
  mvgroup01: type_here
  ```
Note: After entering your ddetails into the terminal, it's CTRL O (thats a capital letter O, not numerical 0) to prompt a save, ENTER to save the file and CTRL X to exit nano.

### 2.10 
## 3.00 





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
