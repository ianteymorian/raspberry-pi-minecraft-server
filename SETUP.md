# PiCraftSMP Setup Guide

This guide explains how PiCraftSMP was built from scratch.

PiCraftSMP is a 24/7 Paper Minecraft Java server running on a Raspberry Pi 4 with:

- 8 GB RAM
- Raspberry Pi OS Lite 64-bit
- Raspberry Pi Connect remote administration
- SSH backup access
- Playit public access
- Port Warp as an additional public route
- Custom Minecraft server icon and MOTD
- Private web dashboard
- Tailscale private remote access
- Automated backups every 48 hours
- Automatic backup transfer to a Windows PC
- 14-day backup retention

> [!IMPORTANT]
> This repository does not contain passwords, authentication keys, private SSH keys, Playit claim tokens, Port Warp credentials, Tailscale authentication details, or the live public Minecraft addresses.
>
> Never put those values in a public GitHub repository.

---

# 1. Hardware

PiCraftSMP was built using:

- Raspberry Pi 4
- 8 GB RAM
- microSD card
- Active cooling
- USB-C Raspberry Pi power supply
- Wi-Fi or Ethernet
- Windows PC for backup storage

A **32 GB or larger microSD card** is recommended.

You also need a microSD card reader so the card can be connected to your normal computer.

You do **not** need a monitor, keyboard, or mouse connected to the Raspberry Pi.

PiCraftSMP is configured as a **headless server** and is controlled remotely.

---

# 2. Install Raspberry Pi Imager

On your normal Windows PC, go to:

<https://www.raspberrypi.com/software/>

Download and install:

```text
Raspberry Pi Imager
```

Insert the microSD card into your computer.

Open Raspberry Pi Imager.

> [!CAUTION]
> Writing Raspberry Pi OS to the microSD card will erase everything currently stored on that card.
>
> Make sure you select the correct storage device.

---

# 3. Configure Raspberry Pi OS

In Raspberry Pi Imager, select:

```text
Device:
Raspberry Pi 4
```

For the operating system select:

```text
Raspberry Pi OS Lite (64-bit)
```

The **Lite** edition is intentional.

A Minecraft server does not need the Raspberry Pi desktop environment, so the Lite version saves resources.

Select your microSD card as the storage device.

Continue to the configuration screens.

---

## Hostname

The original PiCraftSMP Raspberry Pi used:

```text
RasPi-Sever
```

You can use this, although a cleaner hostname for a new build would be:

```text
picraftsmp
```

---

## User account

The original PiCraftSMP user is:

```text
admin
```

Create your own strong password.

> [!WARNING]
> Never put your Raspberry Pi password in GitHub.

---

## Wi-Fi

If you want the Raspberry Pi to use Wi-Fi, enter:

```text
Wi-Fi SSID:
<YOUR_WIFI_NAME>
```

```text
Wi-Fi password:
<YOUR_WIFI_PASSWORD>
```

Select the correct Wi-Fi country for the location where the Raspberry Pi is being used.

If you are using Ethernet, the Pi can also connect automatically when an Ethernet cable is connected.

---

## Time zone

Select your local time zone.

For example, the PiCraftSMP server is located in China, so its local configuration can use the appropriate China/Shanghai time zone.

---

# 4. Enable Raspberry Pi Connect

PiCraftSMP uses **Raspberry Pi Connect** as its main remote administration method.

This lets you access the Raspberry Pi terminal from a web browser without needing a monitor or keyboard attached to the Pi.

In Raspberry Pi Imager, find the:

```text
Raspberry Pi Connect
```

configuration step.

Turn on:

```text
Enable Raspberry Pi Connect
```

Click:

```text
Open Raspberry Pi Connect
```

Your normal web browser will open.

Sign in using your:

```text
Raspberry Pi ID
```

If you do not already have a Raspberry Pi ID, create one.

---

## Create the Raspberry Pi Connect auth key

Raspberry Pi Connect will open a:

```text
New auth key
```

page.

Create the auth key.

The key is a **single-use temporary authentication token** that allows the new Raspberry Pi to link itself to your Raspberry Pi Connect account when it boots.

Your browser should ask for permission to reopen Raspberry Pi Imager.

Allow it.

Raspberry Pi Imager should then show that it received the authentication token.

If it does not appear automatically, open:

```text
Having trouble?
```

on the Raspberry Pi Connect page.

Copy the token and paste it into the token field in Raspberry Pi Imager.

Then continue.

> [!IMPORTANT]
> Personal Raspberry Pi Connect auth keys currently expire after **6 hours**.
>
> Flash and boot the Raspberry Pi while the key is still valid and make sure the Pi has Internet access.
>
> The key is only required to initially connect the new Pi to your account.

> [!CAUTION]
> Never put a Raspberry Pi Connect auth key in GitHub.

---

# 5. Enable SSH as a backup

Raspberry Pi Connect is the main remote-access method used by PiCraftSMP.

However, SSH is also enabled as a useful backup.

In Raspberry Pi Imager, enable:

```text
SSH
```

For the initial setup, allow:

```text
Password authentication
```

This lets you connect from another computer using:

```powershell
ssh admin@<PI_IP_ADDRESS>
```

---

# 6. Flash the microSD card

Check all of the Raspberry Pi Imager settings.

You should now have configured:

- Raspberry Pi 4
- Raspberry Pi OS Lite 64-bit
- Hostname
- `admin` user
- Password
- Wi-Fi if required
- Wi-Fi country
- Time zone
- Raspberry Pi Connect
- SSH

Select the option to write the operating system.

Confirm that the selected microSD card can be erased.

Wait for Raspberry Pi Imager to:

1. Write Raspberry Pi OS.
2. Verify the microSD card.

Do not remove the card during this process.

When Imager reports that it has finished, safely eject the microSD card.

---

# 7. First Raspberry Pi boot

Make sure the Raspberry Pi is unplugged.

Insert the microSD card into the microSD card slot on the underside of the Raspberry Pi.

Make sure the cooling system is connected.

If using Ethernet, connect the Ethernet cable.

Then connect the USB-C power supply.

The Raspberry Pi will start automatically.

Wait approximately:

```text
2-3 minutes
```

The first boot can take longer than later boots.

The Pi needs Internet access during this boot so Raspberry Pi Connect can use the temporary auth key and register the Pi with your Raspberry Pi ID.

---

# 8. Open Raspberry Pi Connect

On your normal computer, open:

<https://connect.raspberrypi.com/>

Sign in using the same Raspberry Pi ID used during Raspberry Pi Imager setup.

Your Raspberry Pi should appear in your device list.

Select it.

Open:

```text
Remote shell
```

You now have a Raspberry Pi terminal directly in your web browser.

It should look similar to:

```text
admin@RasPi-Sever:~ $
```

From this point onward, commands marked as:

```text
bash
```

are entered into the Raspberry Pi terminal.

For PiCraftSMP, this normally means the **Raspberry Pi Connect Remote Shell**.

---

# 9. SSH backup access

You can also access the Pi using SSH.

On a Windows PC, open PowerShell.

If `.local` hostnames are working:

```powershell
ssh admin@RasPi-Sever.local
```

Or, if you used the cleaner hostname:

```powershell
ssh admin@picraftsmp.local
```

The first time you connect, SSH may ask whether you trust the computer.

Type:

```text
yes
```

Enter the Raspberry Pi password.

Nothing will appear on screen while typing the password.

That is normal.

If the `.local` hostname does not work, use the Pi's IP address.

For example, the final PiCraftSMP Wi-Fi IP was:

```powershell
ssh admin@192.168.10.2
```

Your Raspberry Pi may have a different IP.

---

# 10. Check the Raspberry Pi

Check the CPU architecture:

```bash
uname -m
```

The Raspberry Pi 4 used for PiCraftSMP reports:

```text
aarch64
```

Check the IP address:

```bash
hostname -I
```

Check RAM:

```bash
free -h
```

Check storage:

```bash
df -h /
```

Check temperature:

```bash
vcgencmd measure_temp
```

The PiCraftSMP dashboard has shown approximately:

```text
7.64 GB usable RAM
```

The server uses active cooling.

---

# 11. Update Raspberry Pi OS

Update the package list:

```bash
sudo apt update
```

Install available system updates:

```bash
sudo apt full-upgrade -y
```

---

# 12. Install Java

Install Java and `wget`:

```bash
sudo apt install openjdk-25-jre-headless wget -y
```

Check Java:

```bash
java -version
```

Find the exact Java executable:

```bash
readlink -f "$(command -v java)"
```

