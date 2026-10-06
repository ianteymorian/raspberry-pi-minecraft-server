# PiCraftSMP Setup Guide

This documents the actual PiCraftSMP build: a Paper Minecraft server on a Raspberry Pi 4 (8 GB), two public player routes (Playit and Port Warp), a private web dashboard, Tailscale remote administration, and automated off-device backups to Windows.

> [!IMPORTANT]
> This guide is based on the commands used during the original build. Public tunnel addresses, authentication links/tokens, passwords, and SSH private-key contents are intentionally not included. Replace values in `<ANGLE_BRACKETS>` with your own.

## 1. What you need before you start

PiCraftSMP was built with:

- Raspberry Pi 4
- 8 GB RAM (7.64 GB usable shown by the dashboard)
- A microSD card (32 GB or larger is recommended)
- A microSD card reader, or an SD adapter/reader that your computer can use
- A proper Raspberry Pi 4 USB-C power supply
- Active cooling (fan/heatsink)
- Ethernet cable if you want to use wired networking; Wi-Fi also works
- Windows PC for off-device backups
- An Internet connection

You **do not need a monitor, keyboard, or mouse for the Pi** if you follow the headless setup below. We will enable SSH before the Pi ever boots so you can control it from your normal computer.

> [!CAUTION]
> Flashing the microSD card erases everything already on that card. Double-check that you select the correct drive in Raspberry Pi Imager.

## 2. Install Raspberry Pi Imager on your computer

Do these steps on your normal Windows PC, not on the Raspberry Pi.

1. Go to the official Raspberry Pi software page: <https://www.raspberrypi.com/software/>.
2. Download **Raspberry Pi Imager** for Windows.
3. Run the installer and finish the installation.
4. Put the microSD card into your computer's card reader.
5. Open **Raspberry Pi Imager**.

Raspberry Pi Imager is the program that downloads Raspberry Pi OS and writes it to the microSD card.

## 3. Flash Raspberry Pi OS to the microSD card

In Raspberry Pi Imager:

1. Select **Raspberry Pi 4** as the Raspberry Pi device.
2. Select **Raspberry Pi OS Lite (64-bit)** as the operating system.
   - Use the 64-bit version.
   - The **Lite** version is intentional. A Minecraft server does not need a desktop interface.
3. Select your **microSD card** as the storage device.
4. Continue to the OS/customisation settings before writing the card.

Configure the Pi with these settings:

| Setting | PiCraftSMP value / what to enter |
|---|---|
| Hostname | `RasPi-Sever` was used on the original build. `picraftsmp` is a cleaner name if starting again. |
| Username | `admin` |
| Password | Create a strong password. **Do not put it in GitHub.** |
| Wi-Fi SSID | Your Wi-Fi network name, if using Wi-Fi |
| Wi-Fi password | Your Wi-Fi password, if using Wi-Fi |
| Wi-Fi country | The country where the Pi is physically being used |
| Time zone | Your local time zone |
| SSH / remote access | **Enable SSH** and allow password authentication for the initial setup |

If Imager offers **Raspberry Pi Connect**, you may enable it as an additional way to reach the Pi from a browser. SSH is still the main method used in this guide.

Now write the card:

1. Check the selected storage device one more time.
2. Click the button to write/flash the operating system.
3. Accept the warning that the selected microSD card will be erased.
4. Wait while Imager writes and verifies the card. Do not remove it during this process.
5. When Imager says it has finished, safely eject the microSD card from your computer.

You now have a bootable Raspberry Pi OS microSD card.

## 4. Assemble the Raspberry Pi and boot it for the first time

Keep the Raspberry Pi unplugged while connecting everything.

1. Insert the flashed microSD card into the microSD slot on the underside of the Raspberry Pi.
2. Make sure the heatsink/fan or other active cooling is installed correctly.
3. If using Ethernet, connect the Ethernet cable from the Pi to your router or network.
4. If using Wi-Fi, you do not need an Ethernet cable because the Wi-Fi credentials were saved by Imager.
5. Connect the Raspberry Pi power supply last.
6. Wait about 2-3 minutes for the first boot. The first boot can take longer than later boots.

Do not unplug the Pi just because nothing appears on your PC. Raspberry Pi OS Lite is running on the Pi itself; you connect to it over the network.

## 5. Connect to the Raspberry Pi for the first time

On your Windows PC, open **PowerShell** or **Windows Terminal**.

If you used the original hostname, first try:

```powershell
ssh admin@RasPi-Sever.local
```

If you chose `picraftsmp` instead, use:

```powershell
ssh admin@picraftsmp.local
```

The first time you connect, Windows may show a message saying the authenticity of the host cannot be established and ask whether you want to continue. Type:

```text
yes
```

