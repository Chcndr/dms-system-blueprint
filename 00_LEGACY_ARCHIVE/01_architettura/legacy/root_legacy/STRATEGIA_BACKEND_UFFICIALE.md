# STRATEGIA BACKEND UFFICIALE - DMS / MIO-HUB

**Data:** 21 Novembre 2025  
**Autore:** Manus AI  
**Stato:** Ufficiale

---

## 🎯 DECISIONE: BACKEND UFFICIALE

### Backend Attuale (AS-IS)

**Nome:** `mihub-backend-rest`  
**Path Server:** `/root/mihub-backend-rest/` (Hetzner 157.90.29.66)  
**Repository GitHub:** Nessuno (codice locale)  
**Status:** ✅ **BACKEND UFFICIALE IN PRODUZIONE**

**Motivazione:**
- ✅ Funzionante e stabile in produzione
- ✅ Endpoint GIS implementati e testati
- ✅ Database PostgreSQL (Neon) configurato
- ✅ Nginx + PM2 configurati correttamente
- ✅ CORS configurato per frontend Vercel

---

## 📋 STRATEGIA: UNIFICAZIONE BACKEND

### Opzione Scelta: **Unificare tutto in monorepo `mihub`**

**Repository Target:** `Chcndr/mihub`  
**Path Locale:** `/home/ubuntu/mihub`  
**Path Server:** `/root/mihub`

**Motivazione:**
1. **Architettura TO-BE**: Il MASTER SYSTEM PLAN prevede backend TypeScript + tRPC
2. **Type Safety**: TypeScript end-to-end con tRPC
3. **Manutenibilità**: Codice versionato su Git, non locale
4. **Scalabilità**: Monorepo con workspace separati

---

## 🔄 PIANO DI MIGRAZIONE

### Fase 1: Fix Build Monorepo `mihub` (⏳ DA FARE)

**Obiettivo:** Ripristinare build TypeScript funzionante

**Azioni:**
1. Analisi errori TypeScript esistenti
2. Fix dipendenze mancanti:
   - `@vercel/node`
   - `node-fetch`
   - `nanoid`
3. Aggiornamento Drizzle per PostgreSQL:
   - Rimuovere metodi MySQL (`onDuplicateKeyUpdate`)
   - Usare solo metodi PostgreSQL
4. Fix tipi Neon HTTP Query Result
5. Risoluzione conflitti `storage.ts`
6. Test build locale: `npm run build`

**Deliverable:**
- ✅ Build TypeScript senza errori
- ✅ Test locale funzionante
- ✅ Commit su branch `fix/typescript-build`

### Fase 2: Port Modulo GIS da REST a tRPC (⏳ DA FARE)

**Obiettivo:** Migrare endpoint GIS nel monorepo

**Azioni:**
1. Creare router tRPC `gisRouter.ts`
2. Port logica da `/root/mihub-backend-rest/routes/gis.js`
3. Aggiungere validazione input con Zod
4. Integrare con database PostgreSQL:
   - Leggere dati da tabelle `markets`, `market_geometry`, `stalls`
   - Generare GeoJSON dinamicamente
5. Test endpoint tRPC locale
6. Documentare schema input/output

**Deliverable:**
- ✅ Router tRPC `gisRouter.ts`
- ✅ Endpoint `gis.marketMap` funzionante
- ✅ Integrazione database PostgreSQL
- ✅ Commit su branch `feature/gis-trpc`

### Fase 3: Deploy Staging Monorepo (⏳ DA FARE)

**Obiettivo:** Testare monorepo in ambiente staging

**Azioni:**
1. Deploy su porta diversa (es. 3002)
2. Configurare Nginx per staging subdomain
3. Test endpoint GIS staging
4. Test integrazione frontend staging
5. Monitoring logs e performance

**Deliverable:**
- ✅ Backend monorepo su staging
- ✅ Endpoint GIS testati
- ✅ Frontend collegato a staging

### Fase 4: Migrazione Produzione (⏳ DA FARE)

**Obiettivo:** Sostituire backend REST con monorepo

**Azioni:**
1. Backup backend REST attuale
2. Deploy monorepo su porta 3001
3. Aggiornare PM2 process
4. Test endpoint produzione
5. Monitoring 24h
6. Dismissione backend REST

