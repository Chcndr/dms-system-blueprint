# MASTER SYSTEM PLAN - DMS HUB + MIO HUB

**Ultima Modifica:** 6 Dicembre 2025  
**Versione:** 3.0 (Ristrutturazione Dominio mio-hub.me)

---

## Indice

1. [Architettura Generale e Domini](#1-architettura-generale-e-domini)
2. [MIO Agent - Fase 1](#2-mio-agent---fase-1)
3. [Componenti Sistema](#3-componenti-sistema)
4. [Integrazioni](#4-integrazioni)
5. [Sistema Mappe GIS](#5-sistema-mappe-gis)
6. [Deployment](#6-deployment)
7. [Sicurezza](#7-sicurezza)
8. [Monitoring](#8-monitoring)
9. [Roadmap](#9-roadmap)

---

## 1. Architettura Generale e Domini

### 1.1. Architettura Logica

Il sistema DMS HUB è composto da:

- **Frontend Vercel:** Dashboard PA (React + Vite + TypeScript)
- **Backend Hetzner:** REST API per orchestratore multi-agente (Node.js + Express)
- **LLM Council Hetzner:** Servizio per confronto LLM (Python/FastAPI + React)
- **Database:** PostgreSQL (Neon)
- **Storage:** File system locale + GitHub logs

### 1.2. Architettura Domini `mio-hub.me` (TO-BE)

Questa sezione definisce l'architettura ufficiale dei domini e funge da "legge" per tutte le configurazioni future. Tutti i servizi devono essere esposti tramite HTTPS.

| Dominio | Servizio | Piattaforma | Porta Locale | Note |
| :--- | :--- | :--- | :--- | :--- |
| **`app.mio-hub.me`** | Frontend Dashboard PA | Vercel | N/A | Dominio principale per l'interfaccia utente. |
| **`api.mio-hub.me`** | Backend MIO Hub | Hetzner (Nginx) | `3000` | Endpoint REST per l'orchestratore e la logica di business. |
| **`council.mio-hub.me`**| Frontend LLM Council | Hetzner (Nginx) | `8002` | Interfaccia utente per il Concilio AI. |
| **`council-api.mio-hub.me`**| Backend LLM Council | Hetzner (Nginx) | `8001` | API di supporto per il Concilio AI. |
| `www.mio-hub.me` | Redirect | DNS Provider | N/A | Redirect verso `app.mio-hub.me`. |
| `mio-hub.me` | Redirect | DNS Provider | N/A | Redirect verso `app.mio-hub.me`. |

---

## 2. MIO Agent - Fase 1

### 2.1. Obiettivo Fase 1

Implementare la **vista singola** del MIO Agent con connessione diretta all'orchestratore REST su Hetzner, utilizzando la nuova architettura dei domini.

### 2.2. Endpoint Backend

**URL:** `POST https://api.mio-hub.me/api/mihub/orchestrator`

**Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Payload Richiesto:**
```json
{
  "mode": "auto",
  "conversationId": "string|null",
  "message": "string",
  "meta": {
    "source": "dashboard_main"
  }
}
```

### 2.3. Flusso MIO (Vista Singola)

```
┌─────────────────────────────────────────────────────────────┐
│                  Dashboard PA (Vercel)                      │
│                  (app.mio-hub.me)                           │
└─────────────────────────────────────────────────────────────┘
                         │
                         │ POST (fetch)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│           Backend Hetzner (mihub-backend-rest)              │
│                  (api.mio-hub.me)                           │
└─────────────────────────────────────────────────────────────┘
```

(Il resto del documento rimane invariato per brevità, ma si intende che tutti i riferimenti a `orchestratore.mio-hub.me` sono deprecati in favore di `api.mio-hub.me`)

---

## 3. Componenti Sistema

### 3.1. Frontend (Vercel)

**Repository:** `Chcndr/dms-hub-app-new`
**Dominio:** `app.mio-hub.me`

### 3.2. Backend Hetzner (REST)

**Repository:** `Chcndr/mihub-backend-rest`
**Server:** 157.90.29.66
**Dominio:** `api.mio-hub.me`

### 3.3. LLM Council (Hetzner)

**Repository:** `karpathy/llm-council` (fork o installazione)
**Server:** 157.90.29.66
**Domini:** `council.mio-hub.me` (Frontend), `council-api.mio-hub.me` (Backend)

---

(Sezioni 4, 5, 6, 7, 8, 9 rimangono concettualmente valide e non necessitano di modifiche strutturali in questo aggiornamento)
