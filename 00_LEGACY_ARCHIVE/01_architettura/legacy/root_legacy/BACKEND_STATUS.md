# STATO BACKEND - DMS / MIO-HUB

**Data:** 21 Novembre 2025  
**Autore:** Manus AI  
**Stato:** Ufficiale

---

## 📊 SITUAZIONE ATTUALE (AS-IS)

### Backend in Produzione

**Path Server:** `/root/mihub-backend-rest/` (Server Hetzner)
**Repository GitHub:** Nessuno (codice locale sul server)
**Commit/Tag:** N/A (backend standalone, non versionato su Git)

**Caratteristiche:**
- **Tecnologia:** Node.js + Express
- **Porta:** 3001
- **Process Manager:** PM2 (`mihub-backend`)
- **Proxy:** Nginx → `orchestratore.mio-hub.me`
- **Database:** PostgreSQL (Neon)
- **Stato:** ✅ Attivo e funzionante

**Endpoint Implementati:**

| Endpoint | Metodo | Descrizione | Stato |
|---|---|---|---|
| `/health` | GET | Health check backend | ✅ Produzione |
| `/api/dmsHub/*` | * | Endpoint DMS Hub (mock) | ✅ Produzione |
| `/api/logs/*` | * | Endpoint logs (mock) | ✅ Produzione |
| `/api/mihub/*` | * | Endpoint MIO-HUB (mock) | ✅ Produzione |
| `/api/gis/health` | GET | Health check modulo GIS | ✅ Produzione |
| `/api/gis/market-map` | GET | Dati mappa mercato (Editor v3) | ✅ Produzione |

**Modulo GIS:**
- **File Router:** `/root/mihub-backend-rest/routes/gis.js` (111 righe)
- **File Dati:** `/root/mihub-backend-rest/data/editor-v3-sample.json` (3 piazzole esempio)
- **Formato Input:** JSON Editor v3 (container, center, stalls_geojson, markers_geojson, areas_geojson)
- **Formato Output:** GeoJSON compatibile con Leaflet
- **URL Produzione:** https://orchestratore.mio-hub.me/api/gis/market-map
- **Metodo:** GET
- **Autenticazione:** Nessuna (endpoint pubblico)
- **CORS:** Configurato per domini Vercel

**Esempio Risposta `/api/gis/market-map`:**

```json
{
  "success": true,
  "data": {
    "container": [[lat, lng], ...],
    "center": { "lat": 42.758, "lng": 11.114 },
    "stalls_geojson": {
      "type": "FeatureCollection",
      "features": [...]
    },
    "markers_geojson": { "type": "FeatureCollection", "features": [] },
    "areas_geojson": { "type": "FeatureCollection", "features": [] }
  },
  "meta": {
    "endpoint": "gis.marketMap",
    "timestamp": "2025-11-21T19:30:00.000Z",
    "source": "editor-v3-sample.json",
    "stalls_count": 3
  }
}
```

**Middleware:**
- API Guardian: Temporaneamente disabilitato per endpoint GIS
- CORS: Configurato per Vercel domains
- Rate Limiting: Attivo (100 req/15min)

---

## 🎯 TARGET FINALE (TO-BE)

### Backend Monorepo `mihub`

**Repository:** `Chcndr/mihub` (GitHub)  
**Path Locale:** `/home/ubuntu/mihub`  
**Path Server:** `/root/mihub`

**Caratteristiche Target:**
- **Tecnologia:** TypeScript + tRPC + Drizzle ORM
- **Database:** PostgreSQL (Neon) - unico DB
- **Architettura:** Monorepo con workspace separati
- **API Style:** tRPC per type-safety end-to-end

**Stato Attuale:**
- ❌ Build fallisce con errori TypeScript
- ❌ Non deployato in produzione
- ✅ Codice endpoint GIS presente (non utilizzato)

**Errori TypeScript Identificati:**
1. Moduli mancanti (`@vercel/node`, `node-fetch`, `nanoid`)
2. Metodi Drizzle non compatibili (`onDuplicateKeyUpdate`)
3. Problemi di tipi su Neon HTTP Query Result
4. Conflitti di tipi in `storage.ts`

