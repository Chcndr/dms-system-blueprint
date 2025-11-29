# Blueprint Infrastruttura DMS / MIO-hub

**Stato di riferimento:** 29 Novembre 2025
**Autore:** Manus AI Agent (in modalità "Dev Umano Esterno")
**Obiettivo:** Creare una singola fonte di verità per l'infrastruttura, i componenti, gli endpoint e i processi di deploy, basata esclusivamente su dati reali e verificati.

---

## 1. Mappa Componenti Reali

Questa tabella riassume tutti i componenti software identificati nell'ecosistema DMS/MIO-hub, il loro stato attuale e dove trovarli.

| Nome Componente | Repo GitHub | Hosting | Dominio / URL Base | Endpoint Health | Stato |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Frontend Dashboard PA** | `Chcndr/dms-hub-app-new` | Vercel | `https://dms-hub-app-new.vercel.app` | N/A | ✅ **OK** |
| **Backend Unico (GIS+DMS)** | `Chcndr/mihub-backend-rest` | Hetzner | `https://mihub.157-90-29-66.nip.io` | `/api/gis/health` | 🟠 **Parziale** |
| **Database** | N/A | Neon | `ep-bold-silence-adftsojg-pooler...` | N/A | ✅ **OK** |
| **Blueprint Documentazione** | `Chcndr/dms-system-blueprint` | GitHub | `https://github.com/Chcndr/dms-system-blueprint` | N/A | ✅ **OK** |
| **Frontend Legacy** | `Chcndr/dms-hub-app` | Vercel | *Non deployato* | N/A | ⚪ **Da Dismettere** |
| **Backend Monorepo Legacy** | `Chcndr/mihub` | N/A | *Non deployato* | N/A | ⚪ **Da Dismettere** |
| **Backend Monorepo Legacy 2**| `Chcndr/MIO-hub` | N/A | *Non deployato* | N/A | ⚪ **Da Dismettere** |

---

## 2. Verifica Concreta su Hetzner (Backend)

L'infrastruttura backend è centralizzata su un singolo server Hetzner.

- **Server IP**: `157.90.29.66`
- **Servizi Principali**: Solo **Nginx** e **PM2** sono in esecuzione. Non ci sono altri servizi come database (Postgres, MySQL, Redis) attivi direttamente sul server.

### Processo PM2 Attivo

L'unica applicazione Node.js in esecuzione è gestita da PM2.

- **Nome Processo**: `mihub-backend`
- **Stato**: ✅ **Online** (uptime: 36+ minuti, 0 restarts)
- **Path Entrypoint**: `/root/mihub-backend-rest/index.js`
- **Directory di Lavoro**: `/root/mihub-backend-rest`
- **Porta di Ascolto**: `3000` (localhost)
- **URL Pubblico**: `https://mihub.157-90-29-66.nip.io` (via reverse proxy Nginx)

### Endpoint GIS Verificati

Gli endpoint del modulo GIS, critici per la mappa, sono stati verificati e risultano funzionanti.

- **`GET /api/gis/health`**: ✅ **OK**. Restituisce `{"success":true,"service":"gis-module",...}`.
- **`GET /api/gis/market-map?marketCode=GR001`**: ✅ **OK**. Restituisce un GeoJSON valido con `"total_stalls": 160`.

### Residui e Directory Legacy

Sul server sono presenti diverse directory che rappresentano versioni precedenti o backup. Queste directory **non sono utilizzate** dal processo PM2 attivo e sono candidate per la dismissione al fine di ridurre la confusione.

- `/root/mihub` (vecchio monorepo)
- `/root/MIO-hub` (vecchio monorepo)
- `/root/mihub-backend-rest-legacy-20251124`
- `/root/mihub-backend-rest.backup-20251129-005032`

---

## 3. Verifica Concreta su Vercel (Frontend)

Il frontend principale è ospitato su Vercel.

- **Progetto Vercel**: `dms-hub-app-new`
- **Repository Collegato**: `Chcndr/dms-hub-app-new`
- **URL Pubblico**: `https://dms-hub-app-new.vercel.app`

### Configurazione API

La configurazione delle chiamate API dal frontend al backend presenta un'incongruenza critica.

- **`.env.production`**: Specifica `VITE_API_URL=https://orchestratore.mio-hub.me`.
- **Codice Sorgente**: Il codice utilizza `import.meta.env.VITE_API_URL` ma con un fallback a `http://localhost:3000`.
- **Dominio `orchestratore.mio-hub.me`**: Questo dominio **non è configurato** e non punta a nessun IP. Di conseguenza, le chiamate API dal frontend deployato su Vercel **falliscono** o non funzionano come previsto.
- **CORS sul Backend**: Il backend è configurato per accettare richieste da `dms-hub-app-new.vercel.app`, quindi il problema non è legato a CORS ma alla risoluzione del dominio API.