The later working PiCraftSMP service used:

```text
/usr/lib/jvm/java-21-openjdk-arm64/bin/java
```

Your installation may return a different path.

Use the path actually shown on your Pi when configuring the Minecraft service.

---

# 13. Create the Minecraft directory

Create the Minecraft server folder:

```bash
mkdir ~/minecraft
```

Enter it:

```bash
cd ~/minecraft
```

---

# 14. Install Paper Minecraft

The Paper build downloaded during the original PiCraftSMP setup was:

```bash
wget https://fill-data.papermc.io/v1/objects/7b7b3b43c009103e1971a0576c26f655a7dd9b56a0a2a4438e352c03a7fecd08/paper-26.2-123.jar -O paper.jar
```

Start Paper:

```bash
java -Xms2G -Xmx4G -jar paper.jar --nogui
```

The memory configuration means:

```text
-Xms2G = start with 2 GB
-Xmx4G = maximum of 4 GB
```

Paper will create its files and then stop because the Minecraft EULA has not yet been accepted.

---

# 15. Accept the Minecraft EULA

Open:

```bash
nano eula.txt
```

Find:

```text
eula=false
```

Change it to:

```text
eula=true
```

Save using:

```text
Ctrl+O
```

Press Enter.

Exit using:

```text
Ctrl+X
```

Start Minecraft again:

```bash
java -Xms2G -Xmx4G -jar paper.jar --nogui
```

Wait for the server to finish starting.

To stop it safely, type:

```text
stop
```

The finished PiCraftSMP server later reported:

```text
Paper 1.21.11
```

---

# 16. Customize the Minecraft server list

PiCraftSMP was customized so that the Minecraft Multiplayer menu shows:

- A custom Raspberry Pi icon
- `PiCraftSMP` in red
- `SURVIVAL • 1.21.11` underneath

---

## Configure the MOTD

Go to the Minecraft folder:

```bash
cd ~/minecraft
```

Open:

```bash
nano server.properties
```

Find the line beginning with:

```text
motd=
```

Set it to:

```text
motd=\u00A7cPiCraftSMP\n\u00A7fSURVIVAL • 1.21.11
```

The formatting codes mean:

```text
\u00A7c = red
\u00A7f = white
\n      = new line
```

The Minecraft server list will therefore display:

```text
PiCraftSMP
SURVIVAL • 1.21.11
```

with the first line in red.

Save using:

```text
Ctrl+O
```

Press Enter.

Exit using:

```text
Ctrl+X
```

---

## Add the custom server icon

Minecraft uses a file named:

```text
server-icon.png
```

The image needs to be:

```text
64 × 64 pixels
PNG format
Filename: server-icon.png
```

The final file must be located at:

```text
/home/admin/minecraft/server-icon.png
```

If the icon is stored on your Windows PC, open PowerShell in the folder containing the image.

Copy it to the Pi:

```powershell
scp server-icon.png admin@192.168.10.2:/home/admin/minecraft/server-icon.png
```

If your Pi has a different IP address, replace:

```text
192.168.10.2
```

with your Pi's address.

Back in Raspberry Pi Connect, check the image:

```bash
ls -lh ~/minecraft/server-icon.png
```

The filename should be:

```text
server-icon.png
```

---

# 17. Make Minecraft start automatically

Create a systemd service:

```bash
sudo nano /etc/systemd/system/minecraft.service
```

Add:

```ini
[Unit]
Description=Minecraft Paper Server
After=network.target

[Service]
User=admin
WorkingDirectory=/home/admin/minecraft
ExecStart=/usr/lib/jvm/java-21-openjdk-arm64/bin/java -Xms2G -Xmx4G -jar paper.jar --nogui
Restart=on-failure
RestartSec=10
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```

> [!IMPORTANT]
> If the Java command earlier returned a different Java path, use that path in `ExecStart`.

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable automatic startup:

```bash
sudo systemctl enable minecraft
```

Start Minecraft:

```bash
sudo systemctl start minecraft
```

Check it:

```bash
sudo systemctl status minecraft
```

Quick check:

```bash
systemctl is-active minecraft
```

It should return:

```text
active
```

Check that Minecraft is listening on port `25565`:

```bash
ss -ltn | grep 25565
```

View live Minecraft logs:

```bash
sudo journalctl -u minecraft -f
```

