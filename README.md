# Firefly III on Dokploy

Ready-to-use Docker Compose setups for deploying [Firefly III](https://www.firefly-iii.org/) on [Dokploy](https://dokploy.com/). The official Firefly III Docker tutorials use multiple separate `.env` files, which Dokploy doesn't support (as of ersion v0.29.2) — these templates merge them into a single environment variable setup that works seamlessly with Dokploy's UI.

---

## Files

| File | Description |
|---|---|
| `docker-compose.yml` | Core Firefly III (app + db + cron) |
| `docker-compose-importer.yml` | Core + Data Importer (app + db + importer + cron) |
| `.env.example` | All environment variables with descriptions |

Start with `docker-compose.yml` if you just want Firefly III. Use `docker-compose-importer.yml` if you also want the [Data Importer](https://docs.firefly-iii.org/how-to/data-importer/installation/docker/) to import from banks/CSV.

---

## Setup

### 1. Create a new project in Dokploy

In your Dokploy dashboard, create a new **Compose** project and paste in the contents of whichever compose file you want to use.

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

Generate 32-character random strings at [random.org](https://www.random.org/strings/?num=1&len=32&digits=on&upperalpha=on&loweralpha=on&unique=on&format=html&rnd=new).

### 3. Add your domain(s)

In Dokploy, go to the **Domains** tab and add your domain. Set the target port to **`8080`** for each service:

- `app` / `firefly_iii_core` → your main Firefly III domain (port `8080`)
- `importer` / `firefly_iii_importer` → a separate subdomain for the importer (port `8080`) *(importer only)*

Dokploy's Traefik reverse proxy handles SSL and routing automatically — do **not** add a `ports:` mapping in the compose file.

### 4. Deploy

Hit **Deploy** in Dokploy. Once all containers are running, visit your domain to complete setup.

---

## Data Importer setup (extra steps)

After the app is running, the importer needs an access token or OAuth Client ID to talk to Firefly III:

1. Go to `https://your-firefly-domain/profile` → **OAuth** section
2. Either create a **Personal Access Token** and set `FIREFLY_III_ACCESS_TOKEN`, or create an **OAuth Client** and set `FIREFLY_III_CLIENT_ID`
3. Re-deploy (or update the env var in Dokploy and restart the importer container)

> **Note:** `FIREFLY_III_URL` must stay as `http://app:8080` — this is the internal Docker network address. `VANITY_URL` should be your public domain (same as `APP_URL`).

---

## How it differs from the official tutorials

The official Docker guides use multiple env files:
- `.env` for the app and cron services
- `.db.env` for the database
- `.importer.env` for the data importer

Since Dokploy only supports a single environment variable store, these templates:
- Merge all env files into one unified set of variables
- Reuse `DB_USERNAME`, `DB_PASSWORD`, and `DB_DATABASE` across both the `app` and `db` service definitions — no duplication
- Prefix the importer's `LOG_LEVEL` as `IMPORTER_LOG_LEVEL` to avoid collision with the app's `APP_LOG_LEVEL`
- Remove `ports:` bindings so they don't conflict with Dokploy's Traefik

---

## Cron job

The `cron` service runs automatically and calls the Firefly III cron endpoint daily at 3:00 AM in your configured timezone. `STATIC_CRON_TOKEN` must be exactly 32 characters or it will silently fail.

---

## Resources

- [Firefly III Documentation](https://docs.firefly-iii.org/)
- [Firefly III Data Importer Docs](https://docs.firefly-iii.org/how-to/data-importer/installation/docker/)
- [Dokploy Documentation](https://docs.dokploy.com/)