**Strategia di Migrazione:**
1. **Fase 1** (Attuale): Usare backend REST come adapter temporaneo
2. **Fase 2** (Prossima): Fix errori TypeScript nel monorepo
3. **Fase 3** (Finale): Migrazione endpoint da REST a tRPC

---

## 🔄 PIANO DI TRANSIZIONE

### Step 1: Consolidamento Backend REST (✅ COMPLETATO)

- [x] Modulo GIS integrato in `mihub-backend-rest`
- [x] Endpoint `/api/gis/market-map` funzionante
- [x] Deploy in produzione su Hetzner
- [x] Nginx configurato correttamente (porta 3001)
- [x] Test endpoint produzione OK

### Step 2: Fix Monorepo `mihub` (⏳ DA FARE)

- [ ] Analisi completa errori TypeScript
- [ ] Rimozione/isolamento moduli non usati
- [ ] Fix dipendenze mancanti
- [ ] Aggiornamento Drizzle per PostgreSQL
- [ ] Test build locale
- [ ] Test deploy su Hetzner (ambiente staging)

### Step 3: Migrazione Endpoint GIS (⏳ DA FARE)

- [ ] Port endpoint GIS da REST a tRPC
- [ ] Integrazione con database PostgreSQL
- [ ] Test end-to-end
- [ ] Deploy graduale in produzione
- [ ] Dismissione backend REST

---

## 📝 NOTE OPERATIVE

### Accesso Server Hetzner

```bash
# SSH con chiave
ssh -i /home/ubuntu/.ssh/hetzner_server_key root@157.90.29.66

# Verificare stato PM2
pm2 list
pm2 logs mihub-backend

# Restart backend
pm2 restart mihub-backend

# Verificare Nginx
nginx -t
systemctl status nginx
```

### Test Endpoint GIS

```bash
# Health check
curl https://orchestratore.mio-hub.me/api/gis/health

# Market map
curl https://orchestratore.mio-hub.me/api/gis/market-map | jq '.meta'
```

### File Chiave

**Backend REST:**
- `/root/mihub-backend-rest/index.js` - Entry point
- `/root/mihub-backend-rest/routes/gis.js` - Modulo GIS
- `/root/mihub-backend-rest/data/editor-v3-sample.json` - Dati mappa
- `/root/mihub-backend-rest/.env` - Configurazione

**Nginx:**
- `/etc/nginx/sites-available/orchestratore.mio-hub.me.conf`
- `/etc/nginx/sites-enabled/orchestratore.mio-hub.me.conf`

**PM2:**
- Process name: `mihub-backend`
- Working directory: `/root/mihub-backend-rest/`
- Entry point: `index.js`

---

## ⚠️ LIMITAZIONI ATTUALI

### Backend REST Temporaneo

1. **Dati Hardcoded**: Il file `editor-v3-sample.json` contiene solo 3 piazzole di esempio
2. **Nessun Database**: I dati non sono salvati su PostgreSQL
3. **Endpoint Statici**: Non supporta parametri dinamici (es. `marketId`)
4. **API Guardian Disabilitato**: Nessun controllo permessi sugli endpoint GIS

### Prossimi Sviluppi Necessari

1. **Endpoint Dinamici**:
   - `GET /api/gis/markets/:marketId/map` - Mappa specifica mercato
   - `POST /api/gis/markets/:marketId/map` - Caricamento dati Editor v3
   
2. **Integrazione Database**:
   - Salvare dati GIS in tabelle `markets`, `market_geometry`, `stalls`
   - Collegare piazzole GIS con entità `stalls` tramite `gis_feature_id`
   
3. **Sincronizzazione**:
   - Import automatico da Editor v3 a PostgreSQL
   - Export da PostgreSQL a formato GeoJSON

---

## 📚 RIFERIMENTI

- **Master System Plan**: `/home/ubuntu/dms-system-blueprint/MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md`
- **Credenziali**: `/home/ubuntu/dms-system-blueprint/CREDENZIALI_E_ACCESSI.md`
- **Repository Backend**: https://github.com/Chcndr/mihub
- **Repository Frontend**: https://github.com/Chcndr/dms-hub-app-new
- **Repository MIO-HUB**: https://github.com/Chcndr/MIO-hub

---

**Ultimo aggiornamento:** 21 Novembre 2025 - 19:30 GMT+1
