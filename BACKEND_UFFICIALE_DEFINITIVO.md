# BACKEND UFFICIALE DEFINITIVO - DMS / MIO-HUB

**Data:** 21 Novembre 2025  
**Autore:** Manus AI  
**Stato:** UFFICIALE - Fonte di Verità Unica

---

## 🎯 DICHIARAZIONE UFFICIALE

### Backend in Produzione (AS-IS)

**Nome:** `mihub-backend-rest`  
**Status:** ✅ **BACKEND UFFICIALE IN PRODUZIONE**

**Dettagli Tecnici:**
- **Path Server:** `/root/mihub-backend-rest/` (Hetzner 157.90.29.66)
- **Repository GitHub:** Nessuno (codice locale, non versionato)
- **Tecnologia:** Node.js 22.13.0 + Express
- **Database:** PostgreSQL (Neon) - `ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech`
- **ORM:** Nessuno (query dirette)
- **Process Manager:** PM2 (`mihub-backend`, PID 294331)
- **Porta:** 3001
- **Dominio:** `orchestratore.mio-hub.me` (Nginx reverse proxy)
- **Uptime:** 34+ minuti (riavviato 21 Nov 18:04)
- **Memoria:** ~67 MB

**Endpoint Attivi:**
- `GET /health` - Health check backend
- `GET /api/dmsHub/*` - Endpoint DMS Hub
- `GET /api/logs/*` - Endpoint logs
- `GET /api/mihub/*` - Endpoint MIO-HUB
- `GET /api/gis/health` - Health check modulo GIS
- `GET /api/gis/market-map` - Dati mappa mercato (Editor v3)

**Modulo GIS:**
- **File Router:** `/root/mihub-backend-rest/routes/gis.js`
- **File Dati:** `/root/mihub-backend-rest/data/editor-v3-sample.json`
- **Status:** ✅ Integrato e funzionante
- **URL Produzione:** https://orchestratore.mio-hub.me/api/gis/market-map

---

## 📊 CONFORMITÀ AL PIANO UFFICIALE

### ✅ Requisiti Rispettati

| Requisito | Stato | Note |
|---|---|---|
| Database PostgreSQL unico | ✅ | Neon PostgreSQL configurato |
| Nessun MySQL/PlanetScale | ✅ | Completamente rimosso |
| Frontend non accede DB direttamente | ✅ | Solo via Core API |
| Core API su Hetzner | ✅ | `orchestratore.mio-hub.me` |
| Modulo GIS integrato | ✅ | Endpoint funzionanti |

### ⚠️ Limitazioni Temporanee

| Limitazione | Impatto | Piano Risoluzione |
|---|---|---|
| Codice non versionato su Git | ⚠️ Medio | Migrazione a monorepo `mihub` |
| Nessun ORM | ⚠️ Basso | Drizzle in monorepo `mihub` |
| Endpoint statici (no `marketId`) | ⚠️ Medio | Endpoint dinamici in v2 |
| Dati hardcoded (3 piazzole) | ⚠️ Basso | Caricare 160 piazzole Grosseto |

---

## 🗂️ REPOSITORY E CODICE

### Backend Ufficiale Attuale

**Path:** `/root/mihub-backend-rest/`  
**Repository GitHub:** ❌ Nessuno (locale)

**Struttura:**
```
/root/mihub-backend-rest/
├── index.js              # Entry point
├── routes/
│   ├── dmsHub.js        # Route DMS Hub
│   ├── logs.js          # Route logs
│   ├── mihub.js         # Route MIO-HUB
│   └── gis.js           # Route GIS (mappa mercato)
├── data/
│   └── editor-v3-sample.json  # Dati mappa (3 piazzole)
├── .env                 # Configurazione (DATABASE_URL, etc.)
└── package.json         # Dipendenze
```

**Dipendenze Principali:**
- `express` - Web framework
- `cors` - CORS middleware
- `pg` - PostgreSQL client
- `dotenv` - Environment variables

### Repository Storici (Non Usati)

**1. Monorepo `mihub`**

**Path Server:** `/root/mihub/`  
**Repository GitHub:** `Chcndr/mihub`  
**Status:** ❌ Non in produzione (errori build TypeScript)

**Motivo Non Usato:**
- Errori compilazione TypeScript
- Dipendenze mancanti (`@vercel/node`, `nanoid`)
- Metodi Drizzle incompatibili con PostgreSQL

**Futuro:**
- ✅ Target per migrazione futura
- ⏳ Richiede fix build TypeScript
- ⏳ Piano migrazione in 4 fasi documentato

**2. Altri Backend**

Nessun altro backend presente sul server.

---

## 🔄 PIANO MIGRAZIONE (Futuro)

### Obiettivo Finale

**Backend Target:** Monorepo `Chcndr/mihub`  
**Tecnologia:** TypeScript + tRPC + Drizzle ORM  
**Status:** ⏳ Pianificato (non urgente)

### Fasi Migrazione