Press:

```text
Ctrl+C
```

to exit the log viewer.

---

# 18. Check the customized server

Open Minecraft Java Edition.

Go to:

```text
Multiplayer
```

Add the server using its local IP address first.

For the final PiCraftSMP Wi-Fi network this was:

```text
192.168.10.2:25565
```

Refresh the server list.

You should now see:

```text
PiCraftSMP
SURVIVAL • 1.21.11
```

together with the custom `server-icon.png`.

---

# 19. Install Playit

PiCraftSMP uses **Playit** to give Minecraft players a public connection without requiring traditional router port forwarding.

Import the Playit key:

```bash
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/playit.gpg
```

Configure the package repository:

```bash
sudo chmod 0644 /usr/share/keyrings/playit.gpg
```

```bash
sudo curl -fsSL -o /etc/apt/sources.list.d/playit.list https://packages.playit.gg/repo-files/playit-debian.list
```

Update:

```bash
sudo apt update
```

Install Playit:

```bash
sudo apt install playit -y
```

Run:

```bash
playit setup
```

Playit will provide a claim link.

Open the claim link in your browser and connect the Raspberry Pi agent to your Playit account.

> [!CAUTION]
> Never publish the Playit claim URL or authentication token on GitHub.

Create a Minecraft Java tunnel that points to:

```text
127.0.0.1:25565
```

Useful Playit commands:

```bash
sudo systemctl start playit
```

```bash
sudo systemctl stop playit
```

```bash
sudo systemctl restart playit
```

```bash
sudo systemctl status playit
```

```bash
sudo journalctl -u playit -f
```

The live PiCraftSMP Playit hostname is intentionally not included in this repository.

---

# 20. Install Port Warp

Port Warp provides an **additional public route** to the same Minecraft server.

It is not required to use a Hong Kong route.

For the original PiCraftSMP setup:

```text
Minecraft server location: China
Playit route: Japan
Port Warp route: Hong Kong
```

The Hong Kong route was selected because of the location and networking requirements of this particular server.

If you recreate PiCraftSMP somewhere else, choose whichever Port Warp location provides the best connection for you and your players.

Both public routes ultimately connect to the same local Minecraft server:

```text
127.0.0.1:25565
```

Install Port Warp:

```bash
curl -fsSL https://portwarp.com/install | bash
```

The original PiCraftSMP installation reported:

```text
Port Warp v0.3.7
```

at:

```text
/usr/local/bin/pwrp
```

Log in:

```bash
pwrp login
```

Approve the login in your browser.

List tunnels:

```bash
pwrp tunnels
```

Configure the Minecraft tunnel using:

```text
Name: PiCraftSMP
Protocol: TCP
Local host: 127.0.0.1
Local port: 25565
```

Connect:

```bash
pwrp connect
```

Check it:

```bash
pwrp ps --once
```

Configure automatic reconnection:

```bash
pwrp connect --all --save --detach
```

Enable the Port Warp service:

```bash
sudo pwrp service enable
```

Allow the `admin` user service to remain running without an interactive login:

```bash
sudo loginctl enable-linger admin
```

The live Port Warp hostname and public port are intentionally not included in the repository.

---

# 21. Build the PiCraft dashboard

PiCraftSMP has a custom web dashboard for monitoring and controlling the server.

The dashboard displays information including:

- Minecraft status
- Online players
- Raspberry Pi temperature
- CPU usage
- RAM usage
- Storage
- Network usage
- Minecraft latency
- Playit status
- Port Warp status
- Historical metrics
- Server uptime

It also provides administration controls.

The dashboard runs on:

```text
Port 8080
```

Install Python tools:

```bash
sudo apt update
```

```bash
sudo apt install python3-venv python3-pip -y
```

Create the dashboard folder:

```bash
mkdir -p ~/picraft-dashboard
```

Enter it:

```bash
cd ~/picraft-dashboard
```

Create the virtual environment:

```bash
python3 -m venv venv
```

Install the Python packages:

```bash
venv/bin/pip install flask psutil mcstatus
```

Test them:

```bash
venv/bin/python -c "import flask, psutil, mcstatus; print('PiCraft Dashboard ready')"
```

Install Gunicorn:

```bash
venv/bin/pip install gunicorn
```

The dashboard application is stored at:

