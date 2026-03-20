# docmost-selfhosted

A production-ready self-hosted deployment of [Docmost](https://docmost.com/) using Docker Compose.
Docmost is an open-source collaborative wiki and documentation platform — a self-hosted
alternative to Notion, Confluence, and Coda.

---

## Why Docmost

In organizations where teams need a central place to write, share, and maintain knowledge,
Docmost covers a wide range of use cases:

* Internal wikis and knowledge bases
* Technical documentation for engineering teams
* Project documentation and meeting notes
* Onboarding guides and HR documentation
* SOPs and runbooks for operations teams
* Any situation where you need a structured, searchable, collaborative writing environment

The advantage of self-hosting is full control over your data, no per-seat pricing,
and the ability to integrate with your existing infrastructure.

---

## Stack

| Component  | Image                  | Purpose                         |
| ---------- | ---------------------- | ------------------------------- |
| Docmost    | docmost/docmost:latest | Application server              |
| PostgreSQL | postgres:16-alpine     | Primary database                |
| Redis      | redis:7.2-alpine       | Cache and session store         |
| Nginx      | nginx:alpine           | Reverse proxy / TLS termination |

---

## Project Structure

```
docmost-selfhosted/
├── docker-compose.yml
├── .env.example
├── nginx/
│   └── nginx.conf
└── certs/                  # TLS certificates — not committed to git
    ├── fullchain.pem
    └── privkey.pem
```

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/enavid/docmost-selfhosted.git
cd docmost-selfhosted
```

### 2. Configure environment

```bash
cp .env.example .env
```

Open `.env` and fill in:

* `APP_SECRET` — a long random string, generate with `openssl rand -hex 32`
* `DATABASE_URL` — update the password to match `POSTGRES_PASSWORD`
* `POSTGRES_PASSWORD` — strong password for the database
* `APP_URL` — your public domain, e.g. `https://docs.yourdomain.com`
* `MAIL_DRIVER` — set to `smtp` or `postmark`
* SMTP credentials if using email

### 3. Place TLS certificates

```bash
cp /path/to/fullchain.pem ./certs/
cp /path/to/privkey.pem   ./certs/
```

### 4. Configure Nginx

Open `nginx/nginx.conf` and update `server_name` to match your domain.

By default Nginx listens on port `80` (HTTP redirect) and `443` (HTTPS).
If another service is already using these ports on your server, change the
host ports in `docker-compose.yml`:

```yaml
ports:
  - "8090:80"
  - "8453:443"
```

And update the redirect in `nginx/nginx.conf` accordingly:

```nginx
return 301 https://$host:8453$request_uri;
```

### 5. Start

```bash
docker compose up -d
```

Check status:

```bash
docker compose ps
docker compose logs -f docmost
```

---

## Environment Variables

| Variable              | Description                                                |
| --------------------- | ---------------------------------------------------------- |
| `APP_URL`           | Public URL of your Docmost instance                        |
| `APP_SECRET`        | Secret key for signing sessions — must be long and random |
| `DATABASE_URL`      | PostgreSQL connection string                               |
| `POSTGRES_PASSWORD` | Database password                                          |
| `REDIS_URL`         | Redis connection string                                    |
| `MAIL_DRIVER`       | Mail provider —`smtp`or `postmark`                    |
| `SMTP_HOST`         | SMTP server hostname                                       |
| `SMTP_PORT`         | SMTP server port                                           |
| `SMTP_USERNAME`     | SMTP username                                              |
| `SMTP_PASSWORD`     | SMTP password                                              |
| `MAIL_FROM_ADDRESS` | Sender email address                                       |
| `MAIL_FROM_NAME`    | Sender display name                                        |

---

## Security Notes

* Never commit `.env` or any file inside `certs/`
* Use a strong, unique `APP_SECRET` — rotating this will invalidate all sessions
* PostgreSQL is on an internal Docker network and not exposed to the host
* Redis is on an internal Docker network and not exposed to the host
* Only Nginx is exposed externally

---

## Upgrading

```bash
docker compose pull
docker compose up -d
```

Docmost handles database migrations automatically on startup.
