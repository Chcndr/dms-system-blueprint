# Standard Mappe GIS (AS-IS)

**Data:** 22 Novembre 2024  
**Autore:** Manus AI Agent

---

## 1. Stato Attuale (AS-IS)

La Dashboard PA utilizza due componenti mappa standardizzati, ognuno con uno scopo specifico. Questa documentazione riflette lo stato **reale in produzione**.

### 1.1 Mappe Mercati e Posteggi

- **Componente:** `MarketMapComponent`
- **Libreria:** Leaflet + OpenStreetMap
- **Dati:** Riceve dati come props:
  - `mapData`: GeoJSON con geometrie posteggi
  - `stallsData`: Dati aggiornati dal database (stato, operatore, etc.)
- **Endpoint tRPC (chiamati dalla Dashboard PA):**
  - `dmsHub.markets.getMapData`
  - `dmsHub.stalls.listByMarket`
- **Logica Colori:**
  - `free`: Verde (`#10b981`)
  - `occupied`: Rosso (`#ef4444`)
  - `reserved`: Arancione (`#f59e0b`)

### 1.2 Mappe Trasporti Pubblici

- **Componente:** `MobilityMap`
- **Libreria:** Google Maps
- **Dati:** Riceve dati come props:
  - `stops`: Array di fermate/parcheggi
- **Endpoint tRPC (chiamato dalla Dashboard PA):**
  - `mobility.list`

## 2. Componenti Deprecati

- **Componente:** `GISMap`
- **Stato:** ⚠️ **DEPRECATO - NON USARE**
- **Motivo:** Sostituito da `MarketMapComponent`.

## 3. Utilizzo nella Dashboard PA

| Sezione | Componente Usato |
|---|---|
| Overview | `MarketMapComponent` |
| Mercati | `MarketMapComponent` |
| Segnalazioni | `MarketMapComponent` |
| Controlli | `MarketMapComponent` |
| Gestione Mercati | `MarketMapComponent` (interno) |
| Gestione HUB | `MarketMapComponent` (interno) |
| Centro Mobilità | `MobilityMap` |

---

**Autore:** Manus AI Agent  
**Ultimo aggiornamento:** 22 Novembre 2024 - 17:15 GMT+1
