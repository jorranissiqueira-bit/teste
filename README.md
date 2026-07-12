# ValueBet ⚽💛

Sistema para monitorar odds de futebol, identificar **value bets** comparando casas de apostas, analisar os últimos 5 jogos de cada time para calcular padrões históricos, registrar apostas e acompanhar a performance de tipsters — com histórico de 30 dias. Tema escuro no visual do Mercado Livre.

## Stack

**Backend** — Node.js 20 + TypeScript, Express 4, Prisma 5 + PostgreSQL 16, JWT (jsonwebtoken + bcryptjs), Zod, node-cron (job de odds a cada 15 min + cleanup diário), Pino.

**Frontend** — React 18 + Vite + TypeScript, Tailwind CSS (tema ML: amarelo `#FFE600`, azul `#3483FA`, verde `#00A650`, fundos `#0F0F11`/`#18181B`), React Router v6, Recharts, Lucide React, Axios com interceptor JWT.

**Infra** — Docker Compose (postgres + server + client/nginx) para local; Railway (dois serviços) para produção.

## Estrutura

```
valuebet/
├── docker-compose.yml
├── .env.example
├── RAILWAY_DEPLOY.md
├── server/   # API Express + Prisma
└── client/   # SPA React + Vite
```

## Rodando local (Docker)

```bash
cp .env.example .env          # ajuste ODDS_API_KEY se tiver
docker compose up --build
```

- Frontend: http://localhost:5173
- API: http://localhost:3000
- Login: `admin@valuebet.com` / `admin123`

## Rodando local (sem Docker)

Precisa de um PostgreSQL rodando e `DATABASE_URL` apontando pra ele.

```bash
# backend
cd server
npm install
cp ../.env.example .env
npm run prisma:generate
npm run prisma:migrate
npm run prisma:seed
npm run dev            # http://localhost:3000

# frontend (outro terminal)
cd client
npm install
npm run dev            # http://localhost:5173
```

## Lógica de negócio (resumo)

**Value Bet**

```
edge = (odd_maxima / odd_media - 1) × 100
score_confianca = (edge_normalizado × 0.4) + (taxa_historica × 0.6)
  edge_normalizado mapeia 5%–30% → 0–100%
Ativa quando: edge >= edge_minimo (config) E >= 2 casas com odds para o mercado
```

**Padrões** — 30+ padrões por time nos últimos 5 jogos (Over/Under, BTTS, resultado, handicap asiático, double chance, ofensivo/defensivo, sequências, combinados), com penalidade de confiança para amostra pequena.

**The Odds API**

```
GET /v4/sports/{liga}/odds  → regions=br,eu, markets=h2h,totals,btts, oddsFormat=decimal
GET /v4/sports/soccer/scores?daysFrom=30 → popula HistoricoJogo
```

## Endpoints

```
POST /auth/login | GET /auth/me
GET  /dashboard
GET  /value-bets | GET /value-bets/:id
GET  /partidas
GET  /analise/:partida_id
GET  /padroes | GET /padroes/:partida_id
GET/POST /apostas | PATCH/DELETE /apostas/:id
GET  /tipsters | GET /tipsters/:id/dicas
GET/PUT /config
GET  /health
```

## Deploy

Veja [`RAILWAY_DEPLOY.md`](./RAILWAY_DEPLOY.md).

## Ideias de evolução

Notificações push, Kelly Criterion, websocket de odds em tempo real, export CSV/Excel, backtesting, bot de Telegram, multi-usuário com roles, dados de intervalo.
