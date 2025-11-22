# Sistema Gestione Mercati - Documentazione Completa

**Data Completamento:** 21 Novembre 2025  
**Versione:** 1.0.0  
**Stato:** ✅ Produzione

---

## Panoramica Sistema

Il **Sistema Gestione Mercati** è una piattaforma completa per la gestione digitale di mercati ambulanti, posteggi e concessioni, integrata con un sistema GIS (Geographic Information System) per la visualizzazione e gestione geografica delle piazzole.

### Caratteristiche Principali

Il sistema integra un database PostgreSQL relazionale con un backend REST API Node.js e un frontend React moderno, offrendo una dashboard completa per la pubblica amministrazione. La mappa GIS interattiva permette la visualizzazione in tempo reale di 160 posteggi con collegamento bidirezionale tra tabelle e mappa. L'interfaccia supporta la modifica inline dei dati con aggiornamento automatico della mappa, mentre i popup dinamici mostrano informazioni dettagliate leggendo direttamente dal database.

### Stack Tecnologico

Il backend utilizza Node.js con Express.js per le API REST e PostgreSQL (Neon) come database cloud. Il frontend è costruito con React 18, TypeScript, Tailwind CSS e shadcn/ui per i componenti. La mappa GIS si basa su Leaflet con React-Leaflet, mentre il deployment avviene tramite GitHub Actions su Vercel per il frontend e PM2 su Hetzner per il backend.

---

## Architettura Sistema

### Schema Database

Il database è composto da quattro tabelle principali interconnesse.

La tabella **markets** gestisce i mercati con campi per codice identificativo, nome, comune, provincia, indirizzo, giorni di apertura, numero totale posteggi, stato attivo/inattivo, coordinate GPS e ID GIS per collegamento con la mappa.

La tabella **stalls** gestisce i posteggi con numero identificativo, ID GIS per collegamento mappa, dimensioni (larghezza e profondità), tipo (fisso/spunta/libero), stato (libero/occupato/riservato), orientamento e note.

La tabella **vendors** gestisce le imprese con codice identificativo, ragione sociale, partita IVA, referente, contatti (telefono ed email) e stato attivo/inattivo.

La tabella **concessions** gestisce le concessioni collegando mercati, posteggi e imprese, con tipo di concessione (fisso/spunta/temporaneo) e periodo di validità.

### API REST Backend

Il backend espone 21 endpoint REST organizzati in 4 gruppi.

**Markets API** (`/api/markets`):
- GET `/api/markets` - Lista tutti i mercati
- GET `/api/markets/:id` - Dettaglio mercato singolo
- GET `/api/markets/:id/stalls` - Posteggi di un mercato
- POST `/api/markets` - Crea nuovo mercato
- PATCH `/api/markets/:id` - Aggiorna mercato
- DELETE `/api/markets/:id` - Elimina mercato

**Stalls API** (`/api/stalls`):
- GET `/api/stalls/:id` - Dettaglio posteggio con concessione attiva
- PATCH `/api/stalls/:id` - Aggiorna posteggio (tipo, stato, note)
- POST `/api/stalls` - Crea nuovo posteggio
- DELETE `/api/stalls/:id` - Elimina posteggio

**Vendors API** (`/api/vendors`):
- GET `/api/vendors` - Lista imprese (filtro per status)
- GET `/api/vendors/:id` - Dettaglio impresa
- POST `/api/vendors` - Crea nuova impresa
- PATCH `/api/vendors/:id` - Aggiorna impresa
- DELETE `/api/vendors/:id` - Elimina impresa

**Concessions API** (`/api/concessions`):
- GET `/api/concessions` - Lista concessioni (filtri market_id, vendor_id, stall_id)
- GET `/api/concessions/:id` - Dettaglio concessione
- POST `/api/concessions` - Crea concessione + aggiorna stato posteggio
- PATCH `/api/concessions/:id` - Aggiorna concessione
- DELETE `/api/concessions/:id` - Elimina concessione + libera posteggio

### Frontend Dashboard

Il frontend è organizzato in una struttura gerarchica a tre livelli.

