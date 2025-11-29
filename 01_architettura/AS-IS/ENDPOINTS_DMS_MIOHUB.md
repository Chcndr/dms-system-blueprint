## Tabella Endpoint "Verità Operativa"

**Stato di riferimento:** 29 Novembre 2025

Questa tabella confronta il catalogo teorico degli endpoint (da `MIO-hub/api/index.json`) con i risultati dei test reali eseguiti sul backend attivo (`mihub-backend-rest`).

| Method | Path | Descrizione | Stato Reale | Note |
| :--- | :--- | :--- | :--- | :--- |
| GET | `/api/dmsHub/markets/list` | Lista mercati con statistiche | ✅ **OK** | Funzionante, restituisce dati reali. |
| GET | `/api/dmsHub/markets/getById` | Dettagli mercato per ID | ✅ **OK** | Funzionante, restituisce dati reali. |
| GET | `/api/dmsHub/stalls/listByMarket` | Lista posteggi per mercato | 🔴 **ROTTO** | Errore 500: `column s.vendor_id does not exist` |
| GET | `/api/dmsHub/vendors/list` | Lista venditori | 🔴 **ROTTO** | Errore 500: `column c.status does not exist` |
| GET | `/api/dmsHub/concessions/list` | Lista concessioni | ✅ **OK** | Funzionante, restituisce dati reali. |
| GET | `/api/gis/health` | Health check GIS | ✅ **OK** | Funzionante. |
| GET | `/api/gis/market-map` | Mappa mercato Grosseto | ✅ **OK** | Funzionante. |
| GET | `/api/guardian/health` | Health check Guardian | ✅ **OK** | Funzionante. |
| GET | `/api/abacus/sql/tables` | Lista tabelle database | ✅ **OK** | Funzionante. |
| GET | `/api/markets` | Endpoint legacy markets | ✅ **OK** | Funzionante, ma da considerare legacy. |
| GET | `/api/vendors` | Endpoint legacy vendors | ✅ **OK** | Funzionante, ma da considerare legacy. |
| GET | `/api/stalls` | Endpoint legacy stalls | 🔴 **NON IMPLEMENTATO** | Restituisce 404. |
| POST | `/api/dmsHub/stalls/updateStatus` | Aggiorna stato posteggio | ⚪ **NON TESTATO** | Non implementato nel backend attivo. |
| POST | `/api/dmsHub/vendors/create` | Crea nuovo venditore | ⚪ **NON TESTATO** | Non implementato nel backend attivo. |
| POST | `/api/dmsHub/concessions/assign` | Assegna concessione | ⚪ **NON TESTATO** | Non implementato nel backend attivo. |
|
