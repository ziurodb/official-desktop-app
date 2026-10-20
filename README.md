# ziuroDB - Universal Database Admin Dashboard

[![Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=yours&file=render.yaml)

ziuroDB is a modern, full-stack database administration dashboard supporting **MongoDB**, **MySQL**, **PostgreSQL** with a beautiful React UI.

## ✨ Features

- 🔍 Schema explorer & data grid (CRUD: Create/Read/Update/Delete)
- ⚡ SQL/NoSQL query console with history
- 🔐 Role-based auth (JWT + Firebase opt)
- 📊 Dashboard stats & audit logs
- 🔄 Background jobs (BullMQ + Redis)
- 🔒 Encrypted connection strings
- 📱 Responsive UI (Tailwind + shadcn/ui)
- 🖥️ Electron-ready (desktop app capable)

## 🏗️ Tech Stack

- **Backend**: Node.js/Express/TypeScript, Mongoose, mysql2/pg
- **Frontend**: React/Vite/TypeScript, TanStack Router/Query
- **Deploy**: Render (Docker + Static)

## Quick Start (Local)

### Prerequisites

- Node.js 20+
- MongoDB (local/Atlas)
- Redis (optional)

### 1. Clone & Install

```bash
git clone <repo> ziuroDB
cd ziuroDB
```

### Backend

```bash
cd backend
cp .env.example .env  # Edit: MONGODB_URI, JWT_SECRET etc..
npm install
npm run dev
```

Health: http://localhost:5000/api/v1/health

### Frontend

```bash
cd ../frontend
npm install
npm run dev
```

App: http://localhost:5173

## ☁️ Deploy to Render (Recommended)

1. Push to GitHub.
2. Render Dashboard → Connect repo → Deploy (uses `render.yaml`).
3. **Required Secrets** (Dashboard → Environment):
   ```
   MONGODB_URI=mongodb+srv://... (Atlas)
   JWT_SECRET=$(openssl rand -hex 32)
   JWT_REFRESH_SECRET=$(openssl rand -hex 32)
   ENCRYPTION_KEY=$(openssl rand -hex 32)  # CRITICAL: exactly 64 hex chars (32 bytes)
   REDIS_URL=... (opt)
   ```
   **MySQL/PostgreSQL tips**:
   - Add `?sslmode=require` or `ssl=true` to URI (Render/PlanetScale require).
   - SSL enabled. Timeouts 30s. Check Render logs for connect/decrypt errors.
   - Delete & recreate connections after key change (re-encrypts).
4. Backend: ~ziuroDB-backend-xxx.onrender.com
5. Frontend: Auto-linked API_URL.

## 🐳 Docker (Backend)

```bash
docker build -t ziuroDB-backend ./backend
docker run -p 5000:5000 -e MONGODB_URI=... ziuroDB-backend
```

## 🔧 Environment Variables

| Var                          | Required | Desc            | Default       |
| ---------------------------- | -------- | --------------- | ------------- |
| `PORT`                       | No       | HTTP port       | 5000          |
| `MONGODB_URI`/`DATABASE_URL` | Yes      | Audit DB        | -             |
| `JWT_SECRET`                 | Yes      | Auth            | -             |
| `ENCRYPTION_KEY`             | Yes      | Conn strings    | -             |
| `CORS_ORIGIN`                | No       | Allowed origins | Render URL/\* |
| `REDIS_URL`                  | No       | Jobs/cache      | Disabled      |

## 📚 Development

```bash
# Backend
npm run dev  # ts-node
npm run build && npm start  # Prod

# Frontend
npm run dev
npm run build  # dist/ static
```

## 🤝 Contributing

1. Fork & PR.
2. `npm run typecheck` (both).
3. Update tests.

## ⚖️ License

MIT
