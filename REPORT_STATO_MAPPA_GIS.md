# REPORT STATO MAPPA GIS - Dashboard DMS HUB

**Data:** 21 Novembre 2025 - 18:15 GMT+1  
**Fase:** Phase 1 - Integrazione Mappa GIS (Editor v3)  
**Stato:** 🔴 BLOCCATO - Richiede decisione architetturale

---

## EXECUTIVE SUMMARY

**Problema:** La mappa GIS non si carica nella dashboard e l'endpoint API restituisce errore "endpoint is not defined".

**Causa Root:** Sul server Hetzner è deployato un **backend REST con route MOCK** invece del **backend tRPC** che contiene l'endpoint `importFromSlotEditor` necessario per caricare i dati della mappa.

**Impatto:** Impossibile visualizzare la pianta del mercato generata da Editor v3 nella dashboard.

---

## ANALISI DETTAGLIATA

### 1. BACKEND ATTUALE IN PRODUZIONE (Hetzner)

#### Backend REST Mock - ✅ ATTIVO
```
Directory: /root/mihub-backend-rest/
Processo: PM2 (nome: mihub-backend, PID: 49672)
Uptime: 2 giorni
Port: 3001
Tecnologia: Express.js REST API
```

**Endpoint disponibili:**
- `GET /api/dmsHub/markets/list` → Mock data (3 mercati)
- `GET /api/dmsHub/markets/getById` → Mock data
- `GET /api/dmsHub/stalls/listByMarket` → Mock data

**Endpoint MANCANTI:**
- ❌ `POST /api/dmsHub/markets/importFromSlotEditor` → NON ESISTE
- ❌ Nessuna integrazione con database reale
- ❌ Nessun supporto per GeoJSON Editor v3

**Codice sorgente:** `/root/mihub-backend-rest/routes/dmsHub-mock.js`
```javascript
// Solo dati hardcoded, nessuna connessione DB
const mockMarkets = [
  { id: 1, name: "Mercato Albinelli", city: "Modena", ... },
  { id: 2, name: "Mercato Testaccio", city: "Roma", ... },
  { id: 3, name: "Mercato Centrale", city: "Firenze", ... }
];
```

---

### 2. BACKEND TRPC - ❌ NON DEPLOYATO

#### Repository GitHub: Chcndr/mihub
```
Directory locale: /home/ubuntu/mihub/
Directory server: /root/mihub/
Branch: master
Ultimo commit: aacd9e8 (feat: Estensione tabella stalls per Pepe GIS)
Tecnologia: Node.js + tRPC + Drizzle ORM + PostgreSQL
```

**Endpoint IMPLEMENTATI nel codice:**
- ✅ `POST /api/dmsHub/markets/importFromSlotEditor` → File: `apps/api/server/dmsHubRouter.ts` (linea 160-280)
- ✅ `GET /api/dmsHub/markets/list` → Connessione DB reale
- ✅ `GET /api/dmsHub/markets/getById` → Connessione DB reale
- ✅ `GET /api/dmsHub/stalls/listByMarket` → Connessione DB reale

**Struttura codice:**
```
apps/api/
├── src/server.ts          → Entry point Express + tRPC
├── server/
│   ├── routers.ts         → Registrazione router (dmsHub, mihub, etc.)
│   ├── dmsHubRouter.ts    → ✅ Contiene importFromSlotEditor
│   ├── db.ts              → Connessione Drizzle + Neon PostgreSQL
│   └── _core/trpc.ts      → Setup tRPC
├── drizzle/
│   └── schema.ts          → Schema database PostgreSQL
├── package.json
└── .env                   → Variabili ambiente (DATABASE_URL, etc.)
```

**Stato deployment:**
- ❌ NON è in esecuzione su Hetzner
- ❌ Tentativo di build Docker fallito (errori TypeScript)
- ✅ Codice presente su server in `/root/mihub/`

---

### 3. DATI MAPPA GIS DISPONIBILI

#### File JSON Editor v3 - ✅ PRONTO
```
File: /home/ubuntu/dms-hub-app-new/grosseto_160_posteggi_PRONTO.json
Mercato: Grosseto Esperando
Piazzole: 160
Formato: GeoJSON compatibile con Editor v3
```

**Struttura dati:**
```json
{
  "container": [[lat, lng], ...],
  "stalls_geojson": {
    "type": "FeatureCollection",
    "features": [
      {
        "type": "Feature",
        "geometry": { "type": "Point", "coordinates": [lng, lat] },
        "properties": {
          "number": 1,
          "orientation": 122.8,
          "kind": "slot",
          "status": "free",
          "dimensions": "4m × 7.6m"
        }
      },
      // ... altre 159 piazzole
    ]
  }
}
```

