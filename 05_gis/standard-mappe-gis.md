# Standard Mappe GIS

**Data:** 22 Novembre 2024  
**Autore:** Manus AI Agent

---

## 1. Standard Ufficiale

### 1.1 Mappe Mercati e Posteggi

- **Componente:** `MarketMapComponent`
- **Libreria:** Leaflet + OpenStreetMap
- **Dati:** Dati reali dal backend (`markets`, `stalls`)
- **Caratteristiche:**
  - Zoom fisso livello 19
  - Popup informativi
  - Colori basati su stato occupazione

### 1.2 Mappe Trasporti Pubblici

- **Componente:** `MobilityMap`
- **Libreria:** Google Maps
- **Dati:** Dati reali dal backend (`mobility_data`)
- **Caratteristiche:**
  - Calcolo percorsi
  - Icone personalizzate

## 2. Componenti Deprecati

- **Componente:** `GISMap`
- **Stato:** ⚠️ **DEPRECATO - NON USARE**
- **Motivo:** Sostituito da `MarketMapComponent`

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
**Ultimo aggiornamento:** 22 Novembre 2024 - 16:15 GMT+1
