# MASTER SYSTEM PLAN - DMS / MIO-HUB

**Stato:** APPROVED  
**Owner:** Chcndr  
**Executor principale:** Manus / ABACUS  
**Ultimo aggiornamento:** 2025-11-21

---

## 1. Visione Generale del Sistema

### 1.1. Stato Attuale (AS-IS)

L'ecosistema attuale è frammentato su più sistemi:

- **DMS Legacy (Heroku):** Gestionale storico che detiene la logica di business e i dati operativi.
- **Backend (Hetzner):** API Node.js che serve la nuova Dashboard.
- **Database (PlanetScale/MySQL):** Residui di una precedente architettura, da dismettere.
- **Dashboard (Vercel):** Frontend di nuova generazione che deve diventare il punto di accesso unico.

### 1.2. Architettura Attuale (AS-IS Aggiornato - 21 Nov 2025)

✅ **BACKEND UFFICIALE IN PRODUZIONE:** `mihub-backend-rest`

- **DMS Core API (Hetzner):** ✅ Attivo su `orchestratore.mio-hub.me`
  - Path: `/root/mihub-backend-rest/`
  - Tecnologia: Node.js 22.13.0 + Express
  - PM2 Process: `mihub-backend` (porta 3001)
  - Endpoint GIS: ✅ Funzionante
- **Database (PostgreSQL su Neon):** ✅ Unica fonte di verità
  - Host: `ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech`
  - Database: `neondb`
  - MySQL/PlanetScale: ✅ Completamente rimosso
- **Dashboard (Vercel):** ✅ Frontend Next.js
  - Repository: `Chcndr/dms-hub-app-new`
  - Branch GIS: `feature/gis-market-map-v1` ✅ Merged in master
  - Accesso DB: ✅ Solo via Core API (nessun accesso diretto)
- **DMS Legacy (Heroku):** Fonte dati da migrare

### 1.3. Architettura Futura (TO-BE)

Migrazione a monorepo TypeScript + tRPC (non urgente, backend attuale funzionante):

- **DMS Core API:** Monorepo `Chcndr/mihub` (TypeScript + tRPC + Drizzle)
- **Database:** Stesso PostgreSQL (Neon)
- **Dashboard:** Stesso frontend (Vercel)
- **Timeline:** Da definire (richiede fix build TypeScript)

---

## 2. Piano di Migrazione DMS Legacy

La migrazione avverrà in modo incrementale, copiando la logica e i dati dal sistema legacy al nuovo Core API. Le fasi principali sono:

1. **Analisi e Mapping:** Analisi completa del DB e delle funzionalità del DMS legacy.
2. **Sviluppo API:** Implementazione degli endpoint nel nuovo Core API per replicare le funzionalità.
3. **Migrazione Dati:** Trasferimento dei dati da Heroku a Neon.
4. **Validazione e Dismissione:** Verifica del nuovo sistema e spegnimento del legacy.

---

## 3. Integrazione Pepe GIS / Editor v3

Questa è la priorità assoluta per rendere operativa la pianta digitale del mercato.

### 3.1. Struttura Dati (JSON Editor v3)

Il JSON esportato dall'Editor v3 contiene:

- **`container`:** Bounding box del mercato.
- **`stalls_geojson`:** FeatureCollection GeoJSON con le piazzole.

Ogni piazzola (`feature`) ha:

- **`geometry`:** Coordinate `[lon, lat]`.
- **`properties`:** Metadati come `number`, `orientation`, `kind`, `status`, `dimensions`.

### 3.2. Schema Database (PostgreSQL)

La tabella `stalls` viene estesa per includere i dati GIS:

```sql
CREATE TABLE stalls (
    id SERIAL PRIMARY KEY,
    market_id INTEGER NOT NULL REFERENCES markets(id),
    number INTEGER NOT NULL,
    orientation REAL,
    status VARCHAR(50),
    width_m REAL,
    depth_m REAL,
    lat REAL,
    lng REAL,
    raw_geojson JSONB,
    legacy_id VARCHAR(255),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(market_id, number)
);
```

### 3.3. API Necessarie

- **`POST /markets/{id}/map/import`:** Per caricare la pianta dall'Editor v3.
- **`GET /markets/{id}/map`:** Per leggere la mappa di un mercato (piazzole, geometrie, stato).

---

## 4. Dipendenze e Integrazioni

- **MIO-HUB / DMS-HUB:** Frontend principale che consuma le API del Core.
- **App Ambulanti:** Si integrerà con il Core API per presenze e notifiche.
- **Heroku:** Fonte dati temporanea per la migrazione.
- **Hetzner/Vercel/Neon:** Infrastruttura cloud di produzione del nuovo sistema.
