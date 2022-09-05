---
title: Docker Compose – Media Stack
author: Will Garbutt

date: 2022-08-01

---


I thought I would share my first attempt at a docker compose file. This file deploys the containers I need to download and manage my media files.

The first part of this script deploys [NZBGet][1], [Sonarr][2], [Radarr][3], and [Plex][4] containers and links to volumes for configuration data and NAS storage will be found.

```
version: "3.4"
services:
  nzbget:
    container_name: nzbget
    image: linuxserver/nzbget:latest
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=${TZ} # timezone, defined in .env
      - PUID=0
      - PGID=0
    volumes:
      - downloads:/downloads # download folder
      - nzbgetconfig:/config # config files
 
  sonarr:
    container_name: sonarr
    image: linuxserver/sonarr:latest
    restart: unless-stopped
    network_mode: host
    environment:
      - PUID=0
      - PGID=0
      - TZ=${TZ} # timezone, defined in .env
    volumes:
      - sonarrconfig:/config # config files
      - tv:/TV # tv shows folder
      - downloads:/downloads # download folder

  radarr:
    container_name: radarr
    image: linuxserver/radarr:latest
    restart: unless-stopped
    network_mode: host
    environment:
      - PUID=0
      - PGID=0
      - TZ=${TZ} # timezone, defined in .env
    volumes:
      - radarrconfig:/config # config files
      - movies:/Movies # movies folder
      - downloads:/downloads # download folder

  plex-server:
    container_name: plex-server
    image: plexinc/pms-docker:latest
    restart: unless-stopped
    environment:
      - PUID=0
      - PGID=0
      - TZ=${TZ} # timezone, defined in .env
    network_mode: host
    volumes:
      - plexconfig:/config # plex database
      - plexconfig:/transcode # temp transcoded files
      - tv:/tv # media library
      - movies:/movies # media library

```

The second part setups up the actual volumes, I have mounted my NAS as a CIFS share on my Docker VM so media can be shared between Docker and other VMs when needed
```
volumes:
  downloads:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/data/container_config/nas/Downloads'

  tv:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/data/container_config/nas/TV'

  movies:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/data/container_config/nas/Movies'
  
  plexconfig:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/data/container_config/plex'

  sonarrconfig:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/data/container_config/sonarr'

  radarrconfig:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/data/container_config/radarr'

  nzbgetconfig:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/data/container_config/nzbget'
```

I actually deploy and manage this stack via [Portainer][5] as I find at this point in time the GUI very useful. Over time I hope to remove the reliance of the GUI in favour of command line.

 [1]: https://nzbget.net/
 [2]: https://sonarr.tv/
 [3]: https://radarr.video/
 [4]: https://www.plex.tv/
 [5]: https://www.portainer.io/