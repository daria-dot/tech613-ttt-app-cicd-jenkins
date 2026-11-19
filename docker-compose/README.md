# Tic-tac-toe app + MongoDB with Docker Compose

Two services:
- `db`: the official `mongo:7.0` image, with a volume so the data survives restarts
- `app`: my image `dashaneuro/tech613-tttapp:1.2.0`, which connects to Mongo with `MONGODB_URI=mongodb://db:27017/tictactoe` (`db` is the service name)

The healthcheck makes the app wait until Mongo is ready. Without it, the app can start too early and fall back to browser-only mode.

## 01-manual-seed

```bash
cd 01-manual-seed
docker compose up -d
```
- http://localhost:3000 shows "Mode: Persistent with Mongo DB"
- http://localhost:3000/scoreboard shows "No scores yet"

Seed manually:
```bash
docker compose exec app node seeds/seed.js
```
Refresh `/scoreboard` to see 10 scores (AAA 100 to JJJ 1000).

## 02-auto-seed

Same as 01, plus the app runs the seed script before starting:
```yaml
command: sh -c "node seeds/seed.js && node index.js"
```
```bash
cd 02-auto-seed
docker compose up -d
```
`/scoreboard` shows the 10 scores without running anything.

## Useful commands

```bash
docker compose ps          # db should be (healthy)
docker compose logs app
docker compose down        # stop, keep data
docker compose down -v     # stop and delete data
```

## Findings

- **bindIp:** no change needed. The official mongo image already listens on all interfaces (`--bind_ip_all`).
- **Env var in the container:** `export` inside `docker exec` only affects that shell, not the running app. It has to be set in `environment:` in the compose file.
- **Env var name:** my app uses `MONGODB_URI`, not `DB_HOST`.