**Livello 1 - Lista Mercati**: Card con informazioni principali (codice, nome, comune, totale posteggi, stato) e click su card per aprire il dettaglio.

**Livello 2 - Dettaglio Mercato**: Tre tab per gestire diversi aspetti del mercato.

Il **Tab Anagrafica** mostra tutti i campi del mercato in formato read-only con layout a griglia responsive.

Il **Tab Posteggi** presenta un layout split con tabella scrollabile a sinistra (160 righe) e mappa GIS interattiva a destra. Include statistiche real-time (occupati/liberi/riservati), modifica inline con select dropdown, collegamento bidirezionale tabella ↔ mappa e pulsante espandi/comprimi mappa.

Il **Tab Imprese/Concessioni** mostra card imprese con dati completi e tabella concessioni attive con filtri.

### Mappa GIS

La mappa GIS offre diverse funzionalità avanzate.

**Visualizzazione**: 160 rettangoli ruotati perfettamente allineati, numeri scalabili con zoom (formula: `8 * 1.5^(zoom - 18)`), marker rosso "M" al centro mercato e 4 layer selezionabili (Stradale, Satellite, Topografica, Dark Mode).

**Interattività**: Colori dinamici basati su stato DB (verde=libero, rosso=occupato, arancione=riservato), popup con dati dal database (numero, dimensioni, tipo, stato, intestatario), tooltip permanenti con numeri posteggi e zoom/pan con controlli standard Leaflet.

**Collegamento Bidirezionale**: Click su riga tabella → centra mappa sul posteggio con animazione `flyTo()`. Click su rettangolo mappa → evidenzia riga tabella con sfondo colorato.

---

## Dati Demo

### Mercato Grosseto

Il sistema include un mercato demo completo.

**Anagrafica Mercato**:
- Codice: GR001
- Nome: Mercato Grosseto
- Comune: Grosseto
- Giorni: Martedì, Giovedì
- Totale Posteggi: 160
- Stato: Attivo
- Coordinate: 42.7585560, 11.11423200
- GIS Market ID: grosseto-market

**Posteggi**: 160 posteggi numerati da 1 a 160, dimensioni 4m × 7.6m, distribuzione tipi (80% fisso, 15% spunta, 5% libero), distribuzione stati (60% occupato, 30% libero, 10% riservato) e collegamento GIS tramite `gis_slot_id` (formato: "stall-1", "stall-2", ...).

**Imprese**: 5 imprese registrate con codici V001-V005, ragioni sociali complete, partite IVA valide, referenti e contatti completi.

**Concessioni**: 5 concessioni attive per i primi 5 posteggi, tipo fisso, validità 01/01/2025 - 31/12/2025 e collegamento con imprese.

---

## Deployment

### Backend (Hetzner)

Il backend è deployato su server Hetzner con le seguenti specifiche.

**Server**: root@95.217.7.247 (Hetzner Cloud)  
**Directory**: `/root/dms-orchestrator`  
**Process Manager**: PM2 con nome processo `dms-orchestrator`  
**URL Produzione**: https://orchestratore.mio-hub.me  
**Porta**: 3000 (interna), 443 (HTTPS con Nginx reverse proxy)

**Variabili Ambiente**:
```
POSTGRES_URL=postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-a5qlqxvd.us-east-2.aws.neon.tech/neondb?sslmode=require
PORT=3000
NODE_ENV=production
```

**Comandi Utili**:
```bash
# Restart backend
ssh root@95.217.7.247 "pm2 restart dms-orchestrator"

# View logs
ssh root@95.217.7.247 "pm2 logs dms-orchestrator --lines 50"

# Deploy nuove routes
scp routes/*.js root@95.217.7.247:/root/dms-orchestrator/routes/
ssh root@95.217.7.247 "pm2 restart dms-orchestrator"
```

### Frontend (Vercel)

Il frontend è deployato su Vercel con deployment automatico.

**Repository**: https://github.com/Chcndr/dms-hub-app-new  
**Branch**: master  
**URL Produzione**: https://dms-hub-app-new.vercel.app  
**Deployment**: Automatico su push GitHub  
**Build Command**: `npm run build`  
**Output Directory**: `dist`

