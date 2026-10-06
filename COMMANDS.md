# PiCraftSMP Commands

Quick command reference for managing PiCraftSMP.

Most Raspberry Pi commands can be run from the **Raspberry Pi Connect Remote Shell** or through SSH.

> [!NOTE]
> Commands labelled `bash` run on the Raspberry Pi.
> Commands labelled `powershell` run on the Windows PC.

---

## Minecraft Server

### Check Minecraft status

```bash
systemctl status minecraft
```

Quick check:

```bash
systemctl is-active minecraft
```

If the server is running, this should return:

```text
active
```

### Start Minecraft

```bash
sudo systemctl start minecraft
```

### Stop Minecraft

```bash
sudo systemctl stop minecraft
```

### Restart Minecraft

```bash
sudo systemctl restart minecraft
```

### View live Minecraft logs

```bash
sudo journalctl -u minecraft -f
```

Press:

```text
Ctrl+C
```

to stop watching the logs.

### View recent Minecraft logs

```bash
sudo journalctl -u minecraft -n 100
```

### Check whether Minecraft port 25565 is listening

```bash
ss -ltn | grep 25565
```

---

## Raspberry Pi System Information

### Check temperature

```bash
vcgencmd measure_temp
```

Example:

```text
temp=31.6'C
```

### Check RAM

```bash
free -h
```

### Check storage

```bash
df -h
```

Check only the main filesystem:

```bash
df -h /
```

### Check IP addresses

```bash
hostname -I
```

### Check system uptime

```bash
uptime
```

### Check CPU architecture

```bash
uname -m
```

On the Raspberry Pi 4 used for PiCraftSMP this should return:

```text
aarch64
```

### Check operating system information

```bash
cat /etc/os-release
```

### Check running processes

```bash
top
```

Press:

```text
q
```

to exit.

---

## Raspberry Pi Power

### Safely reboot the Pi

```bash
sudo reboot
```

### Safely shut down the Pi

```bash
sudo poweroff
```

> [!CAUTION]
> Always shut the Raspberry Pi down properly before unplugging its power. Removing power while the system is writing to the microSD card can corrupt the filesystem.

---

## Java

### Check Java version

```bash
java -version
```

### Find the Java executable

```bash
readlink -f "$(command -v java)"
```

---

## Minecraft Files

### Go to the Minecraft server folder

```bash
cd ~/minecraft
```

### List Minecraft files

```bash
ls -lah
```

### Edit server.properties

```bash
nano ~/minecraft/server.properties
```

### Edit the EULA

```bash
nano ~/minecraft/eula.txt
```

### Check the server icon

```bash
ls -lh ~/minecraft/server-icon.png
```

The custom Minecraft server icon should be:

```text
64 × 64 pixels
PNG format
server-icon.png
```

---

## Playit

### Check Playit status

```bash
systemctl status playit
```

Quick check:

```bash
systemctl is-active playit
```

### Start Playit

```bash
sudo systemctl start playit
```

### Stop Playit

```bash
sudo systemctl stop playit
```

### Restart Playit

```bash
sudo systemctl restart playit
```

### View live Playit logs

```bash
sudo journalctl -u playit -f
```

### Run Playit setup

```bash
playit setup
```



---

## Port Warp

### List Port Warp tunnels

```bash
pwrp tunnels
```

### Connect Port Warp

```bash
pwrp connect
```

### Check Port Warp connection

```bash
pwrp ps --once
```

### Connect all saved tunnels in the background

```bash
pwrp connect --all --save --detach
```

### Enable Port Warp automatic startup

```bash
sudo pwrp service enable
```

### Allow the Port Warp user service to remain active

```bash
sudo loginctl enable-linger admin
```

> [!NOTE]
> PiCraftSMP uses Port Warp through Hong Kong because the server is located in China. Hong Kong is not required for other installations. Choose whatever route works best from your location.

---

## PiCraft Dashboard

### Check dashboard status

```bash
systemctl status picraft-dashboard
```

Quick check:

```bash
systemctl is-active picraft-dashboard
```

### Start the dashboard

```bash
sudo systemctl start picraft-dashboard
```

### Stop the dashboard

```bash
sudo systemctl stop picraft-dashboard
```

### Restart the dashboard

