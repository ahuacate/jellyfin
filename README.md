# Jellyfin Build
This recipe is for setting up Jellyfin with HAProxy.

Network Prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`

Other Prerequisites are:
- [x] Synology NAS, or linux variant of a NAS, is fully configured as per [SYNOBUILD](https://github.com/ahuacate/synobuild#synobuild).
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building).
- [x] pfSense is fully configured as per [HAProxy in pfSense](https://github.com/ahuacate/proxmox-reverseproxy/blob/master/README.md#haproxy-in-pfsense)
- [x] Jellyfin LXC with Jellyfin SW installed as per [Jellyfin LXC - Ubuntu 16.04](https://github.com/ahuacate/proxmox-lxc/blob/master/README.md#40-jellyfin-lxc---ubuntu-1604).

Tasks to be performed are:
- [ ] Install PiHole LXC
- [ ] Install OpenVPN Gateway LXC


## 1.0 Setup Jellyfin and perform base configuration
In your web browser type `http://192.168.50.111:8096` and a Jellyfin configuration wizard should show. 

### 1.1 Jellyfin base settings
A configuration wizard will ask you to enter some basic details.

| Jellyfin Base Wizard | Value | Notes
| :---  | :---: | :---
| Preferred display language | `English (United Kingdom)` |
| **Tell us about yourself**
| Your First Name | `storm` |
| Password |  Random 16 character password | *Random 16 character password ONLY i.e cA(8&KxjLHz8s4?A*
|Password (confirm) |  Random 16 character password 
| **Setup your media libraries**
||`Skip`
| ** Preferred Metadata Language
| Language | `English`
| Country | `United Kingdom`
| **Configure Remote Access**
| Allow remote connections to this Jellyfin Server | `☑`
| Enable automatic port mapping | `☐`

And Click `Finish`. Now login with username `storm`.

## 2.0 Jellyfin Common Settings
Use the Jellyfin web interface (192.168.50.111:8096) and go to the Configuration Dashboard, by clicking on the 4 square tiles in the top right of your screen,  `Server` > `Select your Section` and set the values as follows, remembering to click `Save` at each section:

| Server | Value | Notes
| :---  | :---: | :---
| **`General`**
| Server name | `jellyfin-site1` | *`site1` refers to your the servers location i.e your home, beach house, office*
| Other settings | Leave all as default
| **`Users`**
| See below section [here]
| **`Library` > `Display`**
| Display a folder view to show plain media folders | `☐` | *Uncheck*
| Display specials within seasons they aired in | `☑` |  *Check*
| Group movies into collections | `☑` |  *Check*
| Enable external content in suggestions | `☐` | *Uncheck*
| **`Library` > `Metadata`**
| Language | `English`| *Or select your preference*
| Country | `United Kingdom` | *Or select your preference*
| **`Library` > `Nfo Settings`**
| Save user watch data to nfo's for | `None`
| Release date format | `yyyy-MM-dd`
| Save image paths within nfo files | `☑`
| Enable path substitution | `☑`
| Copy extrafanart into extrathumbs | `☐`
| **`Library` > `Advanced`**
| Date added behaviour for new content | `Use date scanned into the library`
| **`Playback` > `Streaming`** 
| Internet streaming bitrate limit (Mbps) | `10` | *Up to you what this value is. But I recommend no more than 30% of your WAN upload speed.*
| **`Transcoding`**
| Hardware acceleration | `Video Acceleration API (VA API)(experimental)` |
| VA API Device | Leave as Default | *Should be /dev/dri/renderD128*
| Enable hardware encoding | `☑`
| Transcoding thread count | `Auto`
| FFmpeg path | Leave as Default | *Should be /usr/lib/jellyfin-ffmpeg/ffmpeg*
| Transcode path | `/mnt/transcode` | *Should be /mnt/transcode*
| Audio boost when downmixing | `2`
| H264 encoding preset | `Auto`
| H264 encoding CRF | `23`
| Allow subtitle extraction on the fly | `☑`

And click `Save`.

| Devices | Value | Notes
| :---  | :---: | :---
| **`DLNA` > `Settings`**
| Enable DLNA Play To | `☐` | *Uncheck. I dont use DLNA on any devices*
| Enable DLNA debug logging | `☐` | *Uncheck*
| Enable DLNA server | `☐` | *Uncheck*
| Blast alive messages | `☐` | *Uncheck*

And click `Save`.

| Expert | Value | Notes
| :---  | :---: | :---
| **`Advanced` > `Hosting`**
| LAN networks | `192.168.50.0/24,192.168.1.0/24,192.168.2.0/28` | 
| Bind to local network address | Leave blank
| Local http port number | `8096` 
| Local https port number | `8920`
| Allow remote connections to this Jellyfin Server | `☑` | *Check*
| Remote IP address filter | Leave blank
| Remote IP address filter mode | `Whitelist`
| Public http port number | `8096`
| Public https port number | `8920`
| External domain | Leave blank
| Custom ssl certificate path | Leave blank
| Certificate password | Leave blank
| Secure connection mode | `Handled by reverse proxy`
| Enable automatic port mapping | `☐` | *Uncheck*
| External | Leave blank

And click `Save`.

## 3.0 Add media library to Jellyfin
Jellyfin should have access to your NAS folder shares via your Proxmox hosts NFS mounts. You will add the following media libraries to Jellyfin:
*  Movies
*  TV Shows
*  Music

### 3.1 Add Movies media library
Use the Jellyfin web interface and go to the Configuration Dashboard `Server` > `Library` > `Add Media Library` and set the values as follows:

| `Library` > `Add Media Library` | Value | Notes
| :---  | :---: | :---
| **Movies**
| Show advanced settings | `Enable` | 
| Content type | `Movies`
| Display name | `Movies`
| **Folders**
| LabelFolder | `/mnt/video/movies` | *Browse to the movies folder*
| (Optional) Shared network folder | `nfs://192.168.1.10/volume1/video/movies`
| **Library Settings**
| Preferred download language | `English` | *Or select your preference*
| Country | `United Kingdom` | *Or select your preference*
| Prefer embedded titles over filenames | `☐` 
| Enable real time monitoring | `☑`
| Movie metadata downloaders
| | `☑ TheMovieDb`
| | `☐ The Open Movie Database`
| Automatically refresh metadata from the internet | `Every 90 days`
| Metadata savers | `☐ Nfo`
| **Movie Image Fetchers > `Fetcher Settings`**
| Primary | `☑`
| Art | `☑`
| Banner | `☑`
| Disc | `☐`
| Logo | `☐`
| Thumb | `☑`
| Maximum number of backdrops per item | `1`
| Minimum backdrop download width | `1920`
| TheMovieDb | `☑`
| FanArt | `☑`
| The Open Movie Database | `☐`
| Screen Grabber | `☑`
| Save artwork into media folders | `☑`
| Download images in advance | `☑`
| **Chapter Images**
| Enable chapter image extraction | `☐`
| Extract chapter images during the library scan | `☐`

And click `Save`.

### 3.2 Add TV Shows media library
Use the Jellyfin web interface and go to the Configuration Dashboard `Server` > `Library` > `Add Media Library` and set the values as follows:

| `Library` > `Add Media Library` | Value | Notes
| :---  | :---: | :---
| **TV Shows**
| Show advanced settings | `Enable` | 
| Content type | `TV Shows`
| Display name | `TV Shows`
| **Folders**
| LabelFolder | `/mnt/video/tv` | *Browse to the movies folder*
| (Optional) Shared network folder | `nfs://192.168.1.10/volume1/video/tv`
| **Library Settings**
| Preferred download language | `English` | *Or select your preference*
| Country | `United Kingdom` | *Or select your preference*
| Prefer embedded titles over filenames | `☐` 
| Enable real time monitoring | `☑`
| Series metadata downloaders
| | `☑ TheTVDB`
| | `☑ TheMovieDB`
| | `☐ The Open Movie Database`
| Season metadata downloaders
| | `☐ TheMovieDB`
| Episode metadata downloaders
| | `☑ TheTVDB`
| | `☐ TheMovieDB`
| | `☐ The Open Movie Database`
| Automatically refresh metadata from the internet | `Every 90 days`
| Metadata savers | `☐ Nfo`
| **Series Image Fetchers > `Fetcher Settings`**
| Primary | `☑`
| Art | `☑`
| Banner | `☑`
| Logo | `☑`
| Thumb | `☑`
| Maximum number of backdrops per item | `1`
| Minimum backdrop download width | `1920`
| TheTVDB | `☑`
| FanArt | `☑`
| TheMovieDb | `☐`
| **Season Image Fetchers > `Fetcher Settings`**
| Primary | `☑`
| Banner | `☑`
| Thumb | `☑`
| Maximum number of backdrops per item | `0`
| Minimum backdrop download width | `1920`
| TheTVDB | `☑`
| FanArt | `☐`
| **Episode Image Fetchers**
| TheTVDB | `☑`
| TheMovieDb | `☐`
| The Open Movie Database | `☐`
| Screen Grabber | `☐`
| Save artwork into media folders | `☑`
| Download images in advance | `☑`
| Automatically merge series that are spread across multiple folders | `☐`
| Display missing episodes within seasons | `☐`
| **Chapter Images**
| Enable chapter image extraction | `☐`
| Extract chapter images during the library scan | `☐`

And click `Save`.

### 5.4 Edit Jellyfin `Storm` User
Use the Jellyfin web interface (192.168.50.111:8096) and go to the Configuration Dashboard `Server` > `Users` > `storm` and set the values as follows:

| `Server` > `Users` > `storm` > `Profile Tab` | Value | Notes
| :---  | :---: | :---
| Name | `storm`
| Allow remote connections to this Jellyfin Server | `☑ Enabled` |
| Allow this user to manage the server | `☑ Enabled`
| **Feature Access**
| Allow Live TV access | `☑ Enabled`
| Allow Live TV recording management | `☑ Enabled`
| **Media Playback**
| Allow media playback | `☑ Enabled`
| Allow audio playback that requires transcoding | `☑ Enabled`
| Allow video playback that requires transcoding | `☑ Enabled`
| Allow video playback that requires conversion without re-encoding | `☑ Enabled`
| Internet streaming bitrate limit (Mbps) | Leave blank
| **Allow Media Deletion From**
| All libraries | `☑ Enabled`
| **Remote Control**
| Allow remote control of other users | `☑ Enabled`
| Allow remote control of shared devices | `☑ Enabled`
| **Download & Sync**
| Allow media downloading and syncing | `☐`
| Allow media downloading and syncing that requires transcoding | `☐`
| Allow media conversion | `☐`
| Allow social media sharing | `☐` | *Allways disable for all users*
| Disable this user | `☐`
| Hide this user from login screens | `☑ Enabled`
| Failed login attempts before user is locked out | `0`
| **`Server` > `Users` > `storm` > `Password`**
| New Password | Random 16 character password | * You probably set this on installation. If not, ise a Random 16 character password ONLY i.e cA(8&KxjLHz8s4?A*
| New password confirm | Random 16 character password