**Altri file disponibili:**
- `/home/ubuntu/mihub/apps/frontend/public/data/grosseto_complete.json`
- `/home/ubuntu/dms-hub-app-new/client/public/data/grosseto_full.geojson`
- `/home/ubuntu/dms-gemello-core/data/grosseto_mercato_esperanto.geojson`

---

### 4. DATABASE POSTGRESQL (Neon)

#### Configurazione - ✅ FUNZIONANTE
```
Host: ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech
Database: neondb
User: neondb_owner
Password: npg_lYG6JQ5Krtsi
Account: chcndr@gmail.com
```

**Schema tabelle (Drizzle ORM):**
- ✅ `markets` → Anagrafica mercati
- ✅ `market_geometry` → Geometrie GIS (container, center, GCP, PNG)
- ✅ `stalls` → Piazzole con coordinate geografiche
- ✅ `custom_markers` → Marker personalizzati
- ✅ `custom_areas` → Aree personalizzate

**Stato:**
- ✅ Migrazioni Drizzle applicate
- ✅ Connessione testata e funzionante
- ❌ Nessun dato di Grosseto importato (tabelle vuote)

---

### 5. FRONTEND (Vercel)

#### Dashboard DMS HUB - ✅ DEPLOYATO
```
URL: https://dms-hub-app-new.vercel.app
Repository: Chcndr/dms-hub-app-new
Framework: Next.js + React
```

**Configurazione API:**
```
NEXT_PUBLIC_API_URL: https://orchestratore.mio-hub.me
```

**Problema:**
- Frontend chiama correttamente l'endpoint `POST /api/dmsHub/markets/importFromSlotEditor`
- Ma il backend in produzione (REST mock) NON ha questo endpoint
- Risultato: Errore "endpoint is not defined"

---

## PROBLEMA ARCHITETTURALE

### Discrepanza Backend

Esistono **DUE backend separati** sul server Hetzner:

| Aspetto | Backend REST Mock | Backend tRPC |
|---------|-------------------|--------------|
| **Directory** | `/root/mihub-backend-rest/` | `/root/mihub/` |
| **Stato** | ✅ ATTIVO (PM2) | ❌ NON DEPLOYATO |
| **Tecnologia** | Express REST | Express + tRPC |
| **Database** | ❌ Nessuno (mock) | ✅ PostgreSQL (Neon) |
| **Endpoint GIS** | ❌ Non esiste | ✅ Implementato |
| **Repository** | ❓ Sconosciuto | ✅ GitHub Chcndr/mihub |

### Domanda Critica

**Quale backend è quello "ufficiale" secondo il blueprint?**

Secondo `MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md`:
> - **DMS Core API**: Unico "cervello" del sistema. Gestisce tutta la business logic e l'accesso ai dati. **Hosting: Hetzner**
> - **Tecnologia**: Node.js/tRPC
> - **Database**: PostgreSQL (Neon)

**Risposta:** Il backend tRPC è quello corretto secondo il blueprint.

---

## TENTATIVO DI DEPLOY BACKEND TRPC

### Errori Riscontrati

#### 1. Build Docker fallito
```bash
cd /root/mihub/apps/api
docker compose build
```

**Errore:**
```
error TS2488: Type 'NeonHttpQueryResult<never>' must have a '[Symbol.iterator]()' method
error TS2307: Cannot find module 'nanoid'
error TS2322: Type 'number' is not assignable to type 'string'
error TS2339: Property 'where' does not exist on type 'Omit<PgSelectBase<...>>'
```

**Causa:** Problemi di compatibilità TypeScript nei commit recenti (dopo `f167c60`)

#### 2. Mancanza pnpm-lock.yaml
```
ERR_PNPM_NO_LOCKFILE Cannot install with "frozen-lockfile" because pnpm-lock.yaml is absent
```

**Causa:** Il Dockerfile richiede `pnpm-lock.yaml` ma non esiste nel repository

---

## SOLUZIONI POSSIBILI

### Opzione A: Deploy Backend tRPC (RACCOMANDATO - Allineato al blueprint)

**Vantaggi:**
- ✅ Architettura corretta secondo blueprint
- ✅ Endpoint GIS già implementato
- ✅ Connessione database reale
- ✅ Scalabile e manutenibile

**Passi:**
1. Fixare errori TypeScript nel codice
2. Generare `pnpm-lock.yaml` o modificare Dockerfile
3. Build e deploy con Docker o PM2
4. Sostituire backend REST mock
5. Aggiornare Nginx (se necessario)

**Stima tempo:** 2-4 ore (con supporto sviluppatore)

---

### Opzione B: Implementare endpoint nel Backend REST (QUICK FIX)

**Vantaggi:**
- ✅ Soluzione rapida (30-60 minuti)
- ✅ Nessun cambio architettura
- ✅ Backend già funzionante

**Svantaggi:**
- ❌ Non allineato al blueprint
- ❌ Debito tecnico
- ❌ Dovrà essere rifatto comunque

**Passi:**
1. Aggiungere endpoint in `/root/mihub-backend-rest/routes/dmsHub-mock.js`
2. Implementare logica import GeoJSON
3. Connettere al database PostgreSQL
4. Restart PM2

**Codice da aggiungere:**
```javascript
// POST /api/dmsHub/markets/importFromSlotEditor
router.post('/markets/importFromSlotEditor', async (req, res) => {
  try {
    const { marketName, city, address, slotEditorData } = req.body;
    
    // 1. Connessione DB PostgreSQL
    const { Client } = require('pg');
    const client = new Client({ connectionString: process.env.DATABASE_URL });
    await client.connect();
    
    // 2. Insert market
    const marketResult = await client.query(
      'INSERT INTO markets (name, city, address, lat, lng, active) VALUES ($1, $2, $3, $4, $5, 1) RETURNING id',
      [marketName, city, address, slotEditorData.center.lat, slotEditorData.center.lng]
    );
    const marketId = marketResult.rows[0].id;
    
    // 3. Insert geometry
    await client.query(
      'INSERT INTO market_geometry (market_id, container_geojson, center_lat, center_lng) VALUES ($1, $2, $3, $4)',
      [marketId, JSON.stringify(slotEditorData.container), slotEditorData.center.lat, slotEditorData.center.lng]
    );
    
    // 4. Insert stalls
    for (const stall of slotEditorData.stalls) {
      await client.query(
        'INSERT INTO stalls (market_id, number, lat, lng, area_mq, category) VALUES ($1, $2, $3, $4, $5, $6)',
        [marketId, stall.number, stall.lat, stall.lng, stall.areaMq, stall.category]
      );
    }
    
    await client.end();
    
    res.json({ success: true, marketId });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});
```

---

### Opzione C: Deploy Backend tRPC senza Docker (ALTERNATIVA)

**Vantaggi:**
- ✅ Evita problemi Docker
- ✅ Usa PM2 come backend attuale
- ✅ Allineato al blueprint

**Passi:**
1. Installare dipendenze: `cd /root/mihub/apps/api && npm install`
2. Build con esbuild: `npm run build`
3. Configurare PM2: `pm2 start dist/server.js --name mihub-trpc`
4. Aggiornare Nginx per puntare a nuova porta
5. Fermare backend REST mock

**Problema:** Errori TypeScript bloccano il build

---

## RACCOMANDAZIONE

### Strategia Ibrida (Migliore compromesso)

1. **Immediato (oggi):** Implementare Opzione B (quick fix nel backend REST)
   - Tempo: 1 ora
   - Permette di sbloccare la mappa GIS subito
   - Utente può testare funzionalità

2. **Breve termine (questa settimana):** Fixare backend tRPC e deployare (Opzione A)
   - Richiedere supporto sviluppatore per fix errori TypeScript
   - Pianificare migrazione corretta
   - Documentare processo

3. **Medio termine:** Dismettere backend REST mock
   - Una volta che backend tRPC è stabile
   - Aggiornare documentazione blueprint

---

## PROSSIMI PASSI IMMEDIATI

### Se procedo con Quick Fix (Opzione B):

1. ✅ Aggiungere endpoint in backend REST
2. ✅ Testare import JSON Grosseto
3. ✅ Verificare visualizzazione mappa in dashboard
4. ✅ Documentare soluzione temporanea

### Se procedo con Deploy tRPC (Opzione A):

1. ❓ **RICHIEDE DECISIONE:** Chi può fixare errori TypeScript?
2. ❓ **RICHIEDE DECISIONE:** Usare Docker o PM2?
3. ❓ **RICHIEDE DECISIONE:** Sostituire o affiancare backend REST?

---

## DOMANDE PER IL TEAM

1. **Priorità:** Quick fix o soluzione definitiva?
2. **Supporto:** C'è uno sviluppatore disponibile per fix errori TypeScript?
3. **Architettura:** Conferma che backend tRPC è quello ufficiale?
4. **Timeline:** Quando serve la mappa GIS funzionante?

---

## ALLEGATI

- **Codice endpoint tRPC:** `/home/ubuntu/mihub/apps/api/server/dmsHubRouter.ts` (linea 160-280)
- **Dati JSON pronti:** `/home/ubuntu/dms-hub-app-new/grosseto_160_posteggi_PRONTO.json`
- **Logs backend:** Disponibili via `pm2 logs mihub-backend`
- **Credenziali:** `/home/ubuntu/upload/mihub_credentials(3).pdf`

---

**Stato finale:** ⏸️ IN ATTESA DI DECISIONE STRATEGICA

**Contatto:** Report generato da Manus AI - 21 Nov 2025
