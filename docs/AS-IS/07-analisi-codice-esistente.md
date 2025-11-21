# 07 - Analisi Codice Esistente

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Obiettivo:** Documentare lo stato attuale del codice nei repository frontend e backend.

---

## 1. Frontend (dms-hub-app-new)

**Repository:** `Chcndr/dms-hub-app-new`  
**Hosting:** Vercel  
**Stack:** Next.js, React, tRPC Client

### Configurazione Database

Il frontend è configurato con Drizzle ORM per PostgreSQL:

```typescript
// drizzle.config.ts
export default defineConfig({
  schema: "./drizzle/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: {
    url: connectionString,
  },
});
```

È presente un file di backup `drizzle.config.mysql.backup.ts` che indica una migrazione precedente da MySQL a PostgreSQL.

### Variabili di Ambiente

**File `.env.example`:**
- `VITE_TRPC_URL`: URL del backend tRPC (default: `http://localhost:3000/api/trpc`)
- `DATABASE_URL`: Configurazione database (attualmente MySQL Railway - **da rimuovere**)

**File `.env.production`:**
- `VITE_TRPC_URL`: `https://orchestratore.mio-hub.me`
- `DATABASE_URL`: `mysql://...` (**PROBLEMA: va rimosso**)

### Accessi Diretti al Database

**Risultato della ricerca nel codice sorgente (`src/`):** ✅ **NESSUN ACCESSO DIRETTO AL DATABASE TROVATO**

Il frontend è già conforme all'architettura TO-BE: comunica esclusivamente tramite tRPC con il backend.

### Azioni Richieste

- [ ] Rimuovere la variabile `DATABASE_URL` dal file `.env.production`
- [ ] Rimuovere la variabile `DATABASE_URL` dal file `.env.example`
- [ ] Verificare che non ci siano configurazioni Vercel che includano `DATABASE_URL`

---

## 2. Backend (orchestratore.mio-hub.me)

**URL:** `https://orchestratore.mio-hub.me`  
**Hosting:** Presumibilmente Hetzner (da confermare)  
**Stack:** tRPC (da verificare)

### Stato Analisi

Non è stato possibile identificare il repository GitHub specifico che contiene il codice del backend tRPC. I repository analizzati:

- **MIO-hub**: Contiene struttura per task e configurazione agente, non codice backend
- **dms-gemello-core**: Focalizzato su sistema GIS, non backend Core API

### Azioni Richieste

- [ ] Identificare il repository del backend tRPC
- [ ] Verificare la configurazione del database PostgreSQL
- [ ] Documentare gli endpoint tRPC esistenti
- [ ] Verificare che tutta la business logic sia centralizzata nel backend

---

## 3. Sistema Legacy DMS (Heroku)

**URL:** `https://lapsy-dms.herokuapp.com`  
**Database:** PostgreSQL su Heroku (credenziali non disponibili)

### Stato Analisi

È stata completata un'analisi funzionale completa tramite l'interfaccia web (vedi `references/ANALISI_DMS_LEGACY_COMPLETA.md`).

### Azioni Richieste

- [ ] Ottenere credenziali di accesso in sola lettura al database PostgreSQL di Heroku
- [ ] Analizzare lo schema del database per completare il mapping campo-per-campo
- [ ] Sviluppare script ETL per la migrazione dei dati

---

## 4. Conclusioni

Lo stato attuale del codice è promettente:

- Il frontend è già pulito e non ha accessi diretti al database.
- La configurazione Drizzle è già su PostgreSQL.
- Resta da fare pulizia delle variabili di ambiente obsolete e identificare/documentare il backend.
