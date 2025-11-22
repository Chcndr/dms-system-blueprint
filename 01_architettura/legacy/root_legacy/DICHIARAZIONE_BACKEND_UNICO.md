# DICHIARAZIONE BACKEND UNICO - DMS / MIO-HUB

**Data:** 21 Novembre 2025  
**Autore:** Manus AI  
**Stato:** ✅ UFFICIALE

---

## 🎯 BACKEND UFFICIALE

### Dichiarazione

**Il backend ufficiale del sistema DMS / MIO-HUB è:**

✅ **`mihub-backend-rest`**

**Path:** `/root/mihub-backend-rest/` (Server Hetzner 157.90.29.66)  
**Dominio:** `orchestratore.mio-hub.me`  
**Status:** IN PRODUZIONE

---

## 📊 REPOSITORY E STATO

### Backend Ufficiale (IN PRODUZIONE)

**Nome:** `mihub-backend-rest`  
**Repository GitHub:** ❌ Nessuno (codice locale)  
**Tecnologia:** Node.js 22.13.0 + Express  
**Database:** PostgreSQL (Neon)  
**PM2:** Process `mihub-backend` (porta 3001)  
**Status:** ✅ ATTIVO E FUNZIONANTE

**Endpoint Principali:**
- `/health` - Health check
- `/api/dmsHub/*` - DMS Hub
- `/api/logs/*` - Logs
- `/api/mihub/*` - MIO-HUB
- `/api/gis/*` - Modulo GIS ✅

**Conformità al Piano:**
- ✅ PostgreSQL unico
- ✅ Nessun MySQL/PlanetScale
- ✅ Frontend non accede DB direttamente
- ✅ Core API su Hetzner
- ✅ Modulo GIS integrato

---

### Repository Storici (NON IN PRODUZIONE)

#### 1. Monorepo `mihub`

**Repository GitHub:** `Chcndr/mihub`  
**Path Server:** `/root/mihub/`  
**Tecnologia:** TypeScript + tRPC + Drizzle ORM  
**Status:** ❌ NON IN PRODUZIONE

**Motivo:**
- Errori compilazione TypeScript
- Dipendenze mancanti
- Metodi Drizzle incompatibili

**Futuro:**
- ⏳ Target per migrazione futura
- ⏳ Richiede fix build TypeScript
- ⏳ Piano migrazione in 4 fasi documentato

**Decisione:** MANTENERE come target futuro, NON archiviare

---

## 🚫 REPOSITORY DA ARCHIVIARE

**Nessuno.**

Tutti i repository hanno uno scopo chiaro:
- `mihub-backend-rest` (locale) → Backend ufficiale in produzione
- `Chcndr/mihub` (GitHub) → Target futuro per migrazione

---

## ⚠️ RACCOMANDAZIONI

### 1. Versionare Backend REST

**Problema:** Il backend ufficiale non è versionato su Git

**Soluzione:**
1. Creare repository `Chcndr/mihub-backend-rest`
2. Commit codice attuale
3. Setup CI/CD per deploy automatico

**Priorità:** Alta  
**Timeline:** Prossime 2 settimane

### 2. Migrazione Monorepo

**Problema:** Backend target (`mihub`) non funziona

**Soluzione:**
1. Fix errori TypeScript
2. Port modulo GIS a tRPC
3. Deploy staging e produzione

**Priorità:** Media  
**Timeline:** 1-2 mesi

---

## 📝 CREDENZIALI E ACCESSI

### Backend Ufficiale

**Server:** 157.90.29.66  
**SSH:** `ssh -i /home/ubuntu/.ssh/hetzner_server_key root@157.90.29.66`  
**Path:** `/root/mihub-backend-rest/`  
**PM2:** `pm2 list` → Process `mihub-backend`

**Comandi Utili:**
```bash
# Restart backend
pm2 restart mihub-backend

# Logs
pm2 logs mihub-backend --lines 100

# Status
pm2 status mihub-backend
```

### Database

**PostgreSQL (Neon)**  
**Host:** `ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech`  
**Database:** `neondb`  
**User:** `neondb_owner`  
**Connection String:** Configurata in `/root/mihub-backend-rest/.env`

### Nginx

**Config:** `/etc/nginx/sites-available/orchestratore.mio-hub.me.conf`  
**Proxy:** `localhost:3001`  
**SSL:** Let's Encrypt

---

## 🔄 PIANO MIGRAZIONE (Futuro)

### Obiettivo

Migrare da `mihub-backend-rest` a monorepo `Chcndr/mihub`

### Fasi

**Fase 1: Fix Build TypeScript** (⏳ Da fare)
- Analisi errori compilazione
- Fix dipendenze mancanti
- Aggiornamento Drizzle per PostgreSQL

**Fase 2: Port Modulo GIS** (⏳ Da fare)
- Migrare endpoint da REST a tRPC
- Integrazione database PostgreSQL

**Fase 3: Deploy Staging** (⏳ Da fare)
- Deploy su porta diversa (3002)
- Test integrazione frontend

**Fase 4: Migrazione Produzione** (⏳ Da fare)
- Backup backend REST
- Deploy monorepo su porta 3001
- Dismissione backend REST

**Timeline:** Non definita (nessuna urgenza)

---

## ✅ CONCLUSIONE

**Backend ufficiale dichiarato:** `mihub-backend-rest`  
**Status:** ✅ IN PRODUZIONE E FUNZIONANTE  
**Ambiguità:** ✅ ELIMINATE  
**Rami paralleli:** ✅ CHIARITI

**Prossimi step:**
1. Versionare backend REST su Git
2. Completare modulo GIS con 160 posteggi
3. Pianificare migrazione monorepo (non urgente)

---

**Ultimo aggiornamento:** 21 Novembre 2025 - 20:00 GMT+1  
**Status:** ✅ UFFICIALE - Dichiarazione definitiva
