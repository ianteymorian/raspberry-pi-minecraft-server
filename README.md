# PiCraftSMP 🎮

A 24/7 Minecraft Java server hosted on a Raspberry Pi 4, with remote access, monitoring, automated backups, and a custom web dashboard.

## Features

-  24/7 Minecraft Java server
-  Hosted on a Raspberry Pi 4 with 8 GB RAM
-  Remote access using Playit.gg
-  Custom web-based monitoring dashboard
-  Live player and server status monitoring
-  Raspberry Pi temperature and system monitoring
-  Remote server administration
-  Automated backups every 48 hours
-  14-day backup retention
-  Backups automatically synced to a Windows PC
-  SSH-based remote management

## How It Works

Players connect through Playit.gg, which routes Minecraft traffic to the Raspberry Pi without requiring traditional port forwarding.

```mermaid
flowchart LR
    A[Minecraft Players] --> B[Playit.gg]
    B --> C[Raspberry Pi 4]
    C --> D[PiCraftSMP]
    C --> E[Web Dashboard]
    C --> F[Automated Backups]

##  Hardware

| Hardware | Specification |
|---|---|
| Server | Raspberry Pi 4 |
| RAM | 8 GB |
| Architecture | ARM64 / aarch64 |
| Cooling | Active cooling |
| Network | Ethernet / Wi-Fi |
| Backup device | Windows PC |

##  Software

| Software | Purpose |
|---|---|
| Raspberry Pi OS | Server operating system |
| Java | Runs the Minecraft server |
| Minecraft Java Server | PiCraftSMP game server |
| Playit.gg | Public Minecraft access |
| Port Warp | Network routing / access |
| Tailscale | Private remote access |
| SSH | Remote server administration |
| Custom Web Dashboard | Monitoring and server control |
| Bash scripts | Server and backup automation |
| Windows PowerShell | PC-side backup automation |

##  Networking & Remote Access

PiCraftSMP is designed to remain accessible without requiring players
to install additional networking software.

### Playit.gg

Playit.gg provides the public Minecraft connection and allows players
outside the local network to connect without traditional router port
forwarding.

### Port Warp

Port Warp provides an additional networking solution for accessing
the Minecraft server.

### Tailscale

Tailscale provides private access to the Raspberry Pi for server
administration.

It allows the server to be securely managed remotely without exposing
administrative services directly to the public Internet.

### SSH

SSH is used to remotely access the Raspberry Pi and perform tasks such
as:

- Starting and stopping the Minecraft server
- Checking server status
- Managing files
- Running backups
- Monitoring the Raspberry Pi
