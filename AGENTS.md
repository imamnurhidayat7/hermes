# HERMES - Hostinger Managed Hermes Agent

Proyek ini adalah konteks operasional untuk server Hermes Hostinger, bukan source code aplikasi.

## Server
- IP: 72.62.67.171
- Hostname: srv2112819
- Domain: srv2112819.hstgr.cloud (via TRAEFIK_HOST)
- OS: Ubuntu (ESM)
- Tipe: Hostinger VPS Application Catalog - Hermes Agent (Nous Research) + Traefik

## Akses SSH (best practice - sudah setup)
- Jangan pakai `root` untuk agent. Pakai user khusus read-only:
- `ssh hermes` -> user `opencode` @ 72.62.67.171
- Config ada di `~/.ssh/config`:
  ```
  Host hermes
    HostName 72.62.67.171
    User opencode
    IdentityFile ~/.ssh/hermes_opencode
  ```
- Key lokal: `~/.ssh/hermes_opencode` (pubkey `opencode-hermes-read`)
- Test: `ssh hermes "whoami; hostname"`
- User `opencode` TIDAK punya akses docker.sock dan TIDAK bisa baca `/docker/hermes-agent-imru/data` (owner 10000:10000). Ini disengaja least-privilege.

## Layout di server
- `/docker/hermes-agent-imru/`:
  - `docker-compose.yml` -> image `ghcr.io/hostinger/hvps-hermes-agent:latest`, port internal 4860, volume `./data:/opt/data`, env_file `.env`
  - `.env` -> berisi ADMIN_USERNAME, ADMIN_PASSWORD, DASHBOARD_AUTH_SECRET, TRAEFIK_HOST, NEXOS_API_KEY (JANGAN commit / paste ke chat)
  - `.build.log`, `data/` (persistent memory/skills, owner 10000)
- `/docker/traefik/`:
  - traefik:latest, network_mode host, :80 -> redirect :443, letsencrypt httpChallenge, storage `/letsencrypt/acme.json`
  - label di hermes-agent: `traefik.http.routers.${COMPOSE_PROJECT_NAME}.rule=Host(...)`, entrypoints websecure, tls certresolver letsencrypt
- Tidak ada `/var/www`. `find /*hermes*` hanya ketemu `/docker/hermes-agent-imru`.

## Rules untuk agent sesi berikutnya
1. Selalu gunakan `ssh hermes "..."` untuk baca server. Jangan minta password root, jangan pakai BatchMode root.
2. Read-only default: `cat`, `ls -la`, `docker ps` (akan gagal tanpa docker group - itu normal, jangan paksa sudo).
3. Jangan edit `/docker/hermes-agent-imru/*` manual. Update via Hostinger Docker Manager / dashboard.
4. Jangan pernah tulis secret dari `.env` ke repo ini, ke log, atau ke output. Kalau butuh cek, sebut nama variabelnya saja.
5. Repo lokal ini (`/Users/pusdatin06/Documents/SC/hermes`) memang kosong - hanya untuk docs/konteks, bukan clone dari server.
6. Kalau perlu restart/deploy, minta user jalankan via Terminal yang sudah `ssh root`, jangan via key `opencode`.
7. Dashboard web: `https://hermes-agent-imru.srv2112819.hstgr.cloud` (cek TRAEFIK_HOST kalau berubah).

## Perintah berguna
```bash
ssh hermes "hostname; pwd; ls -la /docker/hermes-agent-imru/"
ssh hermes "cat /docker/hermes-agent-imru/docker-compose.yml"
ssh hermes "cat /docker/traefik/docker-compose.yml"
```
Untuk butuh docker status: minta user root jalankan `docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'` dan paste output.