**Variabili Ambiente**:
```
VITE_API_URL=https://orchestratore.mio-hub.me
```

### Database (Neon PostgreSQL)

Il database è hostato su Neon PostgreSQL cloud.

**Host**: ep-bold-silence-a5qlqxvd.us-east-2.aws.neon.tech  
**Database**: neondb  
**User**: neondb_owner  
**Password**: npg_lYG6JQ5Krtsi  
**SSL Mode**: require  
**Console**: https://console.neon.tech

**Connection String**:
```
postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-a5qlqxvd.us-east-2.aws.neon.tech/neondb?sslmode=require
```

---

## File Principali

### Backend

**Routes**:
- `/root/dms-orchestrator/routes/markets.js` - API mercati
- `/root/dms-orchestrator/routes/stalls.js` - API posteggi
- `/root/dms-orchestrator/routes/vendors.js` - API imprese
- `/root/dms-orchestrator/routes/concessions.js` - API concessioni

**Main**:
- `/root/dms-orchestrator/index.js` - Entry point con registrazione routes

### Frontend

**Componenti**:
- `/client/src/components/GestioneMercati.tsx` - Componente principale (36KB)
- `/client/src/components/ZoomFontUpdater.tsx` - Componente zoom font dinamico

**Pagine**:
- `/client/src/pages/DashboardPA.tsx` - Dashboard PA con tab Gestione Mercati
- `/client/src/pages/MarketGISPage.tsx` - Pagina mappa GIS standalone

### Database

**Schema**:
- `/home/ubuntu/gestione-mercati/schema.sql` - Schema completo database
- `/home/ubuntu/gestione-mercati/seed_data.sql` - Dati demo

---

## Funzionalità Avanzate

### Modifica Inline Posteggi

La modifica inline permette di aggiornare tipo e stato dei posteggi direttamente dalla tabella.

**Workflow**:
1. Click su icona Edit (matita) nella colonna Azioni
2. I campi Tipo e Stato diventano dropdown editabili
3. Selezione nuovo valore dai dropdown
4. Click su icona Save (check) per salvare
5. PATCH request a `/api/stalls/:id` con nuovi valori
6. Aggiornamento automatico tabella e mappa
7. Toast di conferma "Posteggio aggiornato con successo"

**Campi Editabili**:
- **Tipo**: fisso, spunta, libero
- **Stato**: libero, occupato, riservato
- **Note**: campo testo libero (futuro)

### Collegamento Bidirezionale

Il collegamento bidirezionale sincronizza tabella e mappa in tempo reale.

**Tabella → Mappa**:
1. Click su riga tabella
2. Lettura `gis_slot_id` dal record
3. Ricerca feature corrispondente in `mapData.stalls_geojson`
4. Calcolo centro poligono (media coordinate)
5. Chiamata `setMapCenter([lat, lng])`
6. Componente `MapCenterController` esegue `map.flyTo()` con animazione 1s
7. Zoom automatico a livello 19
8. Evidenziazione riga con `bg-[#14b8a6]/20`

**Mappa → Tabella**:
1. Click su rettangolo mappa
2. Lettura numero posteggio da `feature.properties.number`
3. Lookup record stall tramite `stallsByNumber` Map
4. Chiamata `setSelectedStallId(stall.id)`
5. Evidenziazione riga con `bg-[#14b8a6]/20`
6. Scroll automatico alla riga (opzionale, attualmente disabilitato)

### Popup Dinamici

I popup della mappa leggono dati dal database in tempo reale.

**Contenuto Popup**:
- Numero posteggio (es. "Piazzola #1")
- Dimensioni (es. "4m × 7.6m")
- Tipo (es. "Tipo: fisso")
- Stato (es. "Stato: occupato")
- Intestatario (se presente):
  - Ragione sociale impresa
  - Nome referente

**Implementazione**:
```typescript
const dbStall = stallsByNumber.get(props.number);
// Popup mostra dbStall.width, dbStall.depth, dbStall.type, 
// dbStall.status, dbStall.vendor_business_name, etc.
```

### Colori Dinamici

I colori dei rettangoli sulla mappa riflettono lo stato del posteggio nel database.

