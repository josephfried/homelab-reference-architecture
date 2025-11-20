# Homelab Setup Cheat Sheet

A concise, copy‑pasteable guide to rebuild your server quickly. Includes Docker, Compose, Postgres, n8n, Cloudflare Tunnel (named), UFW, and SSH tunnels. Keep this file private; it contains secrets placeholders.

These steps come from my actual homelab setup. I’ve streamlined them for clarity, so adjust as needed based on your host or Ubuntu version.

---

## 0) Base OS prep (fresh Ubuntu)
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y openssh-server
sudo systemctl enable ssh && sudo systemctl start ssh
sudo timedatectl set-timezone America/New_York
```

---

## 1) Headless & power behavior
Disable GUI and keep the machine awake when lid closes.
```bash
sudo systemctl set-default multi-user.target
sudo systemctl isolate multi-user.target
sudo sed -i 's/^#\?HandleLidSwitch.*/HandleLidSwitch=ignore/' /etc/systemd/logind.conf
sudo systemctl restart systemd-logind
```
Console blank after 10 minutes (kernel‑only):
```bash
sudo nano /etc/default/grub
# Add to GRUB_CMDLINE_LINUX_DEFAULT: consoleblank=600
sudo update-grub
sudo reboot
```

---

## 2) Install Docker 

Follow instructions on https://docs.docker.com/engine/install/ubuntu/

Ubuntu’s repository version also works (apt install docker.io docker-compose-v2), but Docker’s official install is recommended.

Add your user to the docker group (so you don’t need sudo every time)
```bash
sudo usermod -aG docker $USER
newgrp docker   # or just log out/in
```

---


## 3) Cloudflare Tunnel (named tunnel with credentials JSON)

> You already have a tunnel named for n8n; reuse it. If you ever need to (re)create on a new box, these are the steps.

### 3a) One-time: create/route (if not already done)
```bash
cloudflared tunnel login
cloudflared tunnel create n8n-tunnel
cloudflared tunnel route dns n8n-tunnel n8n.yourdomain.com
# optional later
# cloudflared tunnel route dns n8n-tunnel nocodb.yourdomain.com
```
This writes a creds file like `~/.cloudflared/<UUID>.json` and sets proxied DNS.

### 3b) Copy creds into the project and write config
```bash
mkdir -p ~/homelab/cloudflared
# copy the actual JSON name you have:
cp ~/.cloudflared/<UUID>.json ~/homelab/cloudflared/
```

Create `~/homelab/cloudflared/config.yml` (see config.example.yml)



**Permissions fix (important):**
```bash
chmod 644 ~/homelab/cloudflared/*.json
chmod 755 ~/homelab/cloudflared
```


---

## 4) UFW (LAN + SSH only)
```bash
sudo apt install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
#sudo ufw allow OpenSSH       # for some distros this might be required
sudo ufw allow from <your-lan-subnet> to any port 5432 proto tcp
sudo ufw allow from <your-lan-subnet> to any port 5678 proto tcp
sudo ufw enable
sudo ufw status verbose
```
> Do not open 80/443; Cloudflare Tunnel is outbound only.
Example for <your-lan-subnet> -> 192.168.100.0/24

---

## 5) Bring it all up
```bash
cd ~/homelab
# set the real 64-hex key in .env first
openssl rand -hex 32  # copy into .env as N8N_ENCRYPTION_KEY

docker compose up -d 
docker compose ps
```
Logs:
```bash
docker compose logs -f cloudflared
# expect: Registered tunnel connection ... protocol=quic

docker compose logs -f n8n
```

Quick checks:
```bash
curl -I https://n8n.yourdomain.com     # expect 200 or 302 with valid cert
```

Open in browser: `https://n8n.yourdomain.com` and complete first-run setup.



---

## 6) Routine ops
```bash
# status
docker compose ps

# logs
docker compose logs -f cloudflared

# upgrade containers
docker compose pull
docker compose up -d

# restart a service
docker compose restart n8n
```

Backups:
```bash
# n8n app data volume
docker run --rm -v n8n_data:/data -v "$PWD":/backup alpine \
  sh -c 'tar czf /backup/n8n_data-$(date +%F).tgz -C / data'

# Postgres (logical dump)
pg_dump -h <server-ip> -U ${PGUSER} -d ${PGDATABASE} > ./pgdump-$(date +%F).sql
```