```text
/home/admin/picraft-dashboard/app.py
```

---

# 22. Give the dashboard system permissions

The dashboard needs permission to perform specific server-management tasks.

Check the `systemctl` location:

```bash
command -v systemctl
```

The PiCraftSMP Raspberry Pi returned:

```text
/usr/bin/systemctl
```

Create the dashboard sudo rules:

```bash
sudo tee /etc/sudoers.d/picraft-dashboard > /dev/null <<'EOF'
admin ALL=(root) NOPASSWD: /usr/bin/systemctl start minecraft
admin ALL=(root) NOPASSWD: /usr/bin/systemctl stop minecraft
admin ALL=(root) NOPASSWD: /usr/bin/systemctl restart minecraft
admin ALL=(root) NOPASSWD: /usr/bin/systemctl restart playit
admin ALL=(root) NOPASSWD: /usr/bin/systemctl reboot
admin ALL=(root) NOPASSWD: /usr/bin/systemctl poweroff
EOF
```

Set secure permissions:

```bash
sudo chmod 440 /etc/sudoers.d/picraft-dashboard
```

Validate the file:

```bash
sudo visudo -cf /etc/sudoers.d/picraft-dashboard
```

It should report:

```text
/etc/sudoers.d/picraft-dashboard: parsed OK
```

---

# 23. Make the dashboard start automatically

Create:

```bash
sudo nano /etc/systemd/system/picraft-dashboard.service
```

Add:

```ini
[Unit]
Description=PiCraftSMP Dashboard
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=admin
WorkingDirectory=/home/admin/picraft-dashboard
Environment=DASH_PASSWORD=<DASHBOARD_PASSWORD>
ExecStart=/home/admin/picraft-dashboard/venv/bin/gunicorn --bind 0.0.0.0:8080 --workers 2 app:app
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Replace:

```text
<DASHBOARD_PASSWORD>
```

with your dashboard password.

> [!CAUTION]
> Never put the real dashboard password in this GitHub repository.

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable and start the dashboard:

```bash
sudo systemctl enable --now picraft-dashboard
```

Check it:

```bash
sudo systemctl status picraft-dashboard --no-pager
```

On the original PiCraftSMP home network, the dashboard is available at:

```text
http://192.168.10.2:8080
```

Your Pi may have a different IP.

Check with:

```bash
hostname -I
```

Port `8080` is deliberately **not publicly exposed through Playit or Port Warp**.

---

# 24. Persistent dashboard metrics

PiCraftSMP stores historical dashboard metrics in:

```text
/home/admin/picraft-dashboard/history.db
```

The collector application is:

```text
/home/admin/picraft-dashboard/collector.py
```

It records information including:

- Temperature
- CPU
- RAM
- Storage
- Upload/download
- Players
- Minecraft query latency
- Playit latency
- Port Warp latency

Create the collector service:

```bash
sudo tee /etc/systemd/system/picraft-collector.service > /dev/null <<'EOF'
[Unit]
Description=PiCraftSMP Metrics Collector
After=network-online.target minecraft.service
Wants=network-online.target

[Service]
Type=simple
User=admin
WorkingDirectory=/home/admin/picraft-dashboard
ExecStart=/home/admin/picraft-dashboard/venv/bin/python /home/admin/picraft-dashboard/collector.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable it:

```bash
sudo systemctl enable --now picraft-collector
```

Check it:

```bash
sudo systemctl status picraft-collector --no-pager
```

---

# 25. Install Tailscale

Raspberry Pi Connect provides convenient browser-based server administration.

PiCraftSMP also uses **Tailscale** for private networking.

Tailscale is for the server administrator, not normal Minecraft players.

It allows private access to things such as:

- PiCraft dashboard
- SSH
- Raspberry Pi network services

Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Sign in:

```bash
sudo tailscale up
```

Open the authentication URL.

Approve the Raspberry Pi.

> [!CAUTION]
> Do not publish Tailscale authentication URLs or keys.

Check the private Tailscale IP:

```bash
tailscale ip -4
```

It will normally look similar to:

```text
100.x.x.x
```

The dashboard can then be opened remotely at:

```text
http://<TAILSCALE_IP>:8080
```

On the home network, the original PiCraftSMP dashboard is:

```text
http://192.168.10.2:8080
```

---

# 26. Create the backup folder

