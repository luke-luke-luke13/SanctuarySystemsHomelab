# Sanctuary Systems Homelab

## Introduction

This project began as a means to learn more about self-hosting services. I have operated a "homelab" for many years now, but it has always been contained to one device running the entire workload. Over time, I have accumulated more hardware to distribute key functions and have gone through the trial and error of getting it all working cohesively. It has been an *expensive, time-consuming,* and *painful process*. However, I believe most passion projects are like this, and being able to power through the unknown to reach the stated goal is worth all of the trouble in the end. 

*Ok, but if I wasn't passionate about this project, what other reasons are there to do it?* Well, for one, computers **are** the future. Having even a 30,000 foot understanding of how these systems work can give you an advantage that few people have. Understanding networking, both inter and intra, allows you to better solve problems in an increasingly online environment. Dipping your toes into *NIX and learning how to set up a computer to do **exactly** as you wish unlocks a vast array of potential solutions, automations, and general prowess over your world. 

*Ok, but what if I don't care about none of that? Why else is such a project necessary?* As you have no doubt experienced in one way or the other, corporate America is becoming increasingly more anti-consumer. Everything exists as a subscription. Software-as-a-service (SAAS) has allowed companies to charge more and more for products while simultaneously stripping features and functionality. Media conglomerates fight tooth and nail for streaming rights, only to move their content to another god-forsaken streaming service that starts at $10+/month with no guarantee that the specific content you paid for will still be available in 6 months (See meme below). On top of that, even purchasing a digital copy of something usually does not give you access to the raw file. You are stuck relying on a company to honor that purchase, host the file, and provide it to you when you want, which they most certainly won't when push comes to shove. Most of the software and services you rely on can be self-hosted on your own hardware, removing the need for monthly subscriptions, reducing your bandwidth consumption, and providing you with better autonomy over the things that matter to you. In the case of digital media like movies, music, and shows, you can *obtain** the raw files in some fashion and store them on your device, forever guaranteeing access without incurring additional costs.

**How you obtain the media is entirely up to you. It can be as legal as purchasing and ripping dvds, or as illegal as straight piracy. For brevity's sake, I will not elaborate on my views towards piracy. (take a wild guess!)*

<img src=".\images\onPoob.jpeg" alt="It's Literally on Poob" style="width:400px;" />

## Lab Overview

