---
title: "Multi-Machine Sync with Syncthing"
date: 2026-08-20 21:52:08 +0200
categories: [Projets, Homelab]
tags: [syncthing, raspberry, automatisation, backup]
image:
  path: /assets/img/Syncthing/Syncthing_Logo.svg.webp
  alt: "Logo Syncthing"
permalink: /posts/syncthing-en/
---

> 💡 This article is entirely in English, the French version can be found [here](/posts/syncthing-fr/).

<!-- more -->

## Context and objective

When working across multiple machines and environments, a lot of time gets wasted manually copying files back and forth to keep everything up to date. The goal of this project was to automate synchronization of active workspaces (daily documents, scripts, Obsidian notes) so the same files are available regardless of the machine. I wanted a transparent solution, without transferring entire virtual disks or relying on public cloud.

## What is Syncthing?

Syncthing is an open-source peer-to-peer (P2P) file synchronization program.

Key characteristics:
- **No third-party cloud** — your files aren't stored on any unknown server, you keep full control of your data
- **Security** — transfers are fully encrypted (TLS), each device must be manually authorized
- **Cross-platform** — compatible with Windows, Linux, macOS

## Why Syncthing instead of a simple network share (NAS)?

Some will ask: *"Why set up a Raspberry Pi hub instead of using a network mount (SMB/NFS) to a server or NAS?"*

The answer comes down to one thing: offline mode. A network share is great for bulk storage or cold archiving. But for an active, mobile workspace, depending on a remote mount is a major constraint — if the VPN tunnel drops on a train or the Wi-Fi cuts out, the mount point breaks, the terminal freezes, and working becomes impossible.

With Syncthing, files live physically on the laptop's local SSD. I get the responsiveness and stability of local storage, even with zero connection. As soon as the computer regains network access (VPN or Wi-Fi), Syncthing silently pushes the delta of changes in the background. The server handles archiving, the Pi handles daily agility.

![alt](/assets/img/Syncthing/schema_etoile.png)

## The solution: a star topology

**Topology:**
- **Central hub** — Raspberry Pi 4, external disk for storage, passive relay
- **Satellite nodes** — physical Linux machine (laptop) and a desktop PC (file vault)

Nodes only ever talk to the hub, never directly to each other.

## Setup

Install on Linux nodes:
```bash
sudo apt update && sudo apt install syncthing
```

Enable in the background, no terminal needed:
```bash
systemctl --user enable syncthing.service
systemctl --user start syncthing.service
```

Interface available at `http://127.0.0.1:8384`.

## The roadblocks (and how I fixed them)

### 1. Docker container storage permissions

The Syncthing container on the RPi didn't have permission to create its `.stfolder` directory — `mkdir /Disk: permission denied`. Docker volume mapped: `/Disk/media` (host) → `/DATA` (container).

Forced creation and permissions via SSH:
```bash
sudo mkdir -p /Disk/media/Sync_files
sudo chmod -R 777 /Disk/media/Sync_files
```

### 2. Docker network isolation

During pairing, client machines (Linux and Windows PCs) showed the RPi as `Disconnected` — the RPi was advertising its container's internal IP, invisible from outside the Docker bridge. Fix: in the remote device's advanced settings on the client side, replace `dynamic` with:

```Plaintext
tcp://IP_RPI:22000
```

This forces routing over the host's actual LAN IP instead of the container's internal one.

### 3. The double versioning trap (conflict hell)

**The mistake to avoid at all costs.** I enabled Staggered File Versioning on every machine (Linux, Windows, and the hub).

Result: chaos. The moment the hub saved a note in real time, all three machines tried to create version histories simultaneously.

**Best practice (the star's golden rule):** versioning is enabled ONLY on the central hub.
- Satellite nodes (PC, laptops, VMs): File Versioning set to **None**
- Central hub (Raspberry Pi): File Versioning set to **Staggered**

This way, satellites act as simple mirrors, and only the RPi silently archives history in case of an error or accidental deletion.

## Takeaway

The architecture now handles disconnections gracefully. Working offline on my laptop, Syncthing stores the delta locally. As soon as I'm back on the network (LAN or VPN), files get pushed silently to the RPi, then picked up by other machines as soon as they're online — no manual intervention needed.