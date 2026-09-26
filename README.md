# Japanese Mahjong Game Record Management System

A full-stack web app for recording Japanese (Riichi) Mahjong games round by round,
recalculating scores, and comparing players' performance.

**Stack:** React 19 · Vite · Tailwind CSS · Recharts · Node.js · Express · PostgreSQL · Prisma ORM

Built as a course project at Northeastern University, Fall 2025.

## Highlights

- **Real data.** The database is seeded with **386 games (4,150 rounds)** played on
  [Mahjong Soul](https://game.maj-soul.com/) between August and November 2025, across
  8 players and 3 rule sets.
- **Statistics computed in SQL.** A single PostgreSQL query built from CTEs and
  `FILTER` aggregates returns 15 statistics per player: total games, highest and
  lowest score, average rank, busting rate, win rate, deal-in rate, tsumo rate,
  tenpai-at-draw rate, exhaustive-draw rate, open-hand rate, riichi rate, dama rate,
  and average points won and lost, using `NULLIF` to avoid division by zero.
- **Cascading score recalculation.** On the upload form, changing any round's result
  recalculates that round's score changes and passes the new starting scores to every
  later round, so final scores and rankings always match the round history.
- **Atomic imports.** Each game (session, players, rounds, per-round player status) is
  written in one Prisma transaction, so a malformed record never leaves a partial game
  in the database.
- **Auth and roles.** Passwords are hashed with bcrypt, sessions use JWT, and admin-only
  routes are protected by role-based middleware.

## Features

**Players**

- Register, log in, and edit their profile, including whether their stats are public
- Upload a game with round-by-round details
- See personal statistics and charts on the home page
- Browse match history filtered by rule set and final ranking, and open any game's
  full round log
- Search for other players and compare points head to head

**Admins**

- View all accounts and ban or reactivate users
- View any player's games and delete invalid sessions

## Database design

Six tables: `users`, `game_types`, `game_sessions`, `session_players`, `round_records`,
and `round_player_status`. Per-round player state (win, tsumo, deal-in, open hand,
riichi, tenpai, han, fu, score change) is stored at the grain of one player in one round.

![E-R diagram](report/ERD.png)

The relational model is in [report/RelationalModel.png](report/RelationalModel.png),
and the full DDL and data dump in [backend/mahjong_db.sql](backend/mahjong_db.sql).

## API

| Area | Endpoints |
| --- | --- |
| Auth | `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`, `PUT /api/auth/profile` |
| Games | `POST /api/gamesession/upload`, `GET /api/gamesession/detail` |
| Players | `GET /api/user/gamesession`, `GET /api/user/roundplayers`, `GET /api/user/datagrid/:id`, `GET /api/user/search`, `GET /api/user/comparepoints`, `GET /api/user/:id` |
| Rule sets | `GET /api/gametype/list`, `GET /api/gametype/detail` |
| Admin | `GET /api/admin/users`, `PUT /api/admin/user/:id/status`, `DELETE /api/admin/gamesession/:uuid` |

The [course report](report/li_final_report.md) describes each flow in more detail.

## Project structure

```
├── my-app/                 front end (React + Vite + Tailwind CSS)
│   └── src/
│       ├── components/     pages: upload, match history, search, stats, profile
│       ├── admin/          admin views and route guard
│       ├── contexts/       AuthContext (authentication state)
│       ├── services/       api.js (HTTP client)
│       └── model/          E-R diagram and relational model pages
├── backend/                API server (Node.js + Express + Prisma)
│   ├── src/
│   │   ├── routes/         auth, user, gametype, gamesession, admin
│   │   ├── controllers/    request handlers and SQL queries
│   │   ├── middlewares/    JWT auth and admin checks
│   │   └── utils/          token helpers, data import
│   └── prisma/             schema, migrations, seed script
├── dataset/                386 game records (JSON), loaded by the seed script
└── report/                 course report, E-R diagram, relational model
```

## Running locally

Requires Node.js 20.19+ (for Vite 7) and PostgreSQL.

**1. Create the database**

```bash
psql -U postgres -c "CREATE DATABASE mahjong_db;"
```

**2. Start the back end**

```bash
cd backend
npm install
cp .env.eg .env        # then edit DATABASE_URL and JWT_SECRET
npx prisma migrate dev
npx prisma db seed     # rule sets, demo accounts, and the 386 games
npm run dev            # http://localhost:5000
```

`.env` looks like:

```env
DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/mahjong_db"
PORT=5000
JWT_SECRET=your_jwt_secret_key
```

> The project uses Prisma 6.19. It does not work with Prisma 7 or later.

**3. Start the front end**

```bash
cd my-app
npm install
npm run dev            # http://localhost:5173
```

To browse the data directly, run `npx prisma studio --port 5556` from `backend/`, or
import [backend/mahjong_db.sql](backend/mahjong_db.sql) into DBeaver.

### Demo accounts (created by the seed script, local use only)

| Role | Username | Password |
| --- | --- | --- |
| Player | `YuuNecro`, `Eucliwood`, `Hellscythe`, `Inui` | `Password1` |
| Admin | `Admin0` | `Test0000` |

## Credits

Built by Jiacong Li. Course group partner: Dawei Feng.

- Game data: [Mahjong Soul](https://game.maj-soul.com/)
- Statistics layout inspired by [amae-koromo](https://amae-koromo.sapk.ch/)
- Charts: [Recharts](https://recharts.github.io/)
