# Deploy no Railway

O projeto sobe como **dois serviços** dentro do mesmo projeto Railway, mais um Postgres nativo.

## 1. Postgres

Crie um banco Postgres no projeto (New → Database → PostgreSQL). Ele expõe `${{Postgres.DATABASE_URL}}`.

## 2. Serviço `server`

- **Root Directory**: `server/`
- Build via nixpacks (já configurado em `server/nixpacks.toml` e `server/railway.json`).
- **Start command**: `prisma migrate deploy && npm run prisma:seed && node dist/index.js`
- **Variáveis de ambiente**:
  - `DATABASE_URL = ${{Postgres.DATABASE_URL}}`
  - `JWT_SECRET = <segredo forte e aleatório>`
  - `ODDS_API_KEY = <sua chave da The Odds API>`
  - `NODE_ENV = production`
  - `TZ = America/Sao_Paulo`
  - `CLIENT_URL = https://valuebet-client.up.railway.app` (a URL pública do client)
  - `PORT` — o Railway injeta automaticamente.

Anote a URL pública gerada, ex.: `https://valuebet-server.up.railway.app`.

## 3. Serviço `client`

- **Root Directory**: `client/`
- Build gera o bundle React e serve via nginx (`client/nginx.conf`, SPA fallback).
- **Variável de BUILD** (importante — o Vite injeta em tempo de build, não de runtime):
  - `VITE_API_URL = https://valuebet-server.up.railway.app`

> Se mudar a URL do server depois, é preciso **rebuildar** o client para o novo `VITE_API_URL` valer.

## 4. Ordem recomendada

1. Suba o Postgres.
2. Suba o `server` (roda migrate + seed automaticamente no start).
3. Suba o `client` apontando `VITE_API_URL` para a URL do server.
4. Atualize `CLIENT_URL` no server com a URL final do client (CORS) e faça redeploy do server.

Login inicial (criado pelo seed): `admin@valuebet.com` / `admin123` — **troque a senha / o seed em produção.**
