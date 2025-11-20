# **Homelab Reference Architecture**

This repository contains a clean, reproducible reference architecture for a small self-hosted environment built with Docker Compose, Postgres, n8n, and optional NocoDB. Secure ingress is provided through a Cloudflared Tunnel.

The goal is to document a simple but realistic multi-service environment that demonstrates service composition, environment configuration, secure ingress patterns, and operational considerations. It is not intended as a production deployment, but as a reproducible template for experimentation and local workflows.

---

## **Architecture Overview**

### **Services**

* **Postgres 16**
  Primary relational datastore with persistent storage.
* **n8n**
  Workflow automation engine.
* **NocoDB (optional)**
  Lightweight database UI and API layer (disabled by default; untested draft configuration).
* **Cloudflared Tunnel**
  Zero-trust ingress via Cloudflare, avoiding exposed ports.

### **Key Features**

* Isolated Docker network (`svc`)
* Named volumes for persistence (`pg_data`, `n8n_data`, `nocodb_data`)
* Modular service definitions (NocoDB optional)
* Secure ingress through Cloudflare Tunnel
* All secrets and environment-specific values externalized via `.env`
* Real-world operational commands and structure

---

## **Repository Structure**

```
.
├── .env.example
├── .gitignore
├── cloudflared
│   └── config.example.yml
├── docker-compose.yml
├── docs
│   └── server-setup.md
└── README.md
 
```

---

## **Getting Started**

### **1. Clone the Repo**

```bash
git clone https://github.com/<youruser>/homelab-reference-architecture.git
cd homelab-reference-architecture
```

---

### **2. Create Your `.env` File**

Copy the example:

```bash
cp .env.example .env
```

Then edit `.env` with your actual values for Postgres, n8n, and (optional) NocoDB.

Docker Compose loads `.env` automatically.

---

### **3. Cloudflared Tunnel Configuration**

This repo includes a **sanitized** example config:

```
cloudflared/config.example.yml
```

Your real config should live under:

```
./cloudflared/config.yml
./cloudflared/<YOUR_TUNNEL_UUID>.json
```

These files are **ignored** by Git and should never be committed.

---

### **4. Start the Stack**

```bash
docker compose up -d
```

Check logs:

```bash
docker compose logs -f
```

View resolved configuration (with variables substituted):

```bash
docker compose config
```

---

For full host configuration and rebuild notes, see [docs/server-setup.md](docs/server-setup.md)

---

## **Optional: Enable NocoDB**

(Note that the noco configuration is a draft, untested in this setup)

If you want to run NocoDB locally (instead of on your Mac), uncomment the entire `nocodb` block in `docker-compose.yml`:

```yaml
#  nocodb:
#    image: nocodb/nocodb:latest
#    ...
```

Then restart:

```bash
docker compose up -d
```

---

## **Updating Services**

```bash
docker compose pull
docker compose up -d
```

---

## **Stopping the Stack**

```bash
docker compose down
```

Persistent volumes remain intact unless removed manually:

```bash
docker volume ls
docker volume rm <name>
```

---

## **Security Notes**

* Secrets never appear in source control
* `.env` is ignored by default
* Cloudflared credentials are never tracked
* Example configs are sanitized and generic
* You can store private notes/scripts in `./private/` (ignored)

---

## **Why This Repo Exists**

This homelab setup reflects a real environment used for:

* workflow experimentation
* integration prototyping
* testing automation and orchestration patterns
* experimenting with zero-trust ingress
* maintaining a reproducible, versioned infra template

It is intentionally small, readable, and practical — suitable for others to fork or extend for their own personal environments.
