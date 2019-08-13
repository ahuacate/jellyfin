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
- [ ] 1.0 Setup Jellyfin and perform base configuration
- [ ] 2.0 Jellyfin Common Settings
- [ ] 3.0 Add media to the Jellyfin Library
- [ ] 4.0 Edit Jellyfin `Storm` User
- [ ] 5.0 Create Jellyfin Remote Access Users
- [ ] 6.0 Create Jellyfin Media Player Users
- [ ] 00.00 Patches & Fixes

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
Use the Jellyfin web interface (192.168.50.111:8096) and go to the Configuration Dashboard, by clicking on the 4 square tiles in the top right of your screen,  `Server` > `Select your Section` and set the values as follows, remembering to click `Save` at each completed section:

| Server | Value | Notes
| :---  | :---: | :---
| **`General`**
| Server name | `jellyfin` | *`site1` refers to your the servers location i.e your home, beach house, office*
| Preferred display language | `English (United Kingdom)`
| **Paths**
| Cache path | `/var/cache/jellyfin`
| Metadata path | `/var/lib/jellyfin/metadata`
| ** Automatic Updates**
| Allow the server to restart automatically to apply updates | `☑`
| **Branding**
| Login Disclaimer | Leave Blank
| Custom css | Leave Blank
| **`Users`**
| See below section [here](https://github.com/ahuacate/jellyfin/blob/master/README.md#40-edit-jellyfin-storm-user)
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

## 3.0 Add media to the Jellyfin Library
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

### 3.3 Add Music media library
Use the Jellyfin web interface and go to the Configuration Dashboard `Server` > `Library` > `Add Media Library` and set the values as follows:

| `Library` > `Add Media Library` | Value | Notes
| :---  | :---: | :---
| **MUSIC**
| Show advanced settings | `Enable` | 
| Content type | `Music`
| Display name | `Music`
| **Folders**
| LabelFolder | `/mnt/music` | *Browse to the movies folder*
| (Optional) Shared network folder | `nfs://192.168.1.10/volume1/music`
| **Library Settings**
| Preferred download language | `English` | *Or select your preference*
| Country | `United Kingdom` | *Or select your preference*
| Enable real time monitoring | `☑`
| MusicAlbum metadata downloaders
| | `☑ MusicBrainz`
| | `☑ TheAudioDB`
| MusicArtist metadata downloaders
| | `☑ MusicBrainz`
| | `☑ TheAudioDB`
| MusicVideo metadata downloaders
| | `☐ TheMovieDb`
| Automatically refresh metadata from the internet | `Every 90 days`
| Metadata savers | `☐ Nfo`
| **MusicAlbum Image Fetchers > `Fetcher Settings`**
| Primary | `☑`
| Disc | `☐`
| FanArt | `☑`
| TheAudioDB | `☑`
| **MusicArtist Image Fetchers > `Fetcher Settings`**
| Primary | `☑`
| Art | `☑`
| Banner | `☑`
| Logo | `☑`
| Maximum number of backdrops per item | `1`
| Minimum backdrop download width | `1920`
| FanArt | `☑`
| TheAudioDB | `☑`
| **Audio Image Fetchers**
| Image Extractor | `☑`
| **MusicVideo Image Fetchers > `Fetcher Settings`**
| Primary | `☑`
| Art | `☑`
| Banner | `☑`
| Disc | `☑`
| Logo | `☑`
| Thumb | 
| Maximum number of backdrops per item | `1`
| Minimum backdrop download width | `1920`
| TheMovieDb | `☐`
| FanArt | `☑`
| Screen Grabber | `☑`
| Save artwork into media folders | `☑`
| Download images in advance | `☑`
| Automatically merge series that are spread across multiple folders | `☐`
| Display missing episodes within seasons | `☐`
| **Chapter Images**
| Enable chapter image extraction | `☐`
| Extract chapter images during the library scan | `☐`

And click `Save`.

## 4.0 Edit Jellyfin `Storm` User
Use the Jellyfin web interface and go to the Configuration Dashboard `Server` > `Users` > `storm` and set the values as follows:

| `Server` > `Users` > `storm` > `Profile Tab` | Value | Notes
| :---  | :---: | :---
| Name | `storm`
| Allow remote connections to this Jellyfin Server | `☑` |
| Allow this user to manage the server | `☑`
| **Feature Access**
| Allow Live TV access | `☑`
| Allow Live TV recording management | `☑`
| **Media Playback**
| Allow media playback | `☑`
| Allow audio playback that requires transcoding | `☑`
| Allow video playback that requires transcoding | `☑`
| Allow video playback that requires conversion without re-encoding | `☑`
| Internet streaming bitrate limit (Mbps) | Leave blank
| **Allow Media Deletion From**
| All libraries | `☑`
| **Remote Control**
| Allow remote control of other users | `☑`
| Allow remote control of shared devices | `☑`
| **Download & Sync**
| Allow media downloading and syncing | `☐`
| Allow media downloading and syncing that requires transcoding | `☐`
| Allow media conversion | `☐`
| Allow social media sharing | `☐` | *Always disable for all users.*
| Disable this user | `☐`
| Hide this user from login screens | `☑` | *Do it.*
| Failed login attempts before user is locked out | `0`
| **`Server` > `Users` > `storm` > `Password`** | | *Always click `Save` between tabs.*
| New Password | Random 16 character password | *You probably set this on installation. If not, use a Random 16 character password ONLY i.e cA(8&KxjLHz8s4?A.*
| New password confirm | Random 16 character password
| Now click `Save`
| Easy Pin Code | Leave Blank
| Enable in-network sign in with my easy pin code | `☐`

And click `Save`.

## 5.0 Create Jellyfin Remote Access Users
You can create individual users specifically for remote access for use on smartphones, tablets and notebook. These remote access users will have their media transcoded to a preset bit rate. The only issue is 4K HDR - the GPU cannot transcode 4K.

There is a Jellyfin App available for Android devices on the Google Play Store.

This recipe uses a HAProxy reverse proxy and this recipe is [HERE](https://github.com/ahuacate/proxmox-reverseproxy/blob/master/README.md#haproxy-in-pfsense).

Use the Jellyfin web interface and go to the Configuration Dashboard `Server` > `Users` > `**+**` and set the values as follows:

| `Server` > `Users` > `newuser` > `Profile Tab` | Value | Notes
| :---  | :---: | :---
| Name | `newuser` | *User first name or nickname. Hint, the Jellyfin android App capitalises the first character when typing in the username field*
| Authentication Provider | `Default`
| Allow remote connections to this Jellyfin Server | `☑` |
| Allow this user to manage the server | `☐` | *Not recommended for remote general users.*
| **Feature Access**
| Allow Live TV access | `☑`
| Allow Live TV recording management | `☑`
| **Media Playback**
| Allow media playback | `☑`
| Allow audio playback that requires transcoding | `☑`
| Allow video playback that requires transcoding | `☑`
| Allow video playback that requires conversion without re-encoding | `☑`
| Internet streaming bitrate limit (Mbps) | `5` | *Ample for smartphones. But value is dependent on your internet upload bandwidth.*
| **Allow Media Deletion From** | | *If you dont want your users wiping your content best disable this feature.*
| All libraries | `☐`
| Movies | `☐`
| Music | `☐`
| TV Shows | `☐`
| **Remote Control**
| Allow remote control of other users | `☐`
| Allow remote control of shared devices | `☐`
| **Download & Sync**
| Allow media downloading and syncing | `☐` | *Best disable otherwise users will be downloading full size Gb content.*
| Allow media downloading and syncing that requires transcoding | `☐`
| Allow media conversion | `☐`
| Allow social media sharing | `☐` | *Always disable for all users.*
| Disable this user | `☐`
| Hide this user from login screens | `☑` | *Do it*
| Failed login attempts before user is locked out | `0`
| **`Server` > `Users` > `newuser` > `Acesss`** | | *Always click `Save` between tabs.*
| **Library Access**
| Enable access to all libraries | `☑` | *By disabling you can individually select libraries from a list.*
| Enable access from all devices |  `☑` | *By disabling you can individually select by username what devices they can connect with. This only applies to devices that can be uniquely identified and will not prevent browser access. Filtering user device access will prevent them from using new devices until they've been approved here.*
| **`Server` > `Users` > `newuser` > `Password`** | | *Always click `Save` between tabs*
| New Password | Random 16 character password | * Always use a Random 16 character password ONLY i.e cA(8&KxjLHz8s4?A. Best use a password generator or management SW*
| New password confirm | Random 16 character password
| Now click `Save`
| Easy Pin Code | Leave Blank
| Enable in-network sign in with my easy pin code | `☐`

And click `Save`.

## 6.0 Create Jellyfin Media Player Users
Here we create LAN users accounts for Kodi media player devices. Transcoding will be disabled so full bandwidth media (i.e 4K HDR) will be streamed to each device.

Use the Jellyfin web interface and go to the Configuration Dashboard `Server` > `Users` > `**+**` and set the values as follows:

| `Server` > `Users` > `newuser` > `Profile Tab` | Value | Notes
| :---  | :---: | :---
| Name | `lounge-01` | *I use lounge-01, lounge-02, bedroom-01, kitchen-01, mancave-01 etc*
| Authentication Provider | `Default`
| Allow remote connections to this Jellyfin Server | `☐` |
| Allow this user to manage the server | `☐` | *Not recommended for remote general users.*
| **Feature Access**
| Allow Live TV access | `☑`
| Allow Live TV recording management | `☑`
| **Media Playback**
| Allow media playback | `☑`
| Allow audio playback that requires transcoding | `☑`
| Allow video playback that requires transcoding | `☑`
| Allow video playback that requires conversion without re-encoding | `☑`
| Internet streaming bitrate limit (Mbps) | Leave Blank | 
| **Allow Media Deletion From** | | *I only enable Movies*
| All libraries | `☐`
| Movies | `☑`
| Music | `☐`
| TV Shows | `☐`
| **Remote Control**
| Allow remote control of other users | `☐`
| Allow remote control of shared devices | `☐`
| **Download & Sync**
| Allow media downloading and syncing | `☐` | *Best disable otherwise users will be downloading full size Gb content.*
| Allow media downloading and syncing that requires transcoding | `☐`
| Allow media conversion | `☐`
| Allow social media sharing | `☐` | *Always disable for all users.*
| Disable this user | `☐`
| Hide this user from login screens | `☐` | *Disabled. Easier for remote control configuration*
| Failed login attempts before user is locked out | `0`
| **`Server` > `Users` > `newuser` > `Acesss`** | | *Always click `Save` between tabs.*
| **Library Access**
| Enable access to all libraries | `☑` | *By disabling you can individually select libraries from a list.*
| Enable access from all devices |  `☑` | *By disabling you can individually select by username what devices they can connect with. This only applies to devices that can be uniquely identified and will not prevent browser access. Filtering user device access will prevent them from using new devices until they've been approved here.*
| **`Server` > `Users` > `newuser` > `Password`** | | *Always click `Save` between tabs*
| New Password | Here I use a common pwd like a WiFi pwd | * Not so critical as this account is acessible by LAN only*
| New password confirm | Retype password
| Now click `Save`
| Easy Pin Code | Leave Blank
| Enable in-network sign in with my easy pin code | `☐`

And click `Save`.

#  00.00 Patches & Fixes
Tweaks and fixes to make broken things work - sometimes.

## 00.01 VAAPI Error
FFMPEG errors started for no known reason. Android Jellyfin client APP reported **Playback Error - No Compatible Streams**.
Fix was reinstalling VAINFO on the Proxmox Host.

```
apt remove vainfo -y &&
apt autoremove vainfo -y &&
apt install -y
```
