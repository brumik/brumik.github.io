---
title: Home Services with Reverse Proxy and VPN
date: 2024-05-10
permalink: /blog/2024/08/14/how-to-set-up-vpn-ssl-proxy
---

To reach your home server or other services using nicely formatted domain names, you need either an IP address reachable from the internet or a VPN. In both cases, you will likely need to set up a reverse proxy.

This article walks you through how to set up Tailscale and Nginx Proxy for your server (Synology NAS) or other services running on your local network.

I am using my Home Assistant Raspberry Pi to facilitate the proxy, but you can use any other machine.

## Setup with Nginx Proxy Manager & Tailscale

For reference, my initial setup was as follows:
- Synology NAS running on `192.168.1.2`, exposing services on ports reachable from the LAN.
- RPi Home Assistant running on `192.168.1.3`.
- Gateway router running on `192.168.1.1`.

### Setup of Dependencies (for Home Assistant):

- [Tailscale](https://tailscale.com/) - Install and log in; this will create the private virtual network accessible from anywhere.
	- Make a note of the private IP address that Tailscale assigns to your device (something like `100.61.92.2`).
- [Nginx Proxy Manager](https://nginxproxymanager.com/guide/) (NPM) - Ensure that it can listen to external ports 80 and 443. If needed, remap the service listening to these ports.
- [AdGuard](https://adguard.com/en/welcome.html) - This will block trackers, but this functionality can be disabled.
- Set up a [Duck DNS](https://www.duckdns.org/) account (it is free).

### Set Up the Reverse Proxy with SSL Certificate

In this step, we will set up the DNS server and the reverse proxy in a way that provides you with a reusable SSL certificate and allows other services like browsers to recognize it.

- In Duck DNS, set up a new domain name (e.g., `myserver.duckdns.org`) and point it to the private IP from the Tailscale network assigned to your Home Assistant where the reverse proxy will be running (e.g., `100.61.92.2`).
- Open the URL for Home Assistant on the port where NPM is running (default 81).
	- The same steps are described in [this video](https://youtu.be/qlcVx-k-02E?si=kwWv2SDGJMMwki8I&t=413).
	- **Set up the SSL certificate**:
		- Select the SSL certificates tab.
		- Add an SSL certificate.
		- Fill out the domain names with your domain from Duck DNS (e.g., `myserver.duckdns.org` and `*.myserver.duckdns.org`). The second part of the example ensures it works for any subdomains.
		- Select the `Use a DNS Challenge` option and choose Duck DNS from the dropdown.
		- Copy your token from the Duck DNS website into the box and press generate.
	- **Set up the first proxy**:
		- Select the Host tab and Proxy Host from the menu.
		- Select Add Proxy Host.
		- Fill out the domain name and ports where NPM can reach the applications. For example:
			- Domain: `npm.myserver.duckdns.org`
			- `https://192.168.1.3:81` (your LAN address)
			- In the SSL tab, select the generated certificate and `Force SSL`.
		- Now you can visit `npm.myserver.duckdns.org` when logged into the same Tailscale network. The website will be considered "safe" as we have the correct certificates for it.
	- **Note**: Many services will fail if we do not enable WebSocket capabilities for the proxy. For example, my Synology refused to connect, and both Home Assistant and Jellyfin require WebSocket for streaming. However, Bitwarden works perfectly without it.

#### How It Works

To have an SSL certificate for a domain name, it has to be public. In this case, we created a public domain `myserver.duckdns.org`. For this domain, we acquired an SSL certificate from Let's Encrypt, and the certificate is saved within our Nginx Proxy Manager. Every time we proxy something, this is the certificate that will be sent to the browser/application. The application can then look it up in the public registry and mark it as valid, so there’s no need to install certificates on your devices each time.

However, you may have noticed that we did not assign a public IP address to the DNS record. You might have one, but exposing it on the internet is not recommended, as it can leave you vulnerable to attacks. If you do, you’ll need to rely on the security of the services you run on your network and your firewalls.

Instead, we assigned the DNS service a private IP address from Tailscale that cannot be tied to you nor accessed from the internet outside your Tailscale network. This means that when you type `myserver.duckdns.org`, it will return a private address. For this to work, you need to be on your Tailscale VPN network. This setup allows you to access your services from anywhere, not just home. However, you might want to access the services from home without using Tailscale.

### Set Up DNS Rewrites When on LAN

We installed AdGuard on our Home Assistant already. AdGuard comes with the capability to rewrite DNS records.

Since you have access to the LAN when at home, you can add the following rewrites:
```
myserver.duckdns.org: 192.168.1.3
*.myserver.duckdns.org: 192.168.1.3
```

This will ensure that when we query this DNS server, we immediately get our LAN network IP for our running proxy. Since the DNS certificates still match, we continue using SSL without problems.

You can set up the DNS server separately on each of your devices, but a more general solution is to configure your router to advertise the DNS server on your network (in my case, `192.168.1.3`). Don’t worry; if it goes down, devices will fall back to the default. This usually only takes effect when devices request or renew their DHCP contracts.

## Why Not Use Reverse Proxy on the Synology NAS

My initial idea was to run the reverse proxy on my Synology, as all of my services are running there. I was already using the inbuilt Nginx reverse proxy for DDNS records, but its UI is rather limited and cannot obtain a Let's Encrypt certificate for wildcard domain names other than `*.synology.me`. Optionally, I could have disabled this proxy, freed up ports 80 and 443, and directed them to a container running NPM, but that would have been a hassle. Another option is to create a `macvlan` Docker network, but this also requires running a script on every restart with `sudo` access.

Moving the services to a separate device greatly simplified the setup and made it less complex for me. I already knew the LAN addresses for my services. Furthermore, debugging a service is now straightforward since I can check it on the LAN first and then add the same address to my NPM setup with a name.

I consider this a significant improvement and will move away from my DDNS setup in favor of this Proxy + VPN setup. 