**Fase 1: Fix Build TypeScript** (⏳ Da fare)
- Analisi errori compilazione
- Fix dipendenze mancanti
- Aggiornamento Drizzle per PostgreSQL
- Test build locale

**Fase 2: Port Modulo GIS** (⏳ Da fare)
- Migrare endpoint da REST a tRPC
- Integrazione database PostgreSQL
- Test endpoint tRPC

**Fase 3: Deploy Staging** (⏳ Da fare)
- Deploy su porta diversa (3002)
- Test integrazione frontend
- Monitoring performance

**Fase 4: Migrazione Produzione** (⏳ Da fare)
- Backup backend REST
- Deploy monorepo su porta 3001
- Dismissione backend REST

**Timeline:** Non definita (backend REST funzionante, nessuna urgenza)

---

## 🚫 REPOSITORY DA ARCHIVIARE

### Nessuno

Tutti i repository esistenti hanno uno scopo:
- `mihub-backend-rest` (locale) → Backend ufficiale in produzione
- `Chcndr/mihub` (GitHub) → Target futuro per migrazione

**Nota:** Il backend REST locale dovrebbe essere versionato su Git per sicurezza, ma non è bloccante.

---

## 📝 ACCESSI E CREDENZIALI

### Server Hetzner

**IP:** 157.90.29.66  
**SSH:** `ssh -i /home/ubuntu/.ssh/hetzner_server_key root@157.90.29.66`  
**Password Backup:** tkFxkgnT4ieh

### Database PostgreSQL (Neon)

**Host:** `ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech`  
**Database:** `neondb`  
**User:** `neondb_owner`  
**Password:** `npg_lYG6JQ5Krtsi`  
**Connection String:** Configurata in `/root/mihub-backend-rest/.env`

### PM2 Backend

**Process Name:** `mihub-backend`  
**Working Directory:** `/root/mihub-backend-rest/`  
**Entry Point:** `index.js`  
**Porta:** 3001

**Comandi Utili:**
```bash
# Verificare stato
pm2 list
pm2 logs mihub-backend

# Restart
pm2 restart mihub-backend

# Stop/Start
pm2 stop mihub-backend
pm2 start mihub-backend
```

### Nginx

**Config:** `/etc/nginx/sites-available/orchestratore.mio-hub.me.conf`  
**Proxy:** `localhost:3001`  
**SSL:** Let's Encrypt (Certbot)

**Comandi Utili:**
```bash
# Test configurazione
nginx -t

# Reload
systemctl reload nginx

# Restart
systemctl restart nginx
```

---

## ✅ CONFORMITÀ STRATEGIA UFFICIALE

### Architettura Rispettata

✅ **Frontend (Vercel)** → Nessun accesso diretto DB  
✅ **Core API (Hetzner)** → Unico punto accesso dati  
✅ **Database (Neon)** → PostgreSQL unico  
✅ **Modulo GIS** → Integrato in Core API

### Principi Rispettati

✅ **Single Source of Truth** → Un solo database PostgreSQL  
✅ **Separazione Responsabilità** → Frontend solo UI, backend solo logic  
✅ **API-First** → Tutte le interazioni via API  
✅ **Nessun MySQL** → Completamente standardizzato su PostgreSQL

---

## 🎯 PROSSIMI STEP

### Priorità Alta

1. **Versionare Backend REST su Git**
   - Creare repo `Chcndr/mihub-backend-rest`
   - Commit codice attuale
   - Setup CI/CD per deploy automatico

2. **Dati Reali Grosseto**
   - Caricare 160 piazzole
   - Sostituire `editor-v3-sample.json`

3. **Deploy Frontend Vercel**
   - Merge branch GIS in master ✅ FATTO
   - Verificare deploy automatico
   - Test endpoint produzione

### Priorità Media

4. **Endpoint Dinamici**
   - Implementare `GET /api/gis/markets/:marketId/map`
   - Supporto multi-mercato

5. **Integrazione Database**
   - Salvare dati GIS in PostgreSQL
   - Endpoint `POST /api/gis/markets/:marketId/map`

### Priorità Bassa

6. **Migrazione Monorepo**
   - Fix build TypeScript
   - Port modulo GIS a tRPC
   - Deploy staging e produzione

---

## 📚 RIFERIMENTI

- **STRATEGIA BACKEND UFFICIALE**: `/home/ubuntu/dms-system-blueprint/STRATEGIA_BACKEND_UFFICIALE.md`
- **MASTER SYSTEM PLAN**: `/home/ubuntu/dms-system-blueprint/MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md`
- **BACKEND STATUS**: `/home/ubuntu/dms-system-blueprint/BACKEND_STATUS.md`
- **CREDENZIALI**: `/home/ubuntu/dms-system-blueprint/CREDENZIALI_E_ACCESSI.md`

---

**Ultimo aggiornamento:** 21 Novembre 2025 - 19:40 GMT+1  
**Status:** ✅ UFFICIALE - Fonte di Verità Unica
