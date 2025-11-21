# Schema Database Pepe GIS

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Obiettivo:** Definire lo schema completo del database PostgreSQL per l'integrazione Pepe GIS.

---

## 1. Tabelle Principali

### 1.1. `markets`

Tabella che contiene le informazioni sui mercati.

```sql
CREATE TABLE markets (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    lat VARCHAR(20) NOT NULL,
    lng VARCHAR(20) NOT NULL,
    opening_hours TEXT,
    active INTEGER DEFAULT 1 NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);
```

### 1.2. `market_geometry`

Tabella che contiene le geometrie del mercato (bounding box, aree custom, ecc.).

```sql
CREATE TABLE market_geometry (
    id SERIAL PRIMARY KEY,
    market_id INTEGER NOT NULL REFERENCES markets(id),
    container_geojson TEXT,
    hub_area_geojson TEXT,
    market_area_geojson TEXT,
    gcp_data TEXT,
    png_metadata TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);
```

### 1.3. `stalls`

Tabella che contiene le piazzole con i dati GIS.

```sql
CREATE TABLE stalls (
    id SERIAL PRIMARY KEY,
    market_id INTEGER NOT NULL REFERENCES markets(id),
    number VARCHAR(20) NOT NULL,
    lat VARCHAR(20) NOT NULL,
    lng VARCHAR(20) NOT NULL,
    area_mq INTEGER,
    status VARCHAR(50) DEFAULT 'free' NOT NULL,
    category VARCHAR(100),
    notes TEXT,
    -- Campi GIS (Pepe GIS / Editor v3)
    orientation REAL,
    kind VARCHAR(50),
    width_m REAL,
    depth_m REAL,
    raw_geojson JSONB,
    legacy_id VARCHAR(255),
    created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
    UNIQUE(market_id, number)
);
```

### 1.4. `custom_markers`

Tabella per marker custom sulla mappa (es. servizi, punti di interesse).

```sql
CREATE TABLE custom_markers (
    id SERIAL PRIMARY KEY,
    market_id INTEGER NOT NULL REFERENCES markets(id),
    name VARCHAR(255) NOT NULL,
    type VARCHAR(100),
    lat VARCHAR(20) NOT NULL,
    lng VARCHAR(20) NOT NULL,
    icon VARCHAR(100),
    description TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);
```

### 1.5. `custom_areas`

Tabella per aree custom sulla mappa (es. zone pedonali, aree verdi).

```sql
CREATE TABLE custom_areas (
    id SERIAL PRIMARY KEY,
    market_id INTEGER NOT NULL REFERENCES markets(id),
    name VARCHAR(255) NOT NULL,
    type VARCHAR(100),
    geojson TEXT NOT NULL,
    color VARCHAR(20),
    description TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);
```

---

## 2. Relazioni

- `market_geometry.market_id` → `markets.id`
- `stalls.market_id` → `markets.id`
- `custom_markers.market_id` → `markets.id`
- `custom_areas.market_id` → `markets.id`

---

## 3. Migrazioni Drizzle

Le migrazioni Drizzle sono già state generate nel repository `mihub/apps/api`. Il file di migrazione è:

- `drizzle/0000_loving_electro.sql`

Per applicare la migrazione al database di produzione:

```bash
cd /opt/mihub/apps/api
pnpm db:push
```

---

## 4. Indici Consigliati

Per ottimizzare le query, si consiglia di creare i seguenti indici:

```sql
CREATE INDEX idx_stalls_market_id ON stalls(market_id);
CREATE INDEX idx_stalls_status ON stalls(status);
CREATE INDEX idx_market_geometry_market_id ON market_geometry(market_id);
CREATE INDEX idx_custom_markers_market_id ON custom_markers(market_id);
CREATE INDEX idx_custom_areas_market_id ON custom_areas(market_id);
```

---

## 5. Note Tecniche

- **Geometrie:** Per la Fase 1, le geometrie sono salvate come `JSONB` o `TEXT` (GeoJSON). Se in futuro servono query geospaziali complesse, si valuterà l'introduzione di PostGIS.
- **Coordinate:** Le coordinate sono salvate come `VARCHAR` per mantenere la precisione massima.
- **Legacy ID:** Il campo `legacy_id` permette di mappare le piazzole del nuovo sistema con quelle del DMS legacy su Heroku.
