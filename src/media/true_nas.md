# Setting up NAS for media streaming and back up

So it came down to TrueNas and Unraid. For a homelaber looking to spend as little as possible, I really wanted to go with Unraid so I can mismatch drives. Of course there's also the upfront cost, so I ended up with TrueNas.

## Machine

- HP Elitedesk 800 G4 Intel core i5-8500 that I picked up on Ebay for $115 with 8GB RAM and 256GB NVME - which I turned into a bootdrive.
- Upgrades:
  - Another 8GB RAM for a total of 16GB. There are 4 slots so I might go to 32GB with [Immich](https://github.com/immich-app/immich) if the need arises.
  - 2.5 Gbe NIC
  - 2x 6TB HHD in mirrow set up -used from Ebay and Amazon
  - 2x 256GB NVME (original install and another I purchased)
  - 1x 6TB - this is sitting on the shelf waiting for a failed HDD.

## Set up

I set up a media pool (tank) and an apps/config pool (nvme_pool).
The media pool is a mirrow of the 2x 6TB HDDs. Most of the items on here are movies/tv that I already have on DVD. I don't want to lose them
but it wouldn't be the end of the world to lose them.

- I'm using [Jellyfin](https://jellyfin.org/) for streaming with the following mapping:

/tank/media/movies
/tank/media/tv

The set up is simple and works for now. With 4 streams going, the CPU usage is hovering around ~35%-40% and RAM barely moved. The plan is to add more services then see how it goes. The media system will probably get the heaviest usage of all the services in the house so an upgrade is probably in the near future.

## Data protection

As of right now, I'm only backing up configs and /tank/docs and soon /tank/media/photos. As I mentioned before, I'm not going to lose my life if all items got wiped. The items I don't want to lose will be backed up to my primary machine for now. Personal photos and documents are important to me. Movies and TV shows are not.

I've set up replication, back up, and alerts for important items.

Important stuff get's backed up to Backblaze.

## Notes:

- So I ended up filling up the additional slots. Machine now sits at 32GB. That ZFS cache is no joke.
- I'm currently running the following services on this machine:
  - immich
  - jellyfin
- It's really convenient running TrueNas with all these media related services via [docker containers](https://technotim.live/posts/truenas-docker-pro/). The data is here and I don't have to mount over the nextwork in another machine. I'm tempted to run the next few services (Nextcloud, Paperless-ngx) on here because of that.
