# Paperless-ngx (Docker Compose)

A minimal, batteries-included Docker Compose setup for **[paperless-ngx](https://github.com/paperless-ngx/paperless-ngx)** — a self-hosted document management system that OCRs your scans and makes them searchable.

---

## What this stack does

This Compose file spins up:

- **Paperless-ngx (web app)** – the UI + background workers (OCR, indexing).
- **PostgreSQL** – database for metadata.
- **Redis** – message broker for background jobs.

Persistent volumes keep your data, media (uploaded PDFs/images), and any import/export folders across restarts.

---

## Quick start

From the directory that contains `docker-compose.yml`:

```bash
# 1) Pull the images (recommended)
docker compose pull

# 2) Start everything in the background
docker compose up -d

# 3) Watch logs (optional)
docker compose logs -f
```

Once the containers are healthy, open the app:

```
http://localhost:8000
```

> If your compose file maps a different host port, use that instead.

### Create an admin user (first run)

If an admin wasn’t auto-created via env vars, create one interactively:

```bash
docker compose exec webserver   python manage.py createsuperuser
```

Then log in with those credentials.

---

## Usage examples

### 1) Upload via the web UI
- Go to the Paperless-ngx web page and sign in.
- Click **Upload** and drop one or more PDFs or images.
- Paperless will OCR and index them automatically. Use search, tags, correspondents, and document types to organize.

### 2) Drop files into a “consume” (watch) folder
If your compose file mounts a host folder to the app’s **consume** path, you can just copy files there to auto-import.

Example (adjust the path to match your volumes):

```bash
# Copy a PDF into the watch folder to import
cp ~/Scans/invoice.pdf ./consume/
```

Paperless will pick it up, OCR it, and (by default) remove it from the consume folder after import.

### 3) Bulk import an existing directory
If you have a big batch of old scans, place them into the mapped **consume** directory. They’ll be processed in the background. You can keep working while the queue runs.

### 4) Export/download documents
You can download individual documents from the UI.  
If you’ve mapped an **export** directory in your volumes, Paperless can write bulk exports there for easy retrieval on the host.

### 5) Organize and find everything fast
- Add **tags** (e.g., `Tax`, `Insurance`, `Receipts`).
- Set **correspondents** (e.g., `Bank`, `Landlord`).
- Assign **document types** (e.g., `Invoice`, `Statement`).
- Use the global **search bar** to query OCR’d text and metadata.

---

## Common maintenance

```bash
# Stop the stack
docker compose down

# Update to the latest images
docker compose pull
docker compose up -d

# View logs
docker compose logs -f webserver
docker compose logs -f db
docker compose logs -f redis

# Shell into the app container (for advanced commands)
docker compose exec webserver bash
```

---

## Notes

- **Ports & paths:** If your host port or volume mount paths differ from the examples above, use whatever is defined in your `docker-compose.yml`.
- **First boot:** The first start may take a bit while Paperless runs DB migrations and warms up the workers.
- **Backups:** Back up your database volume and the media/data volumes regularly (exports are optional but handy).
