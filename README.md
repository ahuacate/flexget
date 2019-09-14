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
- [ ] 1.00 Setting Up Flexget
- [ ] 2.00 Get your `secrets.yml` in order
- [ ] 3.00 Download the FileBot deluge-postprocess.sh script for Deluge
- [ ] 00.00 Patches & Fixes

## 1.00 Setting Up Flexget
FlexGet is a multipurpose automation tool for all of your RSS media. Support for torrents, nzbs, podcasts, comics, TV, movies, RSS, HTML, CSV, and more. We use Flexget in conjunction with Filebot for downloading, post-processing and renaming poorly named and less commonly available content such as documentaries and News media.

This recipes default RSS feed sources are: A) MVGroup; B) ShowRSS; and, C) Documentary Torrents.

Flexget is a command line based application. Flexget uses YAML for configuration. The default prebuilt YAML configuration files are constructed to seek out documentaries and factual TV shows like Panoroma, Frontline and other informative News series. It consists of a set of five files:

**FlexGet Configuration Files**
*  **config.yml**: This is where the smart stuff happens;
*  **secrets.yml**: This is where you must enter your usernames, passwords, api keys and RSS feed URLs for sites like trakt, showrrss. TheTVDb etc.

And,

**RSS Source Regexp Files**
*  **list-showrss.yml**: Regexp entries to help resolve ShowRSS alias's and naming convention issues;
*  **list-mvgroup.yml**: Regexp entries by keywords like bbc, pbs, ch5 etc for match criteria for MVGroup RSS feed; and,
*  **list-documentarytorrents.yml**: Regexp entries by keywords like bbc, pbs, ch5 etc for match criteria for DocumentaryTorrents RSS feed;

The above 5x files would've been installed if you followed the Flexget installation instructions [HERE](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#60-flexget-lxc---ubuntu-1804). To download again go to the Proxmox web interface typhoon-01 > 114 (flexget) > >_ Shell and type the following:
```
wget https://raw.githubusercontent.com/ahuacate/flexget/master/config.yml -P /home/media/flexget &&
wget https://raw.githubusercontent.com/ahuacate/flexget/master/list-showrss.yml.yml -P /home/media/flexget &&
wget https://raw.githubusercontent.com/ahuacate/flexget/master/list-mvgroup.yml -P /home/media/flexget &&
wget https://raw.githubusercontent.com/ahuacate/flexget/master/list-documentarytorrents.yml -P /home/media/flexget
```
Do not download 'secrets.yml' after entering your secrets.yml file credentials - they will be overwritten.
```
wget https://raw.githubusercontent.com/ahuacate/flexget/master/secrets.yml -P /home/media/flexget &&
```

## 2.00 Get your `secrets.yml` in order
The first step is to create user accounts at the following websites (all free).

*  Showrss - https://showrss.info
*  MVGroup - https://mvgroup.org
*  TheTVDb - https://www.thetvdb.com
*  DocumentaryTorrents - http://documentarytorrents.com

Keep your usernames, passwords and api keys ready as you will need them!

