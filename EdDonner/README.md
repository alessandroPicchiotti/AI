# Kanban Project Manager

MVP Kanban con backend .NET 8 (API in-memory) e frontend Vue 3 + Vite.

## Avvio

### Backend (porta 5135)

```powershell
cd backend/Kanban.Api
dotnet run --launch-profile http
```

Swagger: http://localhost:5135/swagger

### Frontend (porta 5173)

```powershell
cd frontend
npm install
npm run dev
```

App: http://localhost:5173

Copia `frontend/.env.example` in `frontend/.env` se necessario (`VITE_API_URL=http://localhost:5135`).

## Test

```powershell
cd backend
dotnet test Kanban.slnx

cd ../e2e
npm install
npx playwright install chromium
npm test
```

## Struttura

- `backend/Kanban.Api` - Web API REST
- `backend/Kanban.Api.Tests` - test xUnit sullo store
- `frontend` - SPA Vue 3
- `e2e` - test Playwright

Vedi `AGENTS.md` per requisiti completi.
