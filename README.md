# Firefly III on Dokploy

A ready-to-use Docker Compose setup for deploying [Firefly III](https://www.firefly-iii.org/) on [Dokploy](https://dokploy.com/). The official Firefly III Docker tutorial uses two separate `.env` files, which Dokploy doesn't support — this template merges them into a single environment variable setup that works seamlessly with Dokploy's UI.

---

## Files

- `docker-compose.yml` — the compose file adapted for Dokploy
- `.env.example` — all required environment variables with descriptions

---

## Setup

### 1. Create a new project in Dokploy

In your Dokploy dashboard, create a new **Compose** project and paste in the contents of `docker-compose.yml`.

### 2. Set your environment variables

Go to the **Environment** tab in Dokploy and paste in the variables from `.env.example`, filling in your own values. At minimum, change these:

| Variable | Description |
|---|---|
| `APP_KEY` | Random string, **exactly 32 characters** |
| `APP_URL` | Your full domain, e.g. `https://firefly.yourdomain.com` |
| `DB_PASSWORD` | A strong password for the database |
| `STATIC_CRON_TOKEN` | Random string, **exactly 32 characters** |
| `SITE_OWNER` | Your email address |
| `TZ` | Your timezone, e.g. `Asia/Dhaka` |

You can generate 32-character random strings at [random.org](https://www.random.org/strings/?num=1&len=32&digits=on&upperalpha=on&loweralpha=on&unique=on&format=html&rnd=new).

### 3. Add a domain

In Dokploy, go to the **Domains** tab and add your domain or subdomain. Set the target port to `8080`. Dokploy's Traefik reverse proxy will handle the rest — do **not** add a `ports:` mapping in the compose file, as port 80/443 is already managed by Dokploy.

### 4. Deploy

Hit **Deploy** in Dokploy. Once all three containers (`firefly_iii_core`, `firefly_iii_db`, `firefly_iii_cron`) are running, visit your domain to complete the setup.

---

## How it differs from the official tutorial

The official [Firefly III Docker guide](https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker/) uses two env files:
- `.env` for the app and cron services
- `.db.env` for the database service

Since Dokploy only supports a single environment variable store, this setup:
- Merges both files into one set of variables
- Uses `${DB_USERNAME}`, `${DB_PASSWORD}`, and `${DB_DATABASE}` in both the `app` and `db` service definitions, so there's no duplication
- Removes the `ports:` binding on the app container so it doesn't conflict with Dokploy's Traefik

---

## Cron job

The `cron` service runs automatically and calls the Firefly III cron endpoint daily at 3:00 AM in your configured timezone. Make sure `STATIC_CRON_TOKEN` is set to exactly 32 characters or the cron job will not work.

---

## Resources

- [Firefly III Documentation](https://docs.firefly-iii.org/)
- [Firefly III Docker Compose Tutorial](https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker/)
- [Dokploy Documentation](https://docs.dokploy.com/)
