---
title: NixOS server migration issues 
date: 2025-05-26
permalink: /blog/2025/05/26/nixos-server-migration-issues
tags: learning nixos server issues
---

While I was migrating my NixOS server to a new PC I hoped for a relatively easy time. The plan was simple:
- Run the backup before migration
- Set up the new PC and deploy the same NixOS config
- Restore on the new PC from the lates backup.

Well, things did not turned out that simple.

### UID and GID issues

Restic backes up all the files with their absolute path and all the metadata. This metadata include the ownership and privileges of the files too. After doing the restore, checking the new PC I noticed that ownerships are messed up. For example mealie files were owned by jellyfin. It was all random.

This happens because these services generate users on the fly so their UID and GID is relatively random (somewhere in the 800-1000 range). The solution was simple. I painstakingly looked up the UID and GID for the services on the old PC in the `/etc/passwd` and `/etc/groups` files and manually declared them in the NixOS configuration. Then I just had to completely reinstall the new machine as I did not wanted to take the chance to mess up ownerships for some other system services.

In the future, I should define all service's ID that I back up explicitly, and somewhere in the 2000 range so it will not colide with other system services.

### Missing backup data (symlinks)

After all seemed to be working, I realized that some services did not retain data, which meant that either the backup or the restore did not process their files. Looking into it I learned 2 things:
- Restic does not follow symlinks (which is good)
- Some NixOS services instead of doing `/var/lib/servicename/files` they put the `files` into `/var/lib/private/servicename/files` and just drop a symlink to thi folder in `/var/lib`. This is also good for some added security.

Here the solution was simple, I had to add to all the services that used the `private` folder the path to the real files, alongside the symlink.

### Postgresql backup and restore

I currently just use one service that is actually using other than `sqlite` database. Immich is creating and saving most of it's data to `postgresql`. There is already a service for PG to back up the database, but restorin it is not included.

So what I ended up doing is:
- Created a systemd service that is dumping the Immich database
- Made the backup service call this service before backup
- Added the database dump into the backup paths.

On the restoration side it was bit more complex. I wrote a script that did the following:
- Stop immich services (require super user)
- Delete the immich database (requires `psql` admin account)
- Restore the Immich database (require immich service account)
- Start the service (requires super user)

The script has to be executed as root or super user that can assume the role of any other user on the system to be able to complete all the steps. All these steps are required to avoid conflicts with existing database and to have the right to read the dump file.

### Other small issues

Initially setting up the QEMU image for HomeAssistant I used the command to run the immage inside the service. Unfortunatelly had some whitespace issue when passing USBs to the VM while generating the command dynamically. The problem was not apparent as it kept saying that it is trying to boot from PCIe device.

Another issue with Home assistant migration was the trusted proxy. In Proxmox the containers communicated with `192.x.x.x` address with each other while in NixOS the default interface become `10.x.x.x`. This meant that I had to change the Trusted Proxy list before could access it from my reverse proxy.

The last weird issue was of course NVIDIA related. For ollama to recognize the 3090 GPU I had to install not only the drivers, but the OpenGL and XServer drivers too. Also `open` drivers did not work, even though they officially recommend them by now.