```bash
sudo systemctl restart picraft-dashboard
```

### View dashboard logs

```bash
sudo journalctl -u picraft-dashboard -f
```

### Local dashboard address

For the original PiCraftSMP network:

```text
http://192.168.10.2:8080
```

Your Raspberry Pi may have a different local IP address.

Check it with:

```bash
hostname -I
```

---

## Metrics Collector

### Check collector status

```bash
systemctl status picraft-collector
```

Quick check:

```bash
systemctl is-active picraft-collector
```

### Start collector

```bash
sudo systemctl start picraft-collector
```

### Stop collector

```bash
sudo systemctl stop picraft-collector
```

### Restart collector

```bash
sudo systemctl restart picraft-collector
```

### View collector logs

```bash
sudo journalctl -u picraft-collector -f
```

### Historical metrics database

```text
/home/admin/picraft-dashboard/history.db
```

---

## Tailscale

### Check Tailscale status

```bash
tailscale status
```

### Show the Pi's Tailscale IP

```bash
tailscale ip -4
```

### Connect Tailscale

```bash
sudo tailscale up
```

### Check the Tailscale service

```bash
systemctl status tailscaled
```

The Tailscale IP normally looks similar to:

```text
100.x.x.x
```

The dashboard can then be accessed privately at:

```text
http://<TAILSCALE_IP>:8080
```

---

## Backups

### List backups currently stored on the Pi

```bash
ls -lh ~/minecraft-backups
```

### Create a backup manually

```bash
sudo /usr/local/sbin/picraft-backup.sh
```

### Check backups again

```bash
ls -lh ~/minecraft-backups
```

A completed backup should create files similar to:

```text
PiCraft-YYYYMMDD-HHMMSS.tar.gz
PiCraft-YYYYMMDD-HHMMSS.tar.gz.sha256
```

### Check the automatic backup timer

```bash
systemctl status picraft-backup.timer
```

### Show scheduled timers

```bash
systemctl list-timers
```

### Check whether the backup timer is active

```bash
systemctl is-active picraft-backup.timer
```

### View backup service logs

```bash
sudo journalctl -u picraft-backup.service
```

---

## Windows Backup Sync

These commands run in **Windows PowerShell**.

### Manually run the PiCraft backup sync

```powershell
powershell -ExecutionPolicy Bypass -File "$env:USERPROFILE\Documents\PiCraft-Backup-Sync.ps1"
```

### View downloaded backups

```powershell
Get-ChildItem "$env:USERPROFILE\Documents\PiCraft Backups"
```

### Check the automatic Windows backup task

```powershell
schtasks /Query /TN "PiCraft Backup Sync"
```

The Windows PC checks for new PiCraft backups every 30 minutes.

---

## SSH

### Connect using the Pi hostname

From Windows PowerShell:

```powershell
ssh admin@RasPi-Sever.local
```

### Connect using the Pi's local IP

For the original PiCraftSMP network:

```powershell
ssh admin@192.168.10.2
```

### Test the dedicated automatic-backup SSH key

```powershell
ssh -i "$env:USERPROFILE\.ssh\picraft_backup_ed25519" admin@192.168.10.2 "echo AUTOMATIC_BACKUP_SSH_WORKS"
```



---

## Update Raspberry Pi OS

Update the package list:

```bash
sudo apt update
```

Install available updates:

```bash
sudo apt full-upgrade -y
```

Restart afterward if required:

```bash
sudo reboot
```

---

## Quick Health Check

These are the most useful commands if you just want to make sure everything is working.

### Minecraft

```bash
systemctl is-active minecraft
```

### Playit

```bash
systemctl is-active playit
```

### Port Warp

```bash
pwrp ps --once
```

### Dashboard

```bash
systemctl is-active picraft-dashboard
```

### Metrics collector

```bash
systemctl is-active picraft-collector
```

### Backup timer

```bash
systemctl is-active picraft-backup.timer
```

### Tailscale

```bash
tailscale status
```

### Temperature

```bash
vcgencmd measure_temp
```

### RAM

```bash
free -h
```

### Storage

```bash
df -h /
```

### IP address

```bash
hostname -I
```

---

## Quick Restart Commands

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

Reboot the entire Raspberry Pi:

```bash
sudo reboot
```

---


