# KORAXX — API (epk-back)

The Laravel API backing [KORAXX](https://github.com/iheblhbib/epk-front), an Electronic Press Kit (EPK) SaaS platform — build, theme, and share professional press kits for artists, labels, and agencies. Built to run on standard shared cPanel hosting (Apache, PHP, MySQL — no Docker, no Redis, no persistent Node server).

> **This repo used to be the `backend/` folder of a single monorepo.** It's now split into two independent repos: this one (the API) and [`epk-front`](https://github.com/iheblhbib/epk-front) (the React SPA). See [docs/architecture.md](docs/architecture.md) for the reasoning and [docs/cpanel-deployment.md](docs/cpanel-deployment.md) for shipping both sides to production.

This repository is at **Phase 16: cPanel Deployment Preparation** — the full product (EPK builder, public/private sharing, analytics, contacts CRM, team management, admin panel, billing, and a security/testing hardening pass) is built; only Phase 17 (final documentation polish) remains. See [ROADMAP.md](ROADMAP.md) for what's built phase-by-phase.

## Local development setup (Windows)

### 1. Prerequisites

- **[Laragon](https://laragon.org/)** with **PHP 8.2 or 8.3** selected as the active version, and its bundled MySQL running. (This project was verified against PHP 8.3.33 / MySQL 8.4 via Laragon.)
- **Composer 2.x**.

Laragon installs its binaries under `C:\laragon\bin\...` without necessarily adding them to your system `PATH`. If `php -v` or `composer -v` doesn't resolve, either add the relevant `C:\laragon\bin\php\<version>` and `C:\laragon\bin\mysql\<version>\bin` folders to `PATH`, or reference the binaries by their full path.

You'll also want the [`epk-front`](https://github.com/iheblhbib/epk-front) repo checked out alongside this one (as a sibling directory) if you're working on the full app end to end — Node.js is only needed over there.

### 2. Database

Create a dedicated database and a scoped (non-root) user — mirroring how cPanel provisions MySQL accounts:

```sql
CREATE DATABASE epk_dev CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'epk_user'@'localhost' IDENTIFIED BY '<a strong password>';
GRANT ALL PRIVILEGES ON epk_dev.* TO 'epk_user'@'localhost';
FLUSH PRIVILEGES;
```

A second database, `epk_test`, is used by the automated test suite (see `phpunit.xml`) so tests never touch your dev data. Create it the same way and grant the same user access to it.

We deliberately use MySQL for local development rather than SQLite: production is MySQL-only, and the app relies on MySQL-specific features (JSON columns, FULLTEXT search), so local dev should mirror it exactly.

### 3. Install and configure

```bash
composer install
cp .env.example .env
php artisan key:generate
```

Edit `.env` — at minimum set `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` to match what you created above, and:

```
APP_URL=http://localhost:8000
FRONTEND_URL=http://localhost:5173
SANCTUM_STATEFUL_DOMAINS=localhost:5173,127.0.0.1:5173
SESSION_DOMAIN=localhost
SESSION_SECURE_COOKIE=false
```

`MAIL_MAILER=log` is a good default for local dev — verification/invitation emails get written to `storage/logs/laravel.log` (including the link) instead of actually sending, so you don't need a working SMTP account just to test auth flows.

### 4. Run it

```bash
php artisan migrate --seed
php artisan storage:link
php artisan serve
```

The API is now running at `http://localhost:8000`. `storage:link` uses PHP's `symlink()` — on Windows this needs either **Developer Mode** enabled (Settings → Update & Security → For Developers) or an elevated terminal; it works without any special privileges on cPanel/Linux.

The seeder creates a demo login: **demo@koraxx.test** / **password**, already a member of a seeded "KORAXX Demo" workspace (as owner), with a second teammate and one pending invitation — so the dashboard isn't empty on first login (once you also have `epk-front` running against this API).

### 5. Verify everything works

```bash
php artisan test        # or vendor/bin/pest
vendor/bin/pint --test  # code style
php artisan route:list  # sanity check
```

## Tech stack

PHP 8.2+, Laravel 12, MySQL, Laravel Sanctum (SPA cookie auth), Eloquent, Policies, database-driven queues and file-based cache (no Redis), Pest for testing.

Full reasoning for these choices — especially the decoupled backend/frontend split and how it maps onto cPanel deployment — is in [docs/architecture.md](docs/architecture.md).

## Documentation

- [ROADMAP.md](ROADMAP.md) — the 17-phase build plan and what's done so far.
- [docs/architecture.md](docs/architecture.md) — system architecture, auth model, authorization design, API conventions.
- [docs/cpanel-deployment.md](docs/cpanel-deployment.md) — step-by-step production deployment: subdomain topology, `.env` setup, SSL, cron, and the redeploy process, for both this repo and `epk-front`.
- [docs/database.md](docs/database.md), [docs/security.md](docs/security.md), [docs/storage.md](docs/storage.md), [docs/custom-domains.md](docs/custom-domains.md), [docs/api.md](docs/api.md), [docs/stripe.md](docs/stripe.md) — deeper reference on specific subsystems.
