---
title: "Home Assistant"
date: 2026-10-25
lastmod: 2026-10-25
slug: home-assistant
type: posts
draft: false
categories:
  - Smart Home
tags:
  - iot
  - home assistant
  - linux
  - docker
  - rsync
---
[Home Assistant](https://www.home-assistant.io/) deployment and management notes using Docker. The main limitation for the Docker version of Home Assistant is that additional application aren't installable through the GUI and must be installed separately.

## Deploy 
Create directory `/opt/homeassistant/` and create the following `docker-compose.yml` file into it. Available image tags can be checked [here](https://ghcr.io/home-assistant/home-assistant).

```yml
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:2026.10"
    restart: unless-stopped
    volumes:
      - /opt/homeassistant/config:/config
      - /run/dbus:/run/dbus:ro
    privileged: true
    network_mode: host
    environment:
      - TZ=Europe/Zurich
```

Start the service with `docker compose up --detach` and it's done !

## Update
It is usually as simple as pulling the newest docker image. Doing a full backup beforehand may be a good idea.

```bash
cd /opt/homeassistant/

# Stop the service
# Note: target image could be downloaded in advance
docker compose down

# (optional) Set the docker image tag to wanted version
nano docker-compose.yml

# Pull the latest image according to your tag
# Note: you can pull it before stopping HA to drastically reduce downtime
docker image pull ghcr.io/home-assistant/home-assistant:wanted_version

# Start the container again and check everything works again
docker compose up --detach

# Cleanup (careless way)
docker image prune -a

# Cleanup (careful way)
docker image ls
docker image rm ghcr.io/home-assistant/home-assistant:the_unused_version
```

## Backup
Once configured through the GUI, automatic backup files land in `/opt/homeassistant/config/backups/`.

The following `rsync` command can be used to pull them on the backup server.

```bash
rsync --verbose your_user@homeassistant_server:/opt/homeassistant/config/backups/* /local/backup/dir/for/homeassistant/
```

Those backup file can be restored on any fresh setup of HA, as long as versions are compatible and the backup key still is in your possession.