**Mapping Colori**:
- 🟢 Verde (`#10b981`): stato = "libero"
- 🔴 Rosso (`#ef4444`): stato = "occupato"
- 🟠 Arancione (`#f59e0b`): stato = "riservato"
- 🔵 Teal (`#14b8a6`): stato = altro (default)

**Aggiornamento Real-Time**: Dopo modifica inline → PATCH API → `fetchData()` → ricarica stalls → aggiorna colori mappa automaticamente.

---

## Procedure Manutenzione

### Aggiungere Nuovo Mercato

Per aggiungere un nuovo mercato al sistema:

1. **Preparare Dati GIS**: Esportare JSON da Slot Editor v3 con container, stalls, markers, areas
2. **Inserire Mercato**: POST `/api/markets` con dati anagrafica
3. **Importare Posteggi**: Iterare su `stalls_geojson.features` e POST `/api/stalls` per ogni posteggio
4. **Verificare**: GET `/api/markets/:id/stalls` per controllare inserimento
5. **Testare Mappa**: Aprire Dashboard PA → Gestione Mercati → selezionare mercato → tab Posteggi

### Aggiornare Concessione

Per aggiornare una concessione esistente:

1. **Identificare Concessione**: GET `/api/concessions?market_id=1` per trovare ID
2. **Aggiornare**: PATCH `/api/concessions/:id` con nuovi dati (valid_to, type, etc.)
3. **Verificare**: GET `/api/concessions/:id` per controllare aggiornamento
4. **Controllare Posteggio**: GET `/api/stalls/:stall_id` per verificare stato aggiornato

### Backup Database

Per eseguire backup del database:

```bash
# Backup completo
pg_dump "postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-a5qlqxvd.us-east-2.aws.neon.tech/neondb?sslmode=require" > backup_$(date +%Y%m%d).sql

# Backup solo dati
pg_dump --data-only "postgresql://..." > backup_data_$(date +%Y%m%d).sql

# Restore
psql "postgresql://..." < backup_20251121.sql
```

### Monitoraggio Sistema

Per monitorare lo stato del sistema:

**Backend Health Check**:
```bash
curl https://orchestratore.mio-hub.me/health
# Risposta attesa: {"status":"ok","version":"1.1.0",...}
```

**Database Connection Test**:
```bash
psql "postgresql://..." -c "SELECT COUNT(*) FROM markets;"
# Risposta attesa: 1 (mercato Grosseto)
```

**Frontend Build Status**:
- Vercel Dashboard: https://vercel.com/chcndr/dms-hub-app-new
- Deployment Logs: Verificare build success e deploy time

---

## Troubleshooting

### Problema: Mappa Non Si Carica

**Sintomi**: Spinner verde infinito, nessun rettangolo visibile.

**Cause Possibili**:
1. Endpoint GIS `/api/gis/market-map` non risponde
2. Dati GeoJSON malformati
3. Coordinate errate

**Soluzioni**:
1. Verificare endpoint: `curl https://orchestratore.mio-hub.me/api/gis/market-map`
2. Controllare console browser per errori JavaScript
3. Verificare formato GeoJSON in risposta API
4. Controllare coordinate centro mercato in database

### Problema: Collegamento Tabella-Mappa Non Funziona

**Sintomi**: Click su riga non centra mappa, click su mappa non evidenzia riga.

**Cause Possibili**:
1. `gis_slot_id` non corrisponde tra database e GeoJSON
2. `stallsByNumber` Map non popolata correttamente
3. Coordinate centro posteggio errate

**Soluzioni**:
1. Verificare `gis_slot_id` in database: `SELECT number, gis_slot_id FROM stalls LIMIT 5;`
2. Verificare `properties.number` in GeoJSON: controllare console browser
3. Aggiungere log in `handleRowClick()` per debug
4. Verificare che `mapData.stalls_geojson.features` sia popolato

### Problema: Modifica Inline Non Salva

**Sintomi**: Click su Save non aggiorna dati, errore in toast.

**Cause Possibili**:
1. Endpoint PATCH `/api/stalls/:id` non risponde
2. Validazione dati fallisce
3. Database connection error

