# Local Docker Development

This setup runs Postgres, Redis, the Go backend, and the Vue frontend in Docker.
Backend and frontend source directories are bind-mounted, so code changes are
picked up without rebuilding the production image.

## Start

```powershell
cd E:\git_project\sub2api\deploy
Copy-Item .env.dev.example .env.dev
docker compose --env-file .env.dev -f docker-compose.dev-hot.yml up --build
```

Open:

- Frontend: http://localhost:3000
- Backend health: http://localhost:8080/health
- Admin login: `admin@sub2api.local` / `sub2api_dev_admin`

The first backend start uses `AUTO_SETUP=true` to create `config.yaml`, apply
database migrations, and create the admin user.

## Common Commands

```powershell
# Start in the background
docker compose --env-file .env.dev -f docker-compose.dev-hot.yml up -d --build

# View logs
docker compose --env-file .env.dev -f docker-compose.dev-hot.yml logs -f backend
docker compose --env-file .env.dev -f docker-compose.dev-hot.yml logs -f frontend

# Restart backend after Go changes
docker compose --env-file .env.dev -f docker-compose.dev-hot.yml restart backend

# Stop services, keep data
docker compose --env-file .env.dev -f docker-compose.dev-hot.yml down

# Reset local dev data
docker compose --env-file .env.dev -f docker-compose.dev-hot.yml down -v
Remove-Item -Recurse -Force .\dev_data
```

## Notes

- Frontend requests are proxied from Vite to the backend service inside Docker.
- The frontend container also mounts `docs/` read-only because some Vue
  components import legal markdown files with Vite `?raw`.
- PostgreSQL is exposed on host port `5433`; Redis is exposed on host port `6380`.
- Local runtime files live under `deploy/dev_data/` and are ignored by git.
- The frontend dev container installs with `--lockfile=false` so startup does
  not rewrite `frontend/pnpm-lock.yaml`; after first install, restarts reuse the
  Docker `frontend_node_modules` volume.
- The existing `deploy/docker-compose.dev.yml` still builds the full embedded
  production-style image from local source. Use that when you want to validate
  the release image rather than do daily development.
