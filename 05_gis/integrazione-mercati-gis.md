# Integrazione Mercati e GIS (Grosseto)

**Data:** 22 Novembre 2024  
**Autore:** Manus AI Agent  
**Versione:** 1.0

---

## 1. Obiettivo

Creare un **gemello digitale** dei mercati di Grosseto, integrando le piantine GIS con i dati gestionali del DMS, per avere una visione in tempo reale dello stato operativo dei mercati.

---

## 2. Flusso Dati: Dal PDF alla Mappa Interattiva

1. **Georeferenziazione PDF**: Le piantine dei mercati in formato PDF vengono georeferenziate in QGIS.
2. **Digitalizzazione Posteggi**: I posteggi vengono digitalizzati come poligoni o punti, e a ciascuno viene assegnato un `stall_code` univoco.
3. **Caricamento su PostGIS**: I dati geospaziali vengono caricati nel database PostGIS, nelle tabelle `dms_market` e `dms_stall`.
4. **Pubblicazione su G3W-SUITE**: Il progetto QGIS viene pubblicato come servizio WMS/WFS su G3W-SUITE.
5. **Visualizzazione in DMS**: La Dashboard PA usa Leaflet/MapLibre per visualizzare i layer WMS e aggiungere overlay GeoJSON con dati in tempo reale (presenze, pagamenti, etc.).

---

## 3. Modello Dati

| Tabella | Campi Chiave |
| :--- | :--- |
| `dms_market` | `id`, `name`, `city`, `address` |
| `dms_stall` | `id`, `market_id`, `stall_code`, `geometry` (PostGIS), `gis_feature_id` |
| `dms_stall_status` | `stall_id`, `date`, `status` (libero, occupato, spuntista), `vendor_id` |

**Collegamento GIS-DMS:**
- Il campo `stall_code` è la chiave univoca che collega il dato geografico (mappa) al dato gestionale (DMS).
- Il popup sulla mappa in G3W-SUITE contiene un **deep-link** a DMS che apre la scheda dell'operatore o del posteggio.

---

## 4. Integrazioni Tecniche

- **GIS Server**: QGIS Server + G3W-SUITE
- **Database**: PostgreSQL + PostGIS
- **Frontend**: Leaflet / MapLibre
- **Servizi**: WMS/WFS/WMTS

---

## 5. Roadmap Implementazione (Grosseto)

- ✅ **Fase 1: Setup Iniziale**
  - Creazione tabelle PostGIS
  - Georeferenziazione e digitalizzazione primo mercato

- ✅ **Fase 2: Pubblicazione e Visualizzazione**
  - Pubblicazione progetto QGIS su G3W-SUITE
  - Creazione mappa Leaflet in DMS con layer WMS

- ⏳ **Fase 3: Integrazione Dati Real-Time**
  - Sviluppo overlay GeoJSON per presenze e pagamenti
  - Implementazione deep-link da G3W a DMS

- ⏳ **Fase 4: Gestione Completa**
  - Sviluppo pannelli laterali interattivi nella mappa
  - Creazione sezione "Gestionale" collegata ai dati della mappa