*Ok, so what does the lab actually do?* My lab is mainly focused around acquiring and streaming media for my own private use. I do NOT pay for any streaming services (except spotify, cause I haven't figured out a good alternative to it yet). Instead, the lab sources my movies/tv shows and automatically downloads them to my storage server, re-names/organizes them, converts/transcodes them into my preferred formats, and then makes them available for me to stream anywhere in the world directly from my own hardware. There are few other tasks that the lab handles, but the current main use case is media.  

## Hardware Overview

### Networking
The networking for this project begins at the router. I am using an EdgeRouter X from Ubiquiti. *I haven't configured this yet because I didn't want to deal with Double NAT, so check back later*.

I am also using a GigaPlus 8x2.5G PoE+ with 2x10G SFP switch for aggregation. I had mixed feelings about buying this switch, as it isn't a typical mainstream brand, but many people online have had good success with this particular model. Time will tell, but so far I am ok with it. Having 8 x 2.5G with PoE+ for less than $100 is a damn good deal.  

<img src=".\images\edgeRouterX.png" alt="Edge Router X" style="width:300px;" /> <img src=".\images\gigaPlusSwitch.jpg" alt="GigaPlus Switch" style="width:300px;" />

### Compute

90% of the project compute comes from a Proxmox host running on a MinisForum MS-01. Hardware specs are as follows:
* Processor: Intel Core i9-13900H (14 Core/20 Thread, Up to 5.4GHz, 24MB Cache)
* Graphics: Intel Iris Xe Graphics (Supports Intel QuickSync Video)
* Memory: 32GB DDR5 
* Storage: 1TB NVME SSD
* Networking: 2x10G SFP+, 2x2.5G RJ45

<center><img src=".\images\minisForumMS01.jpg" alt="MinisForum MS-01" style="width:300px;" /></center>

### Auxilliary Compute

A very small amount of compute resources are available via a Raspberry Pi 5 8GB model. 
The pi is equipped with a PoE+ and M.2 SSD hat, with a 256GB SSD. 

<center> <img src=".\images\rpi.jpg" alt="RPi with PoE SSD Hat" style="width:300px;" /> </center>

### Network Attached Storage

NAS functions are handled by a Synology DS1522+. Drive configuration is as follows:
* Drives 0 & 1: Seagate Ironwolf 8TB
* Drives 2 & 3: Seagate Ironwolf 4TB
* Drive 4: Samsung 850 Pro 256GB SSD Cache

The drives are configured in a Synology Hybrid Raid, which results in approx. 14.5TB usable and 7.3TB of protection.

<center> <img src=".\images\ds1522.jpg" alt="Synology DS1522+ NAS" style="width:300px;" /> </center>

### Power Management
An Eaton Tripp-Lite BC600RNC UPS provides uninterruptable power for the entire rack. The UPS is rated for 600VA and features 4 x 120V outlets as well as network connectivity and remote management. 
<center> <img src=".\images\eatonUPS.jpg" alt="Eaton BC600RNC UPS" style="width:300px;" /> </center>

## Software Overview

### Proxmox Node Theseus
I am using proxmox as my main virtualization platform. As stated previously, I've run a homelab in some capacity for many years now. I have chosen to name the proxmox node Theseus, as it is the same lab with all of its constituent parts having been replaced. The proxmox host has remote folder mounts to the synology NAS through a seperate network interface on both devices (I have them direct wired since I only have 2 x 10G capable devices) I currently have 5 containers active. Their configurations are as follows:

#### Ironsides LXC ``` 4 CPU Cores, 10GB RAM, 280GB Disk ```
Ironsides handles the bulk of my media acquisition and management. This container runs Docker, and has the following containers:

* Prowlarr (Indexer manager)
* Radarr (Movie manager)
* Sonarr (TV Show manager)
* Overseerr (Media Request Manager)
* Portainer (Container manager)
* Unpackarr (Auto file unzipper/extractor)
* qBittorrent (torrent client)
* qBittorrent Port Forwarder (auto updates port from VPN)
* Gluetun (VPN using PIA)
* Tautilli (Plex Media Server User Statistics)
* Prometheus Exporters for each of the above containers (Metrics and monitoring)

#### Tdarr LXC ``` 4 CPU Cores, 4GB RAM, 20GB Disk ```
I am running a Tdarr transcoding container to access the GPU for QuickSync support, transcoding all media on my server into the H.265 codec for reduced filesize and easier file streaming. 


#### Netman LXC ``` 2 CPU Cores, 1GB RAM, 4GB Disk ``` 
Netman handles all network related tasks, including nginx reverse proxy and cloudflare dynamic dns. nginx grants me access to services from outside my local network, and cloudflare ddns auto updates my domain name to match my current IP.


#### Starplex LXC ``` 4 CPU Cores, 4GB RAM, 32GB Disk ```
This is just a run of the mill plex container. Plex handles all of my media playback and streaming, both local and remote. 


#### Prometh LXC ``` 2 CPU Cores, 2GB RAM, 12GB Disk ```
Prometheus container for logging container metrics and data. Also runs Grafana for visualizing the data.


#### Scabby LXC (currently deactivated)
Scabby is manily for webscraping and API access. It runs influxDB and a couple of python scripts for fetching various API endpoints and storing the data. Most notably, it collects daily prices of precious metals, base metals, and other commodities for my personal use. 

---

### RPi 5
This raspberry pi was mainly added as another learning tool. Current plans for the pi include:

#### PiHole DNS
PiHole is a DNS sinkhole that protects devices from unwanted content without needing client-side software installed on each network device. PiHole blocks ads, trackers, and other unnecessary data-scraping tools from reaching their targets as well as preventing devices on the local network from unnecessarily "phoning home" through any manufacturer or developer installed services. 

#### NUT (network UPS tools)
NUT allows a server to monitor the status of a UPS and report that back to all other servers that are configured to listen. When battery percentage drops to a certain point, NUT can automatically issue safe shutdown commands to every server to ensure that things stay safe even during complete power and battery loss. 

#### copyparty fileserver (new addition)
I just saw a promotional video for copyparty, which is an insanely feature rich and powerful bit of open-source fileserver software. I am excited to try this on the Pi to take advantage of the SSD. Need to experiment with it before I pass judgement on it, but so far I really like what I see. copyparty features resumable uploads/downloads, uses python 2 or 3, supports http/https, webdav, ftp, tftp, smp/cifs

## Rack Overview 
The entire system is contained in a 10 inch, 12U DeskPi Rack. I chose this rack for its form-factor (the 10in rack is absolutely killer for small scale deployments) and aesthetics. I'm a sucker for brushed aluminum and the tinted tempered glass side panels really tie the whole thing together. I also wanted something that was completely self-contained, requiring only 2 external connections: 120V power (via 1 plug from the UPS) and networking (via 1 RJ45 cable to the router). The whole thing weighs about 35-50lbs and can easily be moved and redeployed anywhere. 

#### Sanctuary Systems Nameplate
The nameplate was manufactured by SendCutSend out of 0.063" 5052 aluminum and anodized black. The lettering was laser cut and I was honestly so blown away by the quality of the finished part. A DXF for a blanking plate is available, and all you need to do is add your own text or logo. The part cost about $30 USD to manufacture and took about 2 weeks to arrive, but in my opinion it is well worth it. 

#### Rack Shelves and Accessories
As the 10 inch rack form-factor is still relatively new, all of the shelves and accessories I bought came from DeskPi. I have a couple of the 0.5U shelves, 1 of the 1U shelves and a 0.5U 12 Port RJ45 patch panel. All of the blanking plates came with the rack itself, and I'm using them to cover the dead space at the bottom where the UPS lives. 

## Future Plans
Since my background is in industrial controls and automation, I really want to run an instance of HomeAssistant and go through the process of making my living space smart. Unfortunately I'm not a place where I can start making the necessary modifications to my house, but that will come soon enough. 

I also have an interest in software defined radio, and have ideas for both ADSB airplane tracking and downloading directly from the GOES weather satellites. Both require antennas and SDRs, but I've already got my eyes on some interesting hardware. 

## Conclusions
Once you start going down the self-hosting rabbit hole, there is no turning back. It is hard to imagine giving up the things i've grown accustomed to, and even though the hardware has cost a lot over time, it is nothing compared to shelling out hundreds of dollars a year for subscription services. Learning how to deploy, maintain, and use these services just opens the door for more capability in the future. The only real limitation to a setup like this is your willingness to do the work. High end hardware isn't required, and spending money is almost always optional, so the system can grow and change as your needs do. It will never be easy and it will never be quick, but it will always be well worth effort. 