PiCraftSMP creates a world backup every 48 hours.

Create the backup directory:

```bash
mkdir -p /home/admin/minecraft-backups
```

Check it:

```bash
ls -ld /home/admin/minecraft-backups
```

---

# 27. Create the Minecraft backup script

Create the backup script:

```bash
sudo tee /usr/local/sbin/picraft-backup.sh > /dev/null <<'EOF'
#!/bin/bash
set -Eeuo pipefail

MC_DIR="/home/admin/minecraft"
BACKUP_DIR="/home/admin/minecraft-backups"
STAMP="$(date +%Y%m%d-%H%M%S)"
BACKUP="$BACKUP_DIR/PiCraft-$STAMP.tar.gz"

WAS_ACTIVE=0

mkdir -p "$BACKUP_DIR"

if systemctl is-active --quiet minecraft; then
    WAS_ACTIVE=1
    echo "Stopping Minecraft..."
    systemctl stop minecraft
fi

restart_minecraft() {
    if [ "$WAS_ACTIVE" -eq 1 ]; then
        echo "Restarting Minecraft..."
        systemctl start minecraft || true
    fi
}

trap restart_minecraft EXIT

echo "Creating backup..."

tar -C "$MC_DIR" -czf "$BACKUP" \
    world \
    world_nether \
    world_the_end

sha256sum "$BACKUP" > "$BACKUP.sha256"

chown admin:admin "$BACKUP" "$BACKUP.sha256"

trap - EXIT
restart_minecraft

echo
echo "Backup complete:"
echo "$BACKUP"
EOF
```

Make it executable:

```bash
sudo chmod +x /usr/local/sbin/picraft-backup.sh
```

Test it:

```bash
sudo /usr/local/sbin/picraft-backup.sh
```

Check:

```bash
ls -lh ~/minecraft-backups
```

A successful backup creates two files similar to:

```text
PiCraft-YYYYMMDD-HHMMSS.tar.gz
PiCraft-YYYYMMDD-HHMMSS.tar.gz.sha256
```

The `.tar.gz` contains the Minecraft worlds.

The `.sha256` file is used to verify that the backup was copied correctly.

---

# 28. Make backups run every 48 hours

Create:

```bash
sudo tee /usr/local/sbin/picraft-backup-if-due.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail

STAMP_DIR="/var/lib/picraft-backup"
STAMP_FILE="$STAMP_DIR/last-success"
INTERVAL=172800

mkdir -p "$STAMP_DIR"

NOW=$(date +%s)

if [ -f "$STAMP_FILE" ]; then
    LAST=$(cat "$STAMP_FILE")
    AGE=$((NOW - LAST))

    if [ "$AGE" -lt "$INTERVAL" ]; then
        exit 0
    fi
fi

if /usr/local/sbin/picraft-backup.sh; then
    date +%s > "$STAMP_FILE"
fi
EOF
```

Make it executable:

```bash
sudo chmod +x /usr/local/sbin/picraft-backup-if-due.sh
```

Initialize the timer after a successful backup:

```bash
sudo mkdir -p /var/lib/picraft-backup
```

```bash
date +%s | sudo tee /var/lib/picraft-backup/last-success
```

---

# 29. Create the automatic backup timer

Create the backup service:

```bash
sudo tee /etc/systemd/system/picraft-backup.service > /dev/null <<'EOF'
[Unit]
Description=PiCraftSMP Backup Check
After=minecraft.service

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/picraft-backup-if-due.sh
EOF
```

Create the timer:

```bash
sudo tee /etc/systemd/system/picraft-backup.timer > /dev/null <<'EOF'
[Unit]
Description=Check PiCraftSMP Backup Every Hour

[Timer]
OnBootSec=5min
OnUnitActiveSec=1h
Persistent=true

[Install]
WantedBy=timers.target
EOF
```

Reload:

```bash
sudo systemctl daemon-reload
```

Enable:

```bash
sudo systemctl enable --now picraft-backup.timer
```

Check:

```bash
systemctl status picraft-backup.timer --no-pager
```

The system checks every hour.

However, a new backup is only created after:

```text
172800 seconds
```

which is:

```text
48 hours
```

since the previous successful backup.

---

# 30. Create the Windows backup folder

The Windows PC keeps verified copies of the Minecraft backups.

The original location is:

```text
C:\Users\<YOUR_USER>\Documents\PiCraft Backups
```

In Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\Documents\PiCraft Backups"
```

---

# 31. Create a dedicated backup SSH key

Run this in **Windows PowerShell**:

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\picraft_backup_ed25519" -C "PiCraft automatic backup"
```

This creates a dedicated SSH key for the automatic backup system.

Copy **only the public key** to the Raspberry Pi:

```powershell
Get-Content "$env:USERPROFILE\.ssh\picraft_backup_ed25519.pub" | ssh admin@192.168.10.2 "umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

Test it:

```powershell
ssh -i "$env:USERPROFILE\.ssh\picraft_backup_ed25519" admin@192.168.10.2 "echo AUTOMATIC_BACKUP_SSH_WORKS"
```

It should return:

```text
AUTOMATIC_BACKUP_SSH_WORKS
```

> [!CAUTION]
> Never upload this file to GitHub:
>
> ```text
> picraft_backup_ed25519
> ```
>
> It is the private SSH key.

---

# 32. Create the Windows backup sync script

The sync script is stored at:

```text
%USERPROFILE%\Documents\PiCraft-Backup-Sync.ps1
```

Run the following in Windows PowerShell:

```powershell
@'
$ErrorActionPreference = "Stop"

$Pi = "admin@192.168.10.2"
$Key = "$env:USERPROFILE\.ssh\picraft_backup_ed25519"
$RemoteDir = "/home/admin/minecraft-backups"
$LocalDir = "$env:USERPROFILE\Documents\PiCraft Backups"

New-Item -ItemType Directory -Force -Path $LocalDir | Out-Null

# Check whether the Pi is reachable.
$Files = & ssh -i $Key -o BatchMode=yes -o ConnectTimeout=5 $Pi `
    "find $RemoteDir -maxdepth 1 -type f -name 'PiCraft-*.tar.gz' -printf '%f\n'" 2>$null

if ($LASTEXITCODE -ne 0) {
    Write-Host "Pi not reachable. Skipping download."
    $Files = @()
}

foreach ($File in $Files) {

    if ([string]::IsNullOrWhiteSpace($File)) {
        continue
    }

    $File = $File.Trim()
    $ChecksumFile = "$File.sha256"

    $LocalBackup = Join-Path $LocalDir $File
    $LocalChecksum = Join-Path $LocalDir $ChecksumFile

    Write-Host "Copying $File..."

    & scp -i $Key -o BatchMode=yes `
        "${Pi}:${RemoteDir}/${File}" `
        "$LocalBackup"

    if ($LASTEXITCODE -ne 0) {
        Write-Host "Backup copy failed. Pi copy will NOT be deleted."
        continue
    }

    & scp -i $Key -o BatchMode=yes `
        "${Pi}:${RemoteDir}/${ChecksumFile}" `
        "$LocalChecksum"

    if ($LASTEXITCODE -ne 0) {
        Write-Host "Checksum copy failed. Pi copy will NOT be deleted."
        continue
    }

    $Expected = ((Get-Content $LocalChecksum -Raw).Trim() -split "\s+")[0].ToUpper()
    $Actual = (Get-FileHash $LocalBackup -Algorithm SHA256).Hash.ToUpper()

    if ($Expected -eq $Actual) {

        Write-Host "Verified OK: $File"

        & ssh -i $Key -o BatchMode=yes $Pi `
            "rm -f '$RemoteDir/$File' '$RemoteDir/$ChecksumFile'"

        if ($LASTEXITCODE -eq 0) {
            Write-Host "Verified backup removed from Pi."
        }

    } else {

        Write-Host "CHECKSUM FAILED — Pi backup kept."

        Remove-Item $LocalBackup -Force -ErrorAction SilentlyContinue
        Remove-Item $LocalChecksum -Force -ErrorAction SilentlyContinue
    }
}

# Delete laptop backups older than 14 days.
$Cutoff = (Get-Date).AddDays(-14)

Get-ChildItem $LocalDir -File |
    Where-Object {
        $_.Name -like "PiCraft-*" -and
        $_.LastWriteTime -lt $Cutoff
    } |
    Remove-Item -Force

