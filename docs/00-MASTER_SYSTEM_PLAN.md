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

### 1.2. Architettura Futura (TO-BE)

L'obiettivo è consolidare tutto in un'architettura API-first, con un unico database e un frontend disaccoppiato:

- **DMS Core API (Hetzner):** Unico backend che espone tutte le funzionalità.
- **Database (PostgreSQL su Neon):** Unica fonte di verità per tutti i dati.
- **Dashboard (Vercel):** Unico punto di accesso per tutte le operazioni.
- **DMS Legacy (Heroku):** Fonte dati da migrare e poi dismettere.

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
