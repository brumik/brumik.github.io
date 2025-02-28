---
title: Incremental development of my home lab
date: 2025-02-28
permalink: /blog/2025/02/25/incremental-development-of-my-lab
---

I had been at work to create a stable home server (doubbed as lab by many) for me and my close family. Initially I was just bringing up simple docker containers on my Synology server pretty much ad-hock. This was working just fine for the time being. However it was something that I could not scale too well, not to mention maintenance.

## Docker (and compose)

The mainstream way to deploy some services on your home lab I would say is `docker` and especially `docker compose`. These two services are awesome in general (I highly recommend learning them if you are in **any** software engineering role). You can use it on all the major player on the home labbing market: TrueNAS Scale, Unraid, or if you just roll with plain Debian on your machine. 

So I got to work and put up a nice list of services in a dockerized and uniform environment. My stack was around 10 applications (like mealie, jellyfin, or bitwarden) and other 10 supporting services like Traefik for reverse proxy, Authelia and Lldap for user management and few monitoring and updating tools.

This was solid setup, with pretty good stability and with a possibility of auto update for some services with notifications. I also learned a **lot** about docker networking, setting up complex(ish) chains of tools and of course secrets management. I would not say I became a wizzard with docker, but I am super happy about the knowedlege that I gained. 

There were some other shenanigans going on with setting up emailing service and Dynamic DNS and such, but for this blogpost it is not relevant.

### The issue

As stable as it was, there was one big issue with the dockerized setup, and that was the fragmentation of configuration files. Some files like `AdGuard` config was hard to edit from a text editor and I relied on the UI to make changes. Then `Traefik` was half `yml` and half `cli` configuration, while authelia used it's own `configuration.yml`. Most of services had to be configured from environment variables mixed with some UI or other file configurations. 

Afther a while this got a bit old. However I did not thought of changing it yet.

## NixOS journey

As somebody who read my blog at all, you probably know that I am very beginner in NixOS. For a long time I had on my list to pick up SOPS for secrets management as a nice addition (I was using a crappy but working script I wrote to download my secrets from bitwarden). However the complexity was a bit too high for me. Then I seen that many people on `reddit` (where else) are super satisfied with their home lab setup on NixOS.

In paralel I was trying to get away from Synology as my main NAS (or at least moving into a direction that is less dependent). I was initially looking at TrueNAS Scale (tha can do docker). However that would add just one more place to configure my stuff and one more layer of complexity. Thinking about it, the only thing it would give me is a nice UI for ZFS and SMB. 

But wait, I can do ZFS and SMB relatively easily on NixOS already. So here we go, I am in the middle of migrating all my docker services to NixOS. 

### `rm -rf /` and first test of my backup solution

I started with only one service and figuring out backup. I found `restic` and its NixOS package, where the setup was a breeze. I had not much idea how it works but the elegance was stunning.

So you set up the service first so you have a `systemd` timer runnig it, for an extra simplicity in our own module:

```nix
  # create your own module that can be enabled
  options.homelab.backup = {
    enable = lib.mkEnableOption "backup";

	# This is an array of absolute paths we want to back up
    stateDirs = lib.mkOption {
      type = lib.types.listOf lib.types.str;
      default = [ ];
    };
  };

  config = lib.mkIf cfg.enable {
    services.restic.backups = {
      remotebackup = {
        initialize = true;
        paths = cfg.stateDirs;
        repository = "/mnt/share/resticBackup";
        passwordFile = config.sops.secrets."n100/restic-password".path;
        pruneOpts = [ "--keep-daily 4" ];

        timerConfig = {
          OnCalendar = "00:01";
          RandomizedDelaySec = "2h";
        };
      };
    };

```

As you can see this is not defining **what** to back up, just where and how. But now we can add paths to it in different service like backing up our bitwarden:

```nix
	# Enable and configure the service
    services.vaultwarden = {
      enable = true;
      config = {
        DOMAIN = "https://bitwarden.${config.homelab.domain}";
        SIGNUPS_ALLOWED = false;
        ROCKET_ADDRESS = "${cfg.address}";
        ROCKET_PORT = cfg.port;
      };
    };

	# Magically add the paths when this module is used to our backup
    homelab.backup.stateDirs = [
      "/var/lib/vaultwarden/attachments"
      "/var/lib/vaultwarden/db.sqlite3"
    ];
```

Now you have a service that just needs to define what to back up, and it works. For a good measure I also add my `/home/user/docker` folder, since it turns out that my makeshift script that is backing up that folder is not working for 3 months already (failing succesfully, yey)

Fast forward a day, I am setting up `mealie` and migrating the data. The migration consists of 2 steps:
- Copy the `docker/appdata/mealie/data/*` to `/var/lib/mealie`
- Make the `mealie:mealie` user and group the owner of the copied data.

Simple just `cd /var/lib/mealie` use root and then `cp -r /from /to` firts. Now we just do `chowm -R mealie:mealie /*`. Wait what? Yeah, you cannot do anymore `rm -rf /` but giving the ownership of all files in a system to a limited user is not much better. Now all my files are owned by `mealie`. Trying to fix it is a bit futile and rebooting does not work anymore. 

So I left the messed up system alone, spin up a new VM, deployed the same config and prayed that my untested `restic` setup works somehow. So after some digging it turns out that the NixOS service creates not only the systemd service, but also a wrapper with all the evniroment variables defined. 

Then looking at the documentation it says I could just run `restic restore latest --target xxxx`. What is target. I try out only as a test on /home/test and it turns out all the paths I defined above are created realitively to the `--target path` with the **same permissions and owners**. So I just run `restic-mybackup restore lates /` and after fer minutes of waiting I get back everything.

Gotta say, this was my greatest success story in my home lab since I have it. Also I was super lucky.

### Overal experience

In the end the overal experience with NixOS is great. Most of the config can be written in nix (except the UI and database stuff). There are already often sesible defaults so I find myself writing way less code to make something spin up. The security part of it is also easier since now I have to deal with linux only, not to mention the service monitoring got simplified too. 

There is of course the issue that some services are not absolute latest in the stable channel, and using from unstable is not ideal (or you need a lot of workaround). So migrating some sevices a major version back is kinda a pain.

The biggest win maybe for me is the fact that now I can write one nix configuration and get both my server, my pc and my girlfriend's pc in my house updated to it (thanks to the remote deployment capabilities of nix). This also means that I can pin IP addresses and configure SSH keys in the config itself.

In general having all your systems as a derivate of your program (deployment as code I guess?) is way easier to debug and come back to it later than having it in a dozen different formats and files.