Then enter the password you created in Raspberry Pi Imager. The password will not appear on the screen while you type it; that is normal.

When the connection works, your prompt should look roughly like this:

```text
admin@RasPi-Sever:~ $
```

### If the `.local` hostname does not work

Find the Pi's IP address in your router's connected-device/client list, then connect using the IP address:

```powershell
ssh admin@<PI_IP_ADDRESS>
```

For example, PiCraftSMP's final local Wi-Fi address was:

```powershell
ssh admin@192.168.10.2
```

Your Pi may receive a different IP address. Do not assume yours will be `192.168.10.2`.

## 6. Check the Pi and install the base software

Everything below this point is typed **inside the SSH session on the Raspberry Pi**, unless the guide specifically says to use the Windows PC.

Check the CPU architecture:

```bash
uname -m
```

For this build, it should report:

```text
aarch64
```

Check your IP address:

```bash
hostname -I
```

Check free disk space and memory:

```bash
df -h /
free -h
```

Check the Raspberry Pi temperature:

```bash
vcgencmd measure_temp
```

Update the Pi and install Java plus `wget`:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt install openjdk-25-jre-headless wget -y
java -version
```

The original build initially installed OpenJDK 25. The later working `minecraft.service` used the existing Java 21 ARM64 binary at `/usr/lib/jvm/java-21-openjdk-arm64/bin/java`. Before recreating the service on a different Pi, verify the Java path:

```bash
readlink -f "$(command -v java)"
```

If your path differs, use the path printed by that command in `ExecStart` below.

## 7. Install Paper Minecraft

Create the server directory:

```bash
mkdir ~/minecraft
cd ~/minecraft
```

The Paper build downloaded during the original setup was:

```bash
wget https://fill-data.papermc.io/v1/objects/7b7b3b43c009103e1971a0576c26f655a7dd9b56a0a2a4438e352c03a7fecd08/paper-26.2-123.jar -O paper.jar
```

PiCraftSMP is an 8 GB Pi, so Minecraft was run with 2 GB initial heap and a 4 GB maximum heap:

```bash
java -Xms2G -Xmx4G -jar paper.jar --nogui
```

On the first run Paper creates `eula.txt` and exits. Accept the Minecraft EULA:

```bash
nano eula.txt
```

Change:

```text
eula=false
```

to:

```text
eula=true
```

Save with `Ctrl+O`, press Enter, then exit with `Ctrl+X`.

Start Paper again:

```bash
java -Xms2G -Xmx4G -jar paper.jar --nogui
```

Stop the server safely from its console with:

```text
stop
```

The working server later reported `Paper 1.21.11` in the PiCraft dashboard.

## 8. Configure Paper for automatic startup

Create the systemd service:

```bash
sudo nano /etc/systemd/system/minecraft.service
```

The working PiCraftSMP service was:

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

If Step 6 showed a different Java binary, change only the Java path in `ExecStart`.

Then enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable minecraft
sudo systemctl start minecraft
sudo systemctl status minecraft
```

Useful checks:

```bash
systemctl is-active minecraft
sudo journalctl -u minecraft -f
ss -ltn | grep 25565
```

## 9. Local networking

Check the Pi's addresses:

```bash
hostname -I
```

During the final Wi-Fi setup, PiCraftSMP used:

```text
192.168.10.2:25565
```

An earlier Ethernet configuration used `192.168.1.9`. The local IP will depend on the network, so do not copy these addresses blindly.

SSH example from the final Wi-Fi setup:

```bash
ssh admin@192.168.10.2
```

## 10. Install Playit for the global public route

PiCraftSMP uses Playit as the global/fallback route so players can connect without router port forwarding or installing extra software.

The original setup first imported the Playit key:

```bash
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/playit.gpg
```

The final package-repository setup used:

```bash
sudo chmod 0644 /usr/share/keyrings/playit.gpg
sudo curl -fsSL -o /etc/apt/sources.list.d/playit.list https://packages.playit.gg/repo-files/playit-debian.list
sudo apt update
sudo apt install playit -y
```

Claim the Pi's Playit agent:

```bash
playit setup
```

Open the claim link it prints and connect the agent to your Playit account. Do not put the claim URL/token in a public repository.

Create a Minecraft Java tunnel pointing to:

```text
127.0.0.1:25565
```

The working installation provides a `playit` systemd service. Useful controls are:

```bash
sudo systemctl start playit
sudo systemctl stop playit
sudo systemctl restart playit
sudo systemctl status playit
sudo journalctl -u playit -f
```

The public Playit hostname used by the live server is intentionally omitted from this repository.

## 11. Install Port Warp for the China / Hong Kong route

Port Warp provides a second public route to the same Minecraft server. In PiCraftSMP it is used as the China/Hong Kong route while Playit remains the global fallback.