**Deliverable:**
- ✅ Monorepo `mihub` in produzione
- ✅ Backend REST dismesso
- ✅ Un solo backend ufficiale

---

## ⚠️ GESTIONE TRANSITORIA

### Durante la Migrazione

**Backend REST (`mihub-backend-rest`):**
- ✅ Rimane attivo in produzione
- ✅ Continua a servire endpoint GIS
- ✅ Nessuna modifica breaking
- ⚠️ Nuove feature solo in monorepo

**Monorepo (`mihub`):**
- ⏳ Sviluppo attivo su branch separati
- ⏳ Deploy staging per test
- ⏳ Nessun impatto su produzione

### Dopo la Migrazione

**Backend Ufficiale Unico:** `Chcndr/mihub`
- ✅ TypeScript + tRPC + Drizzle
- ✅ PostgreSQL (Neon)
- ✅ Versionato su Git
- ✅ Deploy automatizzato

**Backend REST:**
- ❌ Dismesso
- 📦 Archiviato come backup
- 📝 Documentato in blueprint

---

## 📊 CONFRONTO BACKEND

| Caratteristica | Backend REST (Attuale) | Monorepo mihub (Target) |
|---|---|---|
| **Tecnologia** | Node.js + Express | TypeScript + tRPC |
| **ORM** | Nessuno | Drizzle |
| **Type Safety** | ❌ No | ✅ Sì (end-to-end) |
| **Repository** | ❌ Locale | ✅ GitHub |
| **Versionamento** | ❌ No | ✅ Git |
| **Build** | ✅ Funzionante | ❌ Errori TS |
| **Deploy** | ✅ Produzione | ❌ Non deployato |
| **Endpoint GIS** | ✅ Implementato | ⏳ Da portare |
| **Database** | ✅ PostgreSQL | ✅ PostgreSQL |
| **Manutenibilità** | ⚠️ Media | ✅ Alta |
| **Scalabilità** | ⚠️ Limitata | ✅ Alta |

---

## 🎯 OBIETTIVO FINALE

**Un solo backend ufficiale:** `Chcndr/mihub`

**Caratteristiche:**
- ✅ TypeScript + tRPC per type safety
- ✅ Drizzle ORM per PostgreSQL
- ✅ Versionato su GitHub
- ✅ Build automatizzato
- ✅ Deploy su Hetzner
- ✅ Endpoint GIS integrati
- ✅ Nessun doppione o confusione

**Eliminazione:**
- ❌ Backend REST locale (`mihub-backend-rest`)
- ❌ Codice non versionato
- ❌ Ambiguità tra due backend

---

## 📝 PROSSIMI STEP IMMEDIATI

### 1. Analisi Errori TypeScript (Priorità Alta)

**Obiettivo:** Identificare e categorizzare errori build

**Azioni:**
```bash
cd /home/ubuntu/mihub/apps/api
npm run build 2>&1 | tee build-errors.log
```

**Output Atteso:**
- Lista file con errori
- Tipo di errore (modulo mancante, tipo errato, etc.)
- Moduli usati vs non usati

### 2. Fix Dipendenze (Priorità Alta)

**Obiettivo:** Installare moduli mancanti

**Azioni:**
```bash
npm install @vercel/node node-fetch nanoid
```

### 3. Fix Drizzle PostgreSQL (Priorità Alta)

**Obiettivo:** Rimuovere metodi MySQL

**Azioni:**
- Cercare `onDuplicateKeyUpdate` nel codice
- Sostituire con `onConflictDoUpdate` (PostgreSQL)
- Test query database

### 4. Test Build (Priorità Alta)

**Obiettivo:** Build verde senza errori

**Azioni:**
```bash
npm run build
# Expected: ✓ built in Xs
```

---

## 📚 RIFERIMENTI

- **MASTER SYSTEM PLAN**: `/home/ubuntu/dms-system-blueprint/MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md`
- **BACKEND STATUS**: `/home/ubuntu/dms-system-blueprint/BACKEND_STATUS.md`
- **Repository Monorepo**: https://github.com/Chcndr/mihub
- **Repository Frontend**: https://github.com/Chcndr/dms-hub-app-new

---

**Ultimo aggiornamento:** 21 Novembre 2025 - 20:15 GMT+1