Write-Host "PiCraft backup sync complete."
'@ | Set-Content "$env:USERPROFILE\Documents\PiCraft-Backup-Sync.ps1"
```

Test it manually:

```powershell
powershell -ExecutionPolicy Bypass -File "$env:USERPROFILE\Documents\PiCraft-Backup-Sync.ps1"
```

Check the Windows backup folder:

```powershell
Get-ChildItem "$env:USERPROFILE\Documents\PiCraft Backups"
```

The process is:

```text
Raspberry Pi creates backup
            ↓
Windows copies backup
            ↓
Windows copies SHA-256 file
            ↓
Windows verifies SHA-256
            ↓
If verification succeeds
            ↓
Pi copy is removed
            ↓
Windows keeps backup for 14 days
```

If verification fails, the backup remains on the Raspberry Pi.

---

# 33. Automatically sync backups to Windows

The Windows PC checks the Raspberry Pi every 30 minutes.

Create the scheduled task in Windows PowerShell:

```powershell
schtasks /Create /SC MINUTE /MO 30 /TN "PiCraft Backup Sync" /TR "powershell.exe -NoProfile -ExecutionPolicy Bypass -File `"$env:USERPROFILE\Documents\PiCraft-Backup-Sync.ps1`"" /F
```

Check it:

```powershell
schtasks /Query /TN "PiCraft Backup Sync"
```

If the laptop is turned off or away from the home network, nothing is lost.

The Raspberry Pi keeps the backup until the Windows PC successfully copies and verifies it.

---

# 34. Backup retention

The backup system is designed for:

```text
Backup interval: 48 hours
Windows retention: 14 days
```

Windows automatically removes PiCraft backup files older than 14 days.

---

# 35. Final checks

## Minecraft

```bash
systemctl is-active minecraft
```

Expected:

```text
active
```

## Playit

```bash
systemctl is-active playit
```

Expected:

```text
active
```

## Port Warp

```bash
pwrp ps --once
```

## Dashboard

```bash
systemctl is-active picraft-dashboard
```

Expected:

```text
active
```

## Metrics collector

```bash
systemctl is-active picraft-collector
```

Expected:

```text
active
```

## Backup timer

```bash
systemctl is-active picraft-backup.timer
```

Expected:

```text
active
```

## Tailscale

```bash
tailscale status
```

## Raspberry Pi temperature

```bash
vcgencmd measure_temp
```

## RAM

```bash
free -h
```

## Storage

```bash
df -h /
```

## IP address

```bash
hostname -I
```

---

# 36. Useful restart commands

Restart Minecraft:

```bash
sudo systemctl restart minecraft
```

Restart Playit:

```bash
sudo systemctl restart playit
```

Restart the dashboard:

```bash
sudo systemctl restart picraft-dashboard
```

Restart the metrics collector:

```bash
sudo systemctl restart picraft-collector
```

Reboot the Raspberry Pi:

```bash
sudo reboot
```

Safely shut down the Raspberry Pi:

```bash
sudo poweroff
```

> [!CAUTION]
> Do not simply unplug the Raspberry Pi while it is running. Shut it down first to reduce the risk of corrupting the microSD card.

---

# 37. Security

Never commit any of the following to a public GitHub repository:

- Raspberry Pi password
- Raspberry Pi Connect auth keys
- Raspberry Pi ID credentials
- Dashboard password
- Playit claim links
- Playit authentication tokens
- Port Warp authentication credentials
- Tailscale authentication URLs
- Tailscale authentication keys
- SSH private keys
- Backup SSH private key
- `.env` files containing secrets
- Live public Minecraft addresses unless you deliberately want to advertise the server

The public Minecraft tunnels are for players.

The PiCraft dashboard on port `8080` should remain private and be accessed:

- From the local network
- Through Tailscale

Raspberry Pi administration can be performed using:

- Raspberry Pi Connect
- SSH
- Tailscale

---

# Finished

At this point PiCraftSMP has:

- A Paper Minecraft Java server
- Automatic Minecraft startup
- Custom server icon
- Custom server-list MOTD
- Raspberry Pi Connect administration
- SSH backup administration
- Playit public connectivity
- An optional second public route using Port Warp
- Private Tailscale networking
- A custom monitoring dashboard
- Historical metrics
- Automatic 48-hour world backups
- SHA-256 backup verification
- Automatic Windows backup syncing
- 14-day Windows backup retention

The Raspberry Pi can now operate as an always-on PiCraftSMP Minecraft server.