### 2.10 Setup ShowRSS
Login to your ShowRSS account [HERE](https://showrss.info). Then go to `Change Settings` and set as follows:

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

The custom RSS feed address needs to be inserted into your `secrets.yml` file the the example below:
```
### RSS Feeds
rssfeeds:
  showrss01: http://showrss.info/user/XXXXX.rss?magnets=true&namespaces=true&name=null&quality=anyhd&re=null
```
Finally, add a TV serie(s) to `My Shows`. ShowRSS is best to use for series which are problematic in Sonarr such as News or daily shows. For News or factual TV we recommend adding the following series:
*  60 Minutes (US)
*  Frontline
*  Horizon (UK)
*  Panorama

### 2.11 Setup MVGroup
Add your MVGroup credentials to the `secrets.yml` file by replacing the words **username** and **password** only as follows:
```
### RSS Feeds
rssfeeds:
  mvgroup01: https://username:password@forums.mvgroup.org/rss.php?torrentsonly=1
```

### 2.12 Setup TheTVDb
Add your TheTVDb credentials to the `secrets.yml` file.

### 2.13 Setup Documentary Torrents
Login to your Documentary Torrents account [HERE](http://documentarytorrents.com). Then click on the `Torrents Tab` and scoll to the bottom of the page and click on `Feed Info` or/ click [HERE](http://documentarytorrents.com/rss.php?custom=1). A new page will load titled `Custom RSS XML Feed`. Select the categories you want in your feed. I recommend to flag `☑`  all the categories label `HD` as follows:

| Categories | Value 
| :---  | :---
| **Categories**
|  | `☑` Adventure - HD
|  | `☑` Biography - HD
|  | `☑` Computers - HD
|  | `☑` Crime - HD
|  | `☑` Culture - HD
|  | `☑` Discovery - HD
|  | `☑` Economy - HD
|  | `☑` Food - HD
|  | `☑` HBO - HD
|  | `☑` Health - HD
|  | `☑` History - HD
|  | `☑` NatGeo - HD
|  | `☑` Nature - HD
|  | `☑` PBS - HD
|  | `☑` Politics - HD
|  | `☑` Science - HD
|  | `☑` Space - HD|  | `☑` 
|  | `☑` Technology - HD
|  | `☑` Transport - HD
|  | `☑` Travel - HD
|  | `☑` War - HD
| **Feed Type**
|  | `☑` Download link
| **Log-in Type**
|  | `☑` Alternative (no cookies)
| including dead .torrent | `☐`

Then click `Get Link` and your newly generated RSS link with passkey will show. Cut & Paste into the `secrets.yml` file as follows:
```
### RSS Feeds
rssfeeds:
  documentarytorrents01: type_here
```

### 2.13 How to edit the `secrets.yml` file
Go to the Proxmox web interface typhoon-01 > 114 (flexget) > >_ Shell and type the following:

```
nano /home/media/flexget/secrets.yml
```
Enter your credentials into the `secrets.yml` fields marked **type_here** as follows:
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
  
### TheTVDb Account ###
thetvdb:
  username: type_here
  account_id: type_here
  api_key: type_here
  
### MVGroup Account ###
mvgroup:
  url: https://forums.mvgroup.org/rss.php?torrentsonly=1
  username: type_here
  password: type_here

### RSS Feeds ###
#### Example. 'showrss01: http://showrss.info/user/your_id.rss?magnets=true&namespaces=true&name=null&quality=anyhd&re=null'
rssfeeds:
  showrss01: type_here
  mvgroup01: type_here
  documentarytorrents01: type_here
```
Note: After entering your details into the terminal, it's CTRL O (thats a capital letter O, not numerical 0) to prompt a save, ENTER to save the file, and CTRL X to exit nano.

## 3.00 Download the FileBot deluge-postprocess.sh script for Deluge
Filebot renames and moves all your Flexget downloads ready for viewing to your NAS. This action is done by running a shell script called `deluge-postprocess.sh`. Deluge uses the Execute Plugin to execute `deluge-postprocess.sh` whenever it completes a torrent download.

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
#!/bin/bash

# Input Parameters & Folder Configuration
SERIES_INPUT="/mnt/downloads/deluge/complete/flexget/series"
SERIES_OUTPUT="/mnt/video/documentary/series"
MOVIES_INPUT="/mnt/downloads/deluge/complete/flexget/movies"
MOVIES_OUTPUT="/mnt/video/documentary/movies"
UNSORTED_OUTPUT="/mnt/video/documentary/unsorted"

filebot -script fn:amc --output "$SERIES_OUTPUT" --def "ut_label=series" --action copy --conflict override -non-strict --def artwork=n --def unsorted=y --def unsortedFormat="$MOVIE_INPUT/{fn}.{ext}" --def clean=y "ut_dir=$SERIES_INPUT" "ut_kind=multi" --def "seriesFormat=/mnt/video/documentary/series/{n}/{'S'+s.pad(2)}/{n.replaceAll(/[!?.]+$/).space('.')}.{'s'+s.pad(2)}e{e.pad(2)}.{vf}.{source}.{vc}.{ac}" --def excludeList="/home/media/.filebot/series_amc.txt" -no-xattr --log-file "/home/media/.filebot/amc.log" --def reportError=y > /home/media/.filebot/series_output.txt 2>&1

filebot -script fn:amc --output "$MOVIES_OUTPUT" --db TheMovieDB --def --action copy --conflict override -non-strict --def artwork=n --def excludeList="/home/media/.filebot/movies_amc.txt" --def unsorted=y --def unsortedFormat="$UNSORTED_OUTPUT/{fn}.{ext}" --def clean=y  --def "movieFormat=/mnt/video/documentary/movies/{ny}" "ut_dir=$MOVIES_INPUT" "ut_kind=multi" "ut_label=movies" -no-xattr --log-file "/home/media/.filebot/amc.log" --def reportError=y > /home/media/.filebot/movies_output.txt 2>&1
```
Note: After pasting your key (copy & paste the license key code with your mouse buttons) into the terminal, it's `CTRL O` (thats a capital letter O, not numerical 0) to prompt a save, `Enter` to save the file and `CTRL X` to exit nano.

## 00.00 Patches and Fixes
All CLI commands performed in the `typhoon-01` > `114 (flexget)` > `>_ Shell` :

**Reset your Flexget Database**
This will completely wipe all records back to day 0.
```
flexget reset --sure
```

**Execute a test run only**
```
flexget --test execute
```

**Erase and Clean FileBot Database**
```
rm {/home/media/.filebot/amc.log,/home/media/.filebot/history.xml,/home/media/.filebot/movies_amc.txt,/home/media/.filebot/movies_output.txt,/home/media/.filebot/series_amc.txt,/home/media/.filebot/series_output.txt} &&
filebot -clear-cache
```