### Altri Progetti Vercel

- **`dms-hub-app`**: Esiste un repository GitHub ma non sembra avere un deploy Vercel attivo o una `homepageUrl` configurata. È da considerarsi **legacy**.
- **`dms-cittadini` / `dms-operatore`**: Non sono stati trovati repository GitHub corrispondenti. Tuttavia, sono presenti nella configurazione CORS del backend, suggerendo che potrebbero essere progetti futuri o esistenti con nomi diversi.

---

## 4. Audit Endpoint DMS Hub (`/api/dmsHub`)

È stato eseguito un test completo su tutti gli endpoint del modulo `/api/dmsHub` per verificarne lo stato reale.

### Tabella di Stato Endpoint

| Metodo | Endpoint | Stato | Dettagli Errore / Note |
| :--- | :--- | :--- | :--- |
| GET | `/api/dmsHub/markets/list` | ✅ **Funzionante** | Restituisce la lista dei mercati con statistiche. |
| GET | `/api/dmsHub/markets/getById` | ✅ **Funzionante** | Restituisce i dettagli di un mercato dato il `marketId`. |
| GET | `/api/dmsHub/concessions/list` | ✅ **Funzionante** | Restituisce la lista delle concessioni. |
| GET | `/api/dmsHub/stalls/listByMarket` | 🔴 **Errore 500** | `column s.vendor_id does not exist` |
| GET | `/api/dmsHub/vendors/list` | 🔴 **Errore 500** | `column c.status does not exist` |

### Report Dettagliato dei Test

Di seguito il log completo dei test eseguiti con `curl`.

```text
==================================================================
TEST ENDPOINT DMS HUB - 29 Novembre 2025
==================================================================

-------------------------------------------------------------------
TEST: GET /api/dmsHub/markets/list
-------------------------------------------------------------------
HTTP Status: 200
Response Body:
{
  "success": true,
  "data": [ ... ],
  "count": 1
}
STATO: ✅ FUNZIONANTE

-------------------------------------------------------------------
TEST: GET /api/dmsHub/markets/getById?marketId=1
-------------------------------------------------------------------
HTTP Status: 200
Response Body:
{
  "success": true,
  "data": { ... }
}
STATO: ✅ FUNZIONANTE

-------------------------------------------------------------------
TEST: GET /api/dmsHub/stalls/listByMarket?marketId=1
-------------------------------------------------------------------
HTTP Status: 500
Response Body:
{
  "success": false,
  "error": "Failed to fetch stalls",
  "message": "column s.vendor_id does not exist"
}
STATO: 🔴 ERRORE SERVER (500)

-------------------------------------------------------------------
TEST: GET /api/dmsHub/vendors/list
-------------------------------------------------------------------
HTTP Status: 500
Response Body:
{
  "success": false,
  "error": "Failed to fetch vendors list",
  "message": "column c.status does not exist"
}
STATO: 🔴 ERRORE SERVER (500)

-------------------------------------------------------------------
TEST: GET /api/dmsHub/concessions/list
-------------------------------------------------------------------
HTTP Status: 200
Response Body:
{
  "success": true,
  "data": [ ... ],
  "count": 11
}
STATO: ✅ FUNZIONANTE

==================================================================
```

---

## 5. Allineamento e Prossimi Passi

Questo documento rappresenta la fotografia aggiornata e verificata dell'infrastruttura. Le informazioni qui contenute sono la **singola fonte di verità**.

### Azioni Correttive Immediate

1.  **Risolvere gli endpoint DMS Hub rotti**: È prioritario correggere le query SQL in `routes/dmsHub.js` per allinearle allo schema del database Neon e rendere funzionanti gli endpoint `stalls/listByMarket` e `vendors/list`.
2.  **Correggere la configurazione API del frontend**: Modificare il file `.env.production` nel repository `Chcndr/dms-hub-app-new` per puntare all'URL corretto del backend: `VITE_API_URL=https://mihub.157-90-29-66.nip.io`.

### Piano di Lavoro Futuro

Una volta completate le azioni correttive, si potrà procedere con:

- **Automazione del Deploy**: Configurare un webhook GitHub → Hetzner per automatizzare il deploy del backend.
- **Normalizzazione Domini**: Configurare `mio-hub.me` e `api.mio-hub.me` per puntare rispettivamente a Vercel e Hetzner.
- **Pulizia Legacy**: Rimuovere le directory e archiviare i repository legacy per eliminare la confusione.