**Soluzioni**:
1. Verificare endpoint: `curl -X PATCH https://orchestratore.mio-hub.me/api/stalls/1 -H "Content-Type: application/json" -d '{"status":"libero"}'`
2. Controllare logs backend: `pm2 logs dms-orchestrator`
3. Verificare formato body request in Network tab browser
4. Controllare vincoli database (foreign keys, check constraints)

### Problema: Popup Mappa Vuoti

**Sintomi**: Popup si apre ma non mostra dati, solo numero posteggio.

**Cause Possibili**:
1. `stallsByNumber` Map non popolata
2. Numero posteggio non corrisponde tra GeoJSON e database
3. Dati database mancanti

**Soluzioni**:
1. Aggiungere log: `console.log('stallsByNumber:', stallsByNumber);`
2. Verificare corrispondenza numeri: `SELECT number FROM stalls ORDER BY number;`
3. Controllare che `fetchData()` sia completato prima del render mappa
4. Verificare che `dbStall` non sia `undefined` nel codice popup

---

## Roadmap Futuri Sviluppi

### Fase 1 - Completamento Funzionalità Base (Q1 2026)

**Filtri e Ricerca**: Filtro per stato posteggio (libero/occupato/riservato), ricerca per numero posteggio, filtro per tipo posteggio (fisso/spunta/libero) e ricerca impresa per ragione sociale.

**Gestione Concessioni Avanzata**: Form creazione nuova concessione, selezione posteggio libero da mappa, calcolo automatico scadenze e notifiche scadenza imminente.

**Report e Statistiche**: Dashboard analytics con grafici, report occupazione mensile, storico concessioni per posteggio e export Excel/PDF.

### Fase 2 - Integrazione DMS Legacy (Q2 2026)

**Migrazione Dati**: Analisi schema database Heroku, mapping campi legacy → nuovo schema, script migrazione automatica e validazione dati migrati.

**Replica Logica Gestionale**: Gestione presenze operatori, calcolo tasse e tributi, stampa bollettini pagamento e integrazione con sistema contabile.

**Sincronizzazione**: Sync bidirezionale legacy ↔ nuovo sistema, gestione conflitti e log operazioni.

### Fase 3 - Funzionalità Avanzate (Q3 2026)

**App Mobile Operatori**: Check-in/check-out posteggi, foto documentazione, segnalazioni anomalie e navigazione GPS al posteggio.

**Sistema Prenotazioni**: Prenotazione online posteggi spunta, calendario disponibilità, pagamento online e conferma automatica.

**Integrazione IoT**: Sensori occupazione posteggi real-time, monitoraggio ambientale (temperatura, umidità) e alert automatici.

### Fase 4 - AI e Automazione (Q4 2026)

**Ottimizzazione Allocazione**: AI per suggerire assegnazione posteggi ottimale, predizione domanda futura, bilanciamento tipologie merceologiche e massimizzazione revenue.

**Chatbot Assistenza**: Bot Telegram per operatori, risposte automatiche FAQ, prenotazioni via chat e notifiche personalizzate.

**Analytics Predittivi**: Previsione affluenza mercati, trend occupazione stagionali, analisi comportamento operatori e suggerimenti pricing dinamico.

---

## Contatti e Supporto

**Sviluppatore**: Manus AI Agent  
**Cliente**: DMS MIO-HUB  
**Data Rilascio**: 21 Novembre 2025  
**Versione**: 1.0.0  

**Repository GitHub**:
- Frontend: https://github.com/Chcndr/dms-hub-app-new
- Blueprint: https://github.com/Chcndr/dms-system-blueprint

**URL Produzione**:
- Dashboard: https://dms-hub-app-new.vercel.app/dashboard-pa
- Backend API: https://orchestratore.mio-hub.me

**Documentazione Tecnica**:
- `/home/ubuntu/gestione-mercati/ARCHITETTURA_GESTIONE_MERCATI.md`
- `/home/ubuntu/gestione-mercati/TEST_RESULTS.md`
- `/home/ubuntu/gis-handoff-final/REPORT_FINALE_GIS_21NOV2025.md`

---

**Fine Documentazione**
