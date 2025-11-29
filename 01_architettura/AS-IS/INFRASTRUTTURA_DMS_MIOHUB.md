# Infrastruttura AS-IS: DMS / MIO-hub

**Stato di riferimento:** 29 Novembre 2025
**Autore:** Manus AI Agent

## 1. Componenti Principali

| Componente | Repository | Hosting | URL / IP | Stato |
| :--- | :--- | :--- | :--- | :--- |
| **Frontend PA** | `Chcndr/dms-hub-app-new` | Vercel | `dms-hub-app-new.vercel.app` | ✅ **OK** |
| **Backend Attivo** | `Chcndr/mihub-backend-rest` | Hetzner | `157.90.29.66` | 🟠 **Parziale** |
| **Database** | N/A | Neon | `ep-bold-silence-adftsojg-pooler...` | ✅ **OK** |
| **Monorepo Legacy** | `Chcndr/mihub` | N/A | *Non in uso* | ⚪ **Da Dismettere** |

### Frontend PA

- **URL Produzione**: `https://dms-hub-app-new.vercel.app`
- **Route Chiave**:
  - `/dashboard-pa`: ✅ Funzionante
  - `/market-gis`: 🟠 Parzialmente funzionante (dipende da endpoint rotti)
  - `/guardian/endpoints`: ✅ Funzionante
  - `/guardian/logs`: ✅ Funzionante
- **Configurazione API**: Punta a `orchestratore.mio-hub.me` (dominio **non risolto**), causando errori.

### Backend Attivo (`mihub-backend-rest`)

- **Server**: Hetzner (`157.90.29.66`)
- **Processo PM2**: `mihub-backend` (script: `/root/mihub-backend-rest/index.js`)
- **Porta Interna**: `3000`
- **URL Pubblico**: `https://mihub.157-90-29-66.nip.io` (via Nginx)
- **Stato**: Attualmente serve **solo** il modulo GIS e alcuni endpoint DMS. Non è l'orchestratore completo.

### Modulo GIS

- **Sorgente**: Il codice GIS funzionante è in `mihub-backend-rest`.
- **API Verificate**:
  - `GET /api/gis/health`: ✅ **OK**
  - `GET /api/gis/market-map`: ✅ **OK**
- **Dati**: Utilizza `editor-v3-FINAL-CORRECT.json`.

### Database (Neon)

- **Connessione**: La stringa di connessione nel file `.env` del backend attivo è corretta.
- **Tabelle**: Le tabelle principali (`markets`, `stalls`, `vendors`, `concessions`) esistono, ma lo schema non corrisponde alle query di alcuni endpoint.

## 2. Stato Endpoint

La tabella completa dello stato degli endpoint è disponibile in [ENDPOINTS_DMS_MIOHUB.md](./ENDPOINTS_DMS_MIOHUB.md).

**Riepilogo:**
- **Endpoint DMS Funzionanti**: 3 su 5 (`markets/list`, `markets/getById`, `concessions/list`)
- **Endpoint DMS Rotti**: 2 su 5 (`stalls/listByMarket`, `vendors/list`)
- **Endpoint GIS**: ✅ Tutti funzionanti.

## 3. Discrepanze e Problemi Aperti

1.  **Confusione Repository**: Esistono due repository backend (`mihub-backend-rest` e `mihub`). Solo il primo è attivo, ma il secondo (monorepo) contiene codice che potrebbe essere considerato più recente o completo. **Decisione richiesta**: Quale repository è la fonte di verità per il futuro?
2.  **Configurazione Frontend Errata**: L'URL dell'API nel frontend è sbagliato, impedendo il corretto funzionamento della Dashboard PA.
3.  **Endpoint DMS Rotti**: Due endpoint critici per la visualizzazione di posteggi e venditori sono rotti a causa di errori SQL.
