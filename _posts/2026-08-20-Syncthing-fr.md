---
title: "Synchronisation multi-machines avec Syncthing"
date: 2026-08-20 21:52:08 +0200
categories: [Projets, Homelab]
tags: [syncthing, raspberry, automatisation, backup]
image:
  path: /assets/img/Syncthing/Syncthing_Logo.svg.webp
  alt: "Logo Syncthing"
---

[Read in English](/posts/Multi-Machine-Sync-with-Syncthing/)

## Contexte et objectif

Quand on travaille sur plusieurs machines et environnements, on perd souvent un temps précieux à copier-coller manuellement les fichiers pour tout garder à jour. L'objectif de ce projet est d'automatiser la synchronisation des espaces de travail actifs (documents du quotidien, scripts, notes sur Obsidian) pour avoir systématiquement les mêmes fichiers sous la main, peu importe la machine. 
Je voulais une solution transparente, sans avoir à transférer des disques virtuels entiers ou dépendre du cloud public.

## Qu'est-ce que Syncthing ?

Pour résumer rapidement, **Syncthing** est un programme open-source de synchronisation de fichiers en pair-à-pair (P2P).

**Caractéristiques principales :**
* **Sans cloud tiers :** Vos fichiers ne sont stockés sur aucun serveur inconnu, vous gardez le contrôle total de vos données.
* **Sécurité :** Les transferts sont entièrement chiffrés (TLS) et chaque appareil doit être autorisé manuellement.
* **Multiplateforme :** Compatible avec Windows, Linux, macOS.

## Pourquoi Syncthing et pas un simple partage réseau (NAS) ?

Certains évoqueront surement : *"Pourquoi monter un hub Raspberry Pi au lieu d'utiliser un montage réseau (SMB/NFS) vers un serveur ou un NAS ?"* 

La réponse tient en un mot : **le mode hors ligne**. 
Un partage réseau est parfait pour du stockage de masse ou de l'archivage à froid. Mais pour un espace de travail *actif* et nomade, dépendre d'un montage distant est une contrainte majeure :
"Si le tunnel VPN saute dans le train ou si le Wi-Fi coupe, le point de montage bug, le terminal plante, et il est impossible de travailler"

Avec Syncthing, les fichiers sont présents physiquement sur le SSD local du laptop. Je travaille avec la réactivité et la stabilité du stockage local, même sans aucune connexion. Dès que l'ordinateur retrouve un accès réseau (VPN ou Wi-Fi), Syncthing pousse silencieusement le delta des modifications en tâche de fond. Le serveur gère donc l'archivage, et le Pi gère l'agilité au quotidien.

![alt](/assets/img/Syncthing/schema_etoile.png)

## La solution : une topologie en étoile

Pour cela, on va déployer une architecture centralisée avec **Syncthing**, en utilisant le matériel suivant :
- Un Raspberry Pi 4 comme hub de relais permanent (toujours allumé, cela ne consomme pas énormément).
- Les machines sur lesquelles vous souhaitez que les fichiers soient synchronisés. Elles ne communiqueront qu'avec le hub central.

J'ai abandonné l'idée de transférer mes VM entières pour adopter une approche "in-guest" : synchroniser uniquement les fichiers de travail à l'intérieur des VM et de mes hôtes via Syncthing.

**Topologie :**
* **Hub central** — Raspberry Pi 4, avec un disque externe pour le stockage, relais passif.
* **Nœuds satellites** — Machine physique Linux (Laptop) et un PC fixe (coffre de fichiers).

Les nœuds ne communiquent qu'avec le hub, jamais entre eux directement.

## Mise en place

Installation sur les nœuds Linux :

```bash
sudo apt update && sudo apt install syncthing
```
Activation en tâche de fond, sans terminal ouvert :

```bash
systemctl --user enable syncthing.service
systemctl --user start syncthing.service
```
Interface accessible sur http://127.0.0.1:8384

## Les blocages possible (et comment les résoudre)

1. Droits de stockage sur le conteneur Docker

Le conteneur Syncthing du RPI n'avait pas les droits pour créer son dossier .stfolder — erreur mkdir /Disque: permission denied.
Volume Docker mappé : /Disque/media (hôte) -> /DATA (conteneur).

Forcer la création et les droits :
```bash
sudo mkdir -p /Disque/media/Sync_fichier
sudo chmod -R 777 /Disque/media/Sync_fichier
```

2. Isolation réseau Docker

Lors du jumelage, le côté "client" (Pc linux et Windows) affichait le RPi en Disconnected — le RPi annonçait l'IP interne de son conteneur, invisible depuis l'extérieur du bridge Docker.
Il faut donc modifier les paramètres avancés de l'appareil distant côté client via l'interface web, remplacé dynamic par :

```Plaintext
tcp://IP_RPI:22000
```
Ça force le routage sur l'IP LAN réelle du host plutôt que sur l'IP interne du conteneur.


3. Le piège de la double gestion de versions (L'enfer des conflits)

C'est **l'erreur** à ne surtout pas faire ! Activé l'option "Versions échelonnées" (Staggered File Versioning) sur toutes mes machines (Linux, Windows et le Hub).

Résultat : la catastrophe. Dès que le hub sauvegardait une note en temps réel, les trois machines tentaient de créer des historiques de versions simultanément. 

La bonne pratique (Règle d'or de l'étoile) :
Le versionnage ne s'active QUE sur le Hub central.

    Noeuds satellites (PC, Laptops, VM) : Préservation des fichiers sur Aucune.

    Hub central (Raspberry Pi) : Préservation des fichiers sur Versions échelonnées.

Ainsi, les satellites sont de simples miroirs, et seul le RPi archive l'historique silencieusement en cas d'erreur ou de suppression accidentelle.

## Bilan
L'architecture gère désormais parfaitement les déconnexions. Si on travail hors ligne sur mon laptop, Syncthing stocke le delta localement. Dès qu'on retrouve le réseau (LAN ou VPN), les fichiers sont poussés silencieusement vers le RPI, puis récupérés par les ordinateurs non synchronisé dès qu'ils sont allumés — sans intervention manuelle.
