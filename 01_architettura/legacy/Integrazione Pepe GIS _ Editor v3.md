# Integrazione Pepe GIS / Editor v3

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Stato:** ✅ COMPLETATO E IN PRODUZIONE  
**Obiettivo:** Documentare la struttura del file JSON generato dall'Editor v3 di Pepe GIS, il suo mapping sul modello dati, e l'integrazione nel backend ufficiale.

**Status Implementazione:**
- ✅ Endpoint Backend: `GET /api/gis/market-map` (Produzione)
- ✅ Frontend Dashboard: Componente `MarketGISPage` (Merged in master)
- ✅ URL Produzione: https://orchestratore.mio-hub.me/api/gis/market-map
- ✅ Formato Dati: GeoJSON compatibile Leaflet
- ✅ Build Frontend: Pulito senza errori TypeScript

---

## 1. Struttura del JSON dell'Editor v3

Il file JSON esportato dall'Editor v3 rappresenta la pianta digitale di un mercato. La sua struttura è la seguente:

```json
{
  "container": [ ... ],
  "stalls_geojson": {
    "type": "FeatureCollection",
    "features": [ ... ]
  }
}
```

### 1.1 `container`

- **Tipo**: Array di 4 coordinate `[lat, lon]`.
- **Significato**: Rappresenta il bounding box (rettangolo di contenimento) dell'area del mercato.

### 1.2 `stalls_geojson`

- **Tipo**: Oggetto GeoJSON di tipo `FeatureCollection`.
- **Significato**: Contiene un array di `features`, dove ogni feature rappresenta una singola piazzola.

### 1.3 `features[]`

Ogni elemento dell'array `features` è un oggetto GeoJSON di tipo `Feature` con la seguente struttura:

```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [ ... ]
  },
  "properties": {
    "number": 1,
    "orientation": 122.8,
    "kind": "slot",
    "status": "free",
    "dimensions": "3.99m × 7.59m"
  }
}
```

- **`geometry`**: Oggetto GeoJSON che definisce la posizione della piazzola.
  - **`type`**: `Point`
  - **`coordinates`**: Array `[lon, lat]` con le coordinate della piazzola.

- **`properties`**: Oggetto contenente i metadati della piazzola.
  - **`number`**: Numero identificativo della piazzola.
  - **`orientation`**: Angolo di rotazione in gradi.
  - **`kind`**: Tipo di piazzola (es. `slot`).
  - **`status`**: Stato iniziale della piazzola (es. `free`).
  - **`dimensions`**: Stringa che descrive le dimensioni (es. `"4m × 7.6m"`).

---

## 2. Mapping sul Modello Dati (PostgreSQL)

I dati del JSON v3 vengono mappati sulla tabella `stalls` del database PostgreSQL. La tabella `stalls` (o `market_stalls`) deve essere estesa per includere i dati GIS.

### 2.1 Struttura Tabella `stalls`

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

### 2.2 Mapping dei Campi

| Campo JSON v3 | Campo Tabella `stalls` | Note |
|---|---|---|
| `properties.number` | `number` | Numero della piazzola. |
| `properties.orientation` | `orientation` | Valore numerico. |
| `properties.status` | `status` | Stringa. |
| `properties.dimensions` | `width_m`, `depth_m` | La stringa deve essere parsata per estrarre i due valori numerici. |
| `geometry.coordinates[1]` | `lat` | Latitudine. |
| `geometry.coordinates[0]` | `lng` | Longitudine. |
| Oggetto `feature` completo | `raw_geojson` | L'intera feature GeoJSON viene salvata per riferimento futuro. |

### 2.3 Decisione su PostGIS

Per la Fase 1, si è deciso di adottare un approccio semplificato senza l'uso di PostGIS, utilizzando colonne separate per `lat` e `lng` e un campo `JSONB` per la geometria grezza. Questo permette di accelerare lo sviluppo iniziale.

**Evoluzione futura**: Se le query geospaziali diventeranno complesse (es. ricerca di piazzole in un'area), si valuterà l'introduzione di PostGIS e la sostituzione di `lat`/`lng` con una colonna `geometry(Point, 4326)`.