Install Port Warp on the ARM64 Pi:

```bash
curl -fsSL https://portwarp.com/install | bash
```

The original installation reported Port Warp `v0.3.7` at `/usr/local/bin/pwrp`.

Authenticate the Pi:

```bash
pwrp login
```

Approve the short-code login in a browser, then list the account's tunnels:

```bash
pwrp tunnels
```

Create the Minecraft tunnel with:

```text
Name: PiCraftSMP
Protocol: TCP
Local host: 127.0.0.1
Local port: 25565
```

Connect it:

```bash
pwrp connect
```

To verify a detached tunnel:

```bash
pwrp ps --once
```

PiCraftSMP was then configured to reconnect automatically after reboot:

```bash
pwrp connect --all --save --detach
sudo pwrp service enable
sudo loginctl enable-linger admin
```

The working installation showed a systemd **user** unit named `portwarp.service`, with autostart enabled and all enabled tunnels selected at boot.

The live Port Warp public hostname and port are intentionally omitted from this repository.

## 12. Build the PiCraft dashboard

The dashboard runs privately on port `8080` and provides system metrics, Minecraft status/player information, tunnel status, network information, and server controls.

Install the dashboard tools:

```bash
sudo apt update
sudo apt install python3-venv python3-pip -y
mkdir -p ~/picraft-dashboard
cd ~/picraft-dashboard
python3 -m venv venv
venv/bin/pip install flask psutil mcstatus
```

Verify the Python environment:

```bash
venv/bin/python -c "import flask, psutil, mcstatus; print('PiCraft Dashboard ready')"
```

Install Gunicorn:

```bash
cd ~/picraft-dashboard
venv/bin/pip install gunicorn
```

Place the dashboard application in:

```text
/home/admin/picraft-dashboard/app.py
```

### Give the dashboard only the required system permissions

First verify the systemctl path:

```bash
command -v systemctl
```

The Pi returned `/usr/bin/systemctl`. The dashboard sudoers rules were then created with:

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

Lock and validate the sudoers file:

```bash
sudo chmod 440 /etc/sudoers.d/picraft-dashboard
sudo visudo -cf /etc/sudoers.d/picraft-dashboard
```

It should report:

```text
/etc/sudoers.d/picraft-dashboard: parsed OK
```

### Run the dashboard automatically

The original build temporarily used a simple dashboard password directly in the service. That password is **not** reproduced here. Use your own strong value for `<DASHBOARD_PASSWORD>`.

Create the service:

```bash
sudo nano /etc/systemd/system/picraft-dashboard.service
```

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

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now picraft-dashboard
sudo systemctl status picraft-dashboard --no-pager
```

At home, the dashboard was opened at:

```text
http://192.168.10.2:8080
```

Port `8080` is deliberately **not** exposed through Playit or Port Warp.

## 13. Persistent metrics collection

PiCraftSMP records historical dashboard metrics once per minute in:

```text
/home/admin/picraft-dashboard/history.db
```

The collector tracks temperature, CPU, RAM, storage, upload/download, players, Minecraft query latency, Port Warp latency, and Playit latency.

Place the collector at:

```text
/home/admin/picraft-dashboard/collector.py
```

Create its service:

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

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now picraft-collector
sudo systemctl status picraft-collector --no-pager
```

## 14. Install Tailscale for private remote administration

Tailscale is for the administrator, not Minecraft players. It allows the private dashboard and SSH access to remain off the public Internet.

Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate the Pi:

```bash
sudo tailscale up
```

Open the authentication URL it prints and approve the Pi. Do not publish that URL.

Get the Pi's private Tailscale IPv4 address:

```bash
tailscale ip -4
```

The PiCraft build used a `100.x.x.x` Tailscale address. Access the dashboard remotely with:

```text
http://<TAILSCALE_IP>:8080
```

The local dashboard remains:

```text
http://192.168.10.2:8080
```

## 15. Automated Raspberry Pi backups

Backups are created on the Pi in:

```bash
mkdir -p /home/admin/minecraft-backups
ls -ld /home/admin/minecraft-backups
```

### Backup script

Create `/usr/local/sbin/picraft-backup.sh` exactly as used in the working build:

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

sudo chmod +x /usr/local/sbin/picraft-backup.sh
```

Test it:

```bash
sudo /usr/local/sbin/picraft-backup.sh
ls -lh ~/minecraft-backups
```

The tested PiCraft backup was about 23 MB and produced both a `.tar.gz` archive and a `.sha256` verification file.

### Enforce the 48-hour interval across reboots

Create the due-check script:

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

sudo chmod +x /usr/local/sbin/picraft-backup-if-due.sh
```

Initialize the clock after a successful manual backup:

```bash
sudo mkdir -p /var/lib/picraft-backup
date +%s | sudo tee /var/lib/picraft-backup/last-success
cat /var/lib/picraft-backup/last-success
```

### Create the backup timer

```bash
sudo tee /etc/systemd/system/picraft-backup.service > /dev/null <<'EOF'
[Unit]
Description=PiCraftSMP Backup Check
After=minecraft.service

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/picraft-backup-if-due.sh
EOF

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

Enable and verify it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now picraft-backup.timer
systemctl status picraft-backup.timer --no-pager
```

The timer checks hourly, but `picraft-backup-if-due.sh` only creates a backup after `172800` seconds (48 hours) have elapsed.

## 16. Sync verified backups to Windows

The Windows laptop keeps the off-device copies in:

```text
C:\Users\<YOUR_USER>\Documents\PiCraft Backups
```

### Create a dedicated SSH key

Run in Windows PowerShell:

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\picraft_backup_ed25519" -C "PiCraft automatic backup"
```

Copy **only the public key** to the Pi:

```powershell
Get-Content "$env:USERPROFILE\.ssh\picraft_backup_ed25519.pub" | ssh admin@192.168.10.2 "umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

Test passwordless authentication:

```powershell
ssh -i "$env:USERPROFILE\.ssh\picraft_backup_ed25519" admin@192.168.10.2 "echo AUTOMATIC_BACKUP_SSH_WORKS"
```

Never commit `picraft_backup_ed25519` (the private key) to GitHub.

### Create the sync script

The working script is saved as:

```text
%USERPROFILE%\Documents\PiCraft-Backup-Sync.ps1
```

Its configuration is:

```powershell
$Pi = "admin@192.168.10.2"
$Key = "$env:USERPROFILE\.ssh\picraft_backup_ed25519"
$RemoteDir = "/home/admin/minecraft-backups"
$LocalDir = "$env:USERPROFILE\Documents\PiCraft Backups"
```

The final working script (including the later fix so 14-day cleanup still runs when the Pi is unreachable) is:

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

The script:

1. Finds `PiCraft-*.tar.gz` on the Pi.
2. Copies the archive and matching `.sha256` file with `scp`.
3. Verifies the Windows copy with `Get-FileHash -Algorithm SHA256`.
4. Deletes the Pi copy only after successful verification.
5. Keeps the Pi copy if copying or verification fails.
6. Deletes Windows `PiCraft-*` files older than 14 days.

Test the script manually:

```powershell
powershell -ExecutionPolicy Bypass -File "$env:USERPROFILE\Documents\PiCraft-Backup-Sync.ps1"
```

Check the backup folder:

```powershell
Get-ChildItem "$env:USERPROFILE\Documents\PiCraft Backups"
```

### Run the Windows sync every 30 minutes

Create the scheduled task exactly as used in the working setup:

```powershell
schtasks /Create /SC MINUTE /MO 30 /TN "PiCraft Backup Sync" /TR "powershell.exe -NoProfile -ExecutionPolicy Bypass -File `"$env:USERPROFILE\Documents\PiCraft-Backup-Sync.ps1`"" /F
```

Verify it:

```powershell
schtasks /Query /TN "PiCraft Backup Sync"
```

The final behavior is:

```text
Pi creates backup every 48 hours
          ↓
Windows checks every 30 minutes when the laptop is on
          ↓
archive + SHA-256 file copied
          ↓
Windows verifies SHA-256
          ↓
verified Pi copy removed
          ↓
Windows keeps backups for 14 days
```

If the laptop is off or away from the home network, the backup stays on the Pi until a later sync succeeds.

## 17. Final service checks

Check the important services:

```bash
echo "=== MINECRAFT ==="
systemctl is-active minecraft
echo "=== PLAYIT ==="
systemctl is-active playit
echo "=== DASHBOARD ==="
systemctl is-active picraft-dashboard
echo "=== COLLECTOR ==="
systemctl is-active picraft-collector
echo "=== BACKUP TIMER ==="
systemctl is-active picraft-backup.timer
echo "=== TEMPERATURE ==="
vcgencmd measure_temp
echo "=== MEMORY ==="
free -h
echo "=== IP ==="
hostname -I
```

Port Warp uses a user service, so also check:

```bash
pwrp ps --once
```

## Security notes

Do **not** commit any of the following to a public repository:

- Dashboard passwords
- Playit claim/authentication URLs or tokens
- Port Warp authentication credentials
- Live public server addresses unless you deliberately want to advertise the server
- SSH private keys
- Tailscale authentication URLs or keys
- Any `.env` file containing secrets

The Minecraft player routes may be public, but the dashboard on port `8080` should remain private and accessed locally or over Tailscale.
