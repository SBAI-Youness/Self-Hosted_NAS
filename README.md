# Self-Hosted NAS: A Free Google Drive Replacement

A private cloud storage system built at home, using spare hardware, that handles file storage, photo backup, and secure sharing - with full CRUD access from anywhere in the world, and zero monthly fees.

---

## Why I Built This

<p align="center">
<img src="images/google-storage.png" alt="Google storage full" width="800"/>
</p>

My Google storage hit 14.83/15 GB. That's the free tier shared across everything - Drive, Gmail, Photos - and I'd maxed it out. Every time I wanted to upload something new, I had to go delete something old first, across three different apps, just to make 50 MB of room. For storage that was never really mine to begin with.

So instead of paying Google for more space, I dug out an old **Dell Latitude E6320** laptop that was sitting in a drawer and turned it into my own private cloud.

What I wanted out of it:
- **Storage that scales with my hardware, not a subscription tier**
- **Full ownership** of my documents and photos
- **No recurring fees**, ever
- **Remote access** from any device, anywhere
- **The ability to securely share files** with people who don't have any special software installed

This is the full walkthrough - what I installed, in what order, and why. Everything below is **free** and **open source**.

---

## Achitecture

<p align="center">
<img src="images/architecture.png" alt="Project Architecture" width="900"/>
</p>

### How it Works

The Dell Latitude E6320 runs **Ubuntu Server** - that's the NAS. It's the physical machine and OS in one. Everything else runs on top of it.

**Tailscale** is installed directly on Ubuntu and acts as the access layer. When you open Nextcloud from your own phone or laptop, the request comes in through Tailscale's **VPN tunnel** - encrypted end-to-end, with no ports open on the router. When you share a file with someone who doesn't have Tailscale, **Funnel** takes over - it opens a single public HTTPS endpoint just for that link, while the rest of the server stays completely unreachable from the outside.

Once a request is through Tailscale, it lands at **Docker**, which runs three containers: **Nextcloud** is the main app - the Google Drive replacement users actually interact with. **MariaDB** is the database behind it, storing metadata for every file, folder, user, and share link. **Redis** caches sessions and handles file locking to keep Nextcloud responsive. When a file is uploaded or opened, Nextcloud reads and writes it directly to the **local disk** - a mounted folder on Ubuntu's internal drive, outside of Docker, so the data persists regardless of container restarts.

### Tech Stack

<table>
  <colgroup>
    <col style="width: 180px;">
    <col>
    <col>
  </colgroup>
  <tr>
    <th>Tool</th>
    <th>What it is</th>
    <th>Role in this project</th>
  </tr>
  <tr>
    <td><img src="icons/ubuntu.png" width="30"/> <b>Ubuntu Server</b></td>
    <td>Barebones Linux with no desktop, built for servers</td>
    <td>Host OS - the foundation everything else runs on</td>
  </tr>
  <tr>
    <td><img src="icons/docker.png" width="30"/> <b>Docker</b></td>
    <td>Packages apps and their dependencies into isolated containers</td>
    <td>Runs Nextcloud, MariaDB, and Redis without any manual installation</td>
  </tr>
  <tr>
    <td><img src="icons/nextcloud.png" width="30"/> <b>Nextcloud</b></td>
    <td>Open-source, self-hosted alternative to Google Drive</td>
    <td>The main app - file storage, photo backup, sharing, CRUD</td>
  </tr>
  <tr>
    <td><img src="icons/mariadb.svg" width="30"/> <b>MariaDB</b></td>
    <td>A relational database (MySQL-compatible)</td>
    <td>Stores all of Nextcloud's metadata - files, users, folders, share links</td>
  </tr>
  <tr>
    <td><img src="icons/redis.png" width="30"/> <b>Redis</b></td>
    <td>An in-memory cache</td>
    <td>Keeps Nextcloud fast - handles session caching and file locking</td>
  </tr>
  <tr>
    <td><img src="icons/tailscale.png" width="30"/> <b>Tailscale</b></td>
    <td>A mesh VPN built on WireGuard, with a built-in feature called Funnel that can selectively expose services to the public internet</td>
    <td>VPN tunnel for my own devices - encrypted, no open ports. Funnel for everyone else - lets friends open shared links in any browser, no Tailscale needed</td>
  </tr>

---
