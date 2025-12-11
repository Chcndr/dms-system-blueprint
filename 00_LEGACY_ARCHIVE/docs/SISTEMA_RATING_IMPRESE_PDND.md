# SISTEMA RATING IMPRESE PDND - Documentazione Completa

**Data Implementazione:** 7 Dicembre 2025  
**Versione:** 1.0  
**Status:** ✅ FASE 1, 2 & 3 Completate - Deploy Produzione Attivo

---

## Indice

1. [Panoramica](#1-panoramica)
2. [Architettura](#2-architettura)
3. [Database Schema](#3-database-schema)
4. [Backend API](#4-backend-api)
5. [Logica Semaforo Rating](#5-logica-semaforo-rating)
6. [Procedura Deploy](#6-procedura-deploy)
7. [Testing](#7-testing)
8. [Monitoraggio](#8-monitoraggio)
9. [Roadmap](#9-roadmap)

---

## 1. Panoramica

### 1.1. Obiettivo

Evolvere il sistema Imprese in due aree funzionali distinte:

1. **Gestione Mercati > Tab Imprese:** Anagrafica Master (Data Entry) con campi PDND completi
2. **Dashboard > Qualificazioni:** Motore di Rating (Consultazione) con semaforo Verde/Giallo/Rosso

### 1.2. Fasi Implementate

| Fase | Descrizione | Status | Data |
|------|-------------|--------|------|
| **FASE 1** | Database Upgrade (Neon PostgreSQL) | ✅ Completata | 07/12/2025 |
| **FASE 2** | Backend API (Node.js + Express) | ✅ Completata | 07/12/2025 |
| **FASE 3** | Frontend Anagrafica Master | ✅ Completata | 07/12/2025 |
| **FASE 4** | Frontend Rating & Compliance | 🔄 In corso | - |

### 1.3. Componenti

- **Database:** Neon PostgreSQL (3 tabelle: imprese, dms_durc_snapshots, dms_suap_instances)
- **Backend:** Node.js + Express (2 file routes: imprese.js, qualificazioni.js)
- **Frontend:** React + TypeScript (Vercel)
- **Monitoraggio:** Guardian Logs + Debug Dashboard

---

## 2. Architettura

### 2.1. Diagramma Flusso

```
┌─────────────────────────────────────────────────────────────┐
│                 FRONTEND (Vercel)                           │
│  https://dms-hub-app-new.vercel.app                         │
└─────────────────────────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┬─────────────────┐
        │               │               │                 │
        ▼               ▼               ▼                 ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Tab IMPRESE │ │  Tab LOGS   │ │  Tab DEBUG  │ │Tab INTEGRAZ.│
│ (Anagrafica)│ │ (Monitorag.)│ │ (Health)    │ │ (Test API)  │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
        │               │               │                 │
        └───────────────┼───────────────┴─────────────────┘
                        │
                        ▼
            ┌───────────────────────┐
            │  Backend Hetzner      │
            │  Port 3000            │
            │                       │
            │ routes/imprese.js     │
            │ routes/qualificazioni.│
            │ routes/migratePDND.js │
            └───────────────────────┘
                        │
                        ▼
            ┌───────────────────────┐
            │  Neon PostgreSQL      │
            │                       │
            │ • imprese (23 col)    │
            │ • dms_durc_snapshots  │
            │ • dms_suap_instances  │
            │ • qualificazioni      │
            └───────────────────────┘
```

### 2.2. Aree Funzionali

#### Area 1: Anagrafica Master (Data Entry)

**Dove:** Gestione Mercati > Tab "Imprese / Concessioni"  
**Funzione:** Inserimento e modifica dati imprese con campi PDND completi

**Campi Form (3 gruppi):**

1. **Identità:**
   - Denominazione (ex Ragione Sociale)
   - P.IVA e Codice Fiscale
   - Forma Giuridica (Dropdown: SRL, SNC, DI...)
   - REA e CCIAA

2. **Sede Legale:**
   - Indirizzo (Via, Civico)
   - Comune, CAP, Provincia

3. **Contatti & Attività:**
   - PEC (Campo obbligatorio/evidenziato)
   - Telefono e Referente
   - Codice ATECO

#### Area 2: Motore di Rating (Consultazione)

**Dove:** Dashboard > Tab "Qualificazioni"  
**Funzione:** Visualizzazione stato conformità con semaforo

**Elementi UI:**
- 🟢🟡🔴 Badge semaforo
- Card "Stato Conformità" con dettagli
- Click su semaforo → Modale con breakdown completo

---

## 3. Database Schema

### 3.1. Tabella `imprese` (Espansa)

**Campi aggiunti (14 nuovi):**

```sql
ALTER TABLE imprese 
  RENAME COLUMN ragione_sociale TO denominazione;

ALTER TABLE imprese ADD COLUMN:
  numero_rea VARCHAR(20),
  cciaa_sigla VARCHAR(5),
  forma_giuridica VARCHAR(100),
  stato_impresa VARCHAR(50),
  indirizzo_via VARCHAR(255),
  indirizzo_civico VARCHAR(20),
  indirizzo_cap VARCHAR(10),
  indirizzo_provincia VARCHAR(5),
  indirizzo_stato VARCHAR(50) DEFAULT 'Italia',
  pec VARCHAR(255),
  codice_ateco VARCHAR(20),
  descrizione_ateco TEXT,
  data_iscrizione_ri DATE,
  flag_impresa_artigiana BOOLEAN DEFAULT FALSE;
```

**Totale colonne:** 9 → **23**

### 3.2. Tabella `dms_durc_snapshots`

Storico verifiche DURC (Documento Unico Regolarità Contributiva):

```sql
CREATE TABLE dms_durc_snapshots (
  id SERIAL PRIMARY KEY,
  impresa_id INTEGER NOT NULL REFERENCES imprese(id) ON DELETE CASCADE,
  protocollo_inps VARCHAR(100),
  esito_regolarita VARCHAR(50) NOT NULL,  -- REGOLARE / IRREGOLARE
  data_scadenza DATE NOT NULL,            -- Cruciale per semaforo
  data_verifica TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  raw_response JSONB                      -- Payload PDND originale
);

CREATE INDEX idx_durc_impresa ON dms_durc_snapshots(impresa_id);
CREATE INDEX idx_durc_scadenza ON dms_durc_snapshots(data_scadenza);
```

**Funzione:** Traccia tutte le verifiche DURC nel tempo per audit e calcolo rating.

### 3.3. Tabella `dms_suap_instances`

Istanze SUAP (Sportello Unico Attività Produttive):

```sql
CREATE TABLE dms_suap_instances (
  id SERIAL PRIMARY KEY,
  impresa_id INTEGER NOT NULL REFERENCES imprese(id) ON DELETE CASCADE,
  market_id INTEGER,                      -- FK logica verso mercati
  stall_id INTEGER,                       -- FK logica verso posteggi
  cui VARCHAR(100),                       -- Codice Univoco Istanza
  tipo_pratica VARCHAR(100),              -- APERTURA, SUBINGRESSO, RINNOVO
  stato_pratica VARCHAR(50),              -- ACCOLTA, SOSPESA, RIGETTATA
  data_scadenza_concessione DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_suap_impresa ON dms_suap_instances(impresa_id);
CREATE INDEX idx_suap_market ON dms_suap_instances(market_id);
```

**Funzione:** Collega imprese a mercati/posteggi tramite pratiche SUAP.

---

## 4. Backend API

### 4.1. File `routes/imprese.js` (760 righe)

**Endpoint implementati:**

| Metodo | Endpoint | Descrizione | Parametri |
|--------|----------|-------------|-----------|
| GET | `/api/imprese` | Lista imprese | `?status=ATTIVA&comune=Bologna&settore=Alimentare` |
| GET | `/api/imprese/:id` | Dettaglio impresa | - |
| POST | `/api/imprese` | Crea impresa | Body: 23 campi |
| PUT | `/api/imprese/:id` | Aggiorna impresa | Body: campi da aggiornare |
| DELETE | `/api/imprese/:id` | Elimina impresa | - |
| **GET** | **`/api/imprese/:id/rating`** | **🚦 Calcola semaforo** | - |

**Esempio chiamata:**

```bash
# Lista imprese attive a Bologna
curl "http://157.90.29.66:3000/api/imprese?status=ATTIVA&comune=Bologna"

# Dettaglio impresa ID 1
curl "http://157.90.29.66:3000/api/imprese/1"

# Calcola rating impresa ID 1
curl "http://157.90.29.66:3000/api/imprese/1/rating"
```

### 4.2. File `routes/qualificazioni.js` (280 righe)

**Endpoint implementati:**

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| GET | `/api/qualificazioni` | Lista tutte le qualificazioni |
| GET | `/api/qualificazioni/impresa/:id` | Qualificazioni per impresa |
| GET | `/api/qualificazioni/durc/:id` | Storico DURC per impresa |
| POST | `/api/qualificazioni/durc` | Salva snapshot DURC |
| GET | `/api/qualificazioni/suap/:id` | Istanze SUAP per impresa |
| POST | `/api/qualificazioni/suap` | Crea istanza SUAP |
| PUT | `/api/qualificazioni/suap/:id` | Aggiorna istanza SUAP |

**Esempio chiamata:**

```bash
# Storico DURC impresa ID 1
curl "http://157.90.29.66:3000/api/qualificazioni/durc/1"

# Salva nuovo DURC
curl -X POST "http://157.90.29.66:3000/api/qualificazioni/durc" \
  -H "Content-Type: application/json" \
  -d '{
    "impresa_id": 1,
    "protocollo_inps": "DURC-2025-001",
    "esito_regolarita": "REGOLARE",
    "data_scadenza": "2025-06-01"
  }'
```

### 4.3. File `routes/migratePDND.js` (95 righe)

**Endpoint migration:**

```bash
POST /api/admin/migrate-pdnd
```

**Funzione:**
- Esegue migration SQL sul database Neon
- Idempotente (può essere eseguito più volte)
- Registrato in MIO-hub/api/index.json
- Monitorato in sezioni Logs e Debug

**Esecuzione:**

```bash
curl -X POST http://157.90.29.66:3000/api/admin/migrate-pdnd
```

**Risposta attesa:**

```json
{
  "success": true,
  "message": "PDND Migration completed: imprese expanded, dms_durc_snapshots and dms_suap_instances created"
}
```

---

## 5. Logica Semaforo Rating

### 5.1. Endpoint Rating

**URL:** `GET /api/imprese/:id/rating`

**Funzione:** Calcola lo stato di conformità dell'impresa basandosi su:
- Stato impresa (ATTIVA, CESSATA, IN LIQUIDAZIONE)
- DURC (esito e scadenza)
- PEC (presenza)
- Concessioni (stato e scadenza)

### 5.2. Algoritmo

```javascript
let status = 'VERDE'; // Default: conforme
const warnings = [];
const errors = [];
const today = new Date();

// ROSSO: Stato impresa non attiva
if (impresa.stato_impresa !== 'ATTIVA') {
  status = 'ROSSO';
  errors.push({
    type: 'stato_impresa',
    message: `Impresa in stato: ${impresa.stato_impresa}`,
    severity: 'critical'
  });
}

// ROSSO/GIALLO: DURC
if (durc) {
  const scadenzaDurc = new Date(durc.data_scadenza);
  const giorniMancanti = Math.ceil((scadenzaDurc - today) / (1000 * 60 * 60 * 24));

  if (durc.esito_regolarita === 'IRREGOLARE') {
    status = 'ROSSO';
    errors.push({ type: 'durc', message: 'DURC IRREGOLARE', severity: 'critical' });
  } else if (scadenzaDurc < today) {
    status = 'ROSSO';
    errors.push({ type: 'durc', message: 'DURC SCADUTO', severity: 'critical' });
  } else if (giorniMancanti <= 30) {
    if (status !== 'ROSSO') status = 'GIALLO';
    warnings.push({ type: 'durc', message: `DURC in scadenza tra ${giorniMancanti} giorni` });
  }
} else {
  if (status !== 'ROSSO') status = 'GIALLO';
  warnings.push({ type: 'durc', message: 'DURC non presente' });
}

// GIALLO: PEC mancante
if (!impresa.pec || impresa.pec.trim() === '') {
  if (status !== 'ROSSO') status = 'GIALLO';
  warnings.push({ type: 'pec', message: 'PEC mancante o non valida' });
}

// ROSSO/GIALLO: Concessioni
concessioni.forEach(conc => {
  if (conc.data_scadenza_concessione) {
    const scadenzaConc = new Date(conc.data_scadenza_concessione);
    const giorniMancanti = Math.ceil((scadenzaConc - today) / (1000 * 60 * 60 * 24));

    if (scadenzaConc < today) {
      status = 'ROSSO';
      errors.push({ type: 'concessione', message: 'Concessione SCADUTA' });
    } else if (giorniMancanti <= 30) {
      if (status !== 'ROSSO') status = 'GIALLO';
      warnings.push({ type: 'concessione', message: `Concessione in scadenza tra ${giorniMancanti} giorni` });
    }
  }
});
```

### 5.3. Stati Semaforo

| Status | Score | Label | Descrizione |
|--------|-------|-------|-------------|
| 🟢 VERDE | 100 | Conforme | Impresa attiva, DURC regolare, PEC presente, concessioni attive |
| 🟡 GIALLO | 50 | Azione Richiesta | DURC in scadenza, PEC mancante, concessione in scadenza |
| 🔴 ROSSO | 0 | Non Conforme | Impresa cessata, DURC irregolare/scaduto, concessione scaduta |

### 5.4. Risposta JSON Completa

```json
{
  "success": true,
  "data": {
    "impresa_id": 1,
    "denominazione": "Alimentari Rossi & C.",
    "rating": {
      "status": "GIALLO",
      "score": 50,
      "label": "Azione Richiesta"
    },
    "checks": {
      "stato_impresa": {
        "value": "ATTIVA",
        "valid": true
      },
      "durc": {
        "esito": "REGOLARE",
        "scadenza": "2025-02-01",
        "valid": false
      },
      "pec": {
        "value": "info@rossi.pec.it",
        "valid": true
      },
      "concessioni": {
        "count": 2,
        "attive": 2
      }
    },
    "warnings": [
      {
        "type": "durc",
        "message": "DURC in scadenza tra 25 giorni",
        "severity": "warning",
        "data_scadenza": "2025-02-01"
      }
    ],
    "errors": []
  }
}
```

---

## 6. Procedura Deploy

### 6.1. Prerequisiti

- Accesso SSH al server Hetzner (root@157.90.29.66)
- Database Neon PostgreSQL configurato
- Variabile ambiente `POSTGRES_URL` impostata

### 6.2. Step Deploy Backend

> ✅ **Status:** Deploy completato il 07/12/2025. Backend attivo su porta 3000.

**1. Connessione SSH:**

```bash
ssh root@157.90.29.66
```

**2. Pull codice aggiornato:**

```bash
cd /root/mihub-backend-rest
git pull origin master
```

**3. Restart server PM2:**

```bash
pm2 restart mihub-backend
pm2 logs mihub-backend --lines 20
```

**4. Verifica health:**

```bash
curl http://localhost:3000/health | jq
```

### 6.3. Step Migration Database

**Eseguire UNA SOLA VOLTA:**

```bash
curl -X POST http://157.90.29.66:3000/api/admin/migrate-pdnd
```

**Verifica risposta:**

```json
{
  "success": true,
  "message": "PDND Migration completed: imprese expanded, dms_durc_snapshots and dms_suap_instances created"
}
```

**Controllo log:**

Dashboard → Tab "LOGS" → Tab "Imprese API"

---

## 7. Testing

### 7.1. Test Endpoint Imprese

**Lista imprese:**

```bash
curl http://157.90.29.66:3000/api/imprese | jq
```

**Verifica campi PDND:**

```bash
curl http://157.90.29.66:3000/api/imprese/1 | jq '.data | keys'
```

**Output atteso:**

```json
[
  "id",
  "denominazione",
  "partita_iva",
  "codice_fiscale",
  "numero_rea",
  "cciaa_sigla",
  "forma_giuridica",
  "stato_impresa",
  "indirizzo_via",
  "indirizzo_civico",
  "indirizzo_cap",
  "indirizzo_provincia",
  "indirizzo_stato",
  "pec",
  "codice_ateco",
  "descrizione_ateco",
  "data_iscrizione_ri",
  "flag_impresa_artigiana",
  "created_at",
  "updated_at"
]
```

### 7.2. Test Endpoint Rating

**Calcola rating:**

```bash
curl http://157.90.29.66:3000/api/imprese/1/rating | jq
```

**Verifica semaforo:**

```bash
curl http://157.90.29.66:3000/api/imprese/1/rating | jq '.data.rating.status'
```

**Output possibili:** `"VERDE"`, `"GIALLO"`, `"ROSSO"`

### 7.3. Test Endpoint DURC

**Salva snapshot DURC:**

```bash
curl -X POST http://157.90.29.66:3000/api/qualificazioni/durc \
  -H "Content-Type: application/json" \
  -d '{
    "impresa_id": 1,
    "protocollo_inps": "DURC-2025-TEST-001",
    "esito_regolarita": "REGOLARE",
    "data_scadenza": "2025-06-01"
  }' | jq
```

**Verifica storico:**

```bash
curl http://157.90.29.66:3000/api/qualificazioni/durc/1 | jq
```

### 7.4. Test Endpoint SUAP

**Crea istanza SUAP:**

```bash
curl -X POST http://157.90.29.66:3000/api/qualificazioni/suap \
  -H "Content-Type: application/json" \
  -d '{
    "impresa_id": 1,
    "market_id": 1,
    "cui": "SUAP-2025-GR001-TEST",
    "tipo_pratica": "APERTURA",
    "stato_pratica": "ACCOLTA",
    "data_scadenza_concessione": "2026-12-31"
  }' | jq
```

**Verifica istanze:**

```bash
curl http://157.90.29.66:3000/api/qualificazioni/suap/1 | jq
```

---

## 8. Monitoraggio

### 8.1. Sezione LOGS

**Percorso:** Dashboard → Tab "LOGS" → Tab "Imprese API"

**Funzionalità:**
- Visualizza tutti i log delle chiamate API a Imprese e Qualificazioni
- Filtra automaticamente solo endpoint `/api/imprese*` e `/api/qualificazioni*`
- Mostra errori in evidenza (sfondo rosso)
- Refresh automatico ogni 10 secondi

**Endpoint monitorati:**
- GET /api/imprese
- GET /api/imprese/:id
- GET /api/qualificazioni
- GET /api/imprese/:id/qualificazioni
- POST /api/admin/migrate-pdnd

### 8.2. Sezione DEBUG

**Percorso:** Dashboard → Tab "DEBUG & DEV" → Card "System Health Check"

**Funzionalità:**
- Lista tutti gli endpoint disponibili con status "Ready"
- Verifica visiva rapida che gli endpoint siano configurati

**Output:**

```
$ Imprese & Qualificazioni API Status...
✓ GET /api/imprese - Ready
✓ GET /api/imprese/:id - Ready
✓ GET /api/qualificazioni - Ready
✓ GET /api/imprese/:id/qualificazioni - Ready

$ Admin Migration API Status...
✓ POST /api/admin/migrate-pdnd - Ready
```

### 8.3. Sezione INTEGRAZIONI

**Percorso:** Dashboard → Tab "Integrazioni" → Tab "API Dashboard" → Categoria "MIO Agent"

**Funzionalità:**
- Test interattivo degli endpoint
- Input JSON per parametri
- Visualizzazione risposta formattata
- Misurazione tempo di risposta

**Esempio test:**

1. Trova endpoint `/api/imprese/:id/rating`
2. Inserisci Request Body: `{"id": 1}`
3. Clicca **Test ▶️**
4. Verifica risposta JSON con semaforo

---

## 9. Roadmap

### 9.1. FASE 3: Frontend Anagrafica Master

**Obiettivo:** Espandere form "Nuova Impresa" in Gestione Mercati

**Task:**
- [ ] Modificare `ImpreseQualificazioniPanel.tsx`
- [ ] Creare modale con 3 gruppi campi (Identità, Sede, Contatti)
- [ ] Validazione campi obbligatori (denominazione, P.IVA, PEC)
- [ ] Dropdown forma giuridica (SRL, SNC, DI, etc.)
- [ ] Autocomplete comune/provincia
- [ ] Integrazione chiamate API POST/PUT

**Stima:** 2-3 giorni

### 9.2. FASE 4: Frontend Rating & Compliance

**Obiettivo:** Implementare semaforo in Dashboard Qualificazioni

**Task:**
- [ ] Creare componente `RatingBadge.tsx` (🟢🟡🔴)
- [ ] Card "Stato Conformità" con dettagli warnings/errors
- [ ] Modale breakdown completo al click su semaforo
- [ ] Integrazione chiamata API `GET /api/imprese/:id/rating`
- [ ] Refresh automatico rating ogni 5 minuti
- [ ] Notifiche push per cambio stato (VERDE → GIALLO → ROSSO)

**Stima:** 3-4 giorni

### 9.3. FASE 5: Integrazione PDND Reale

**Obiettivo:** Collegare API PDND per verifica DURC automatica

**Task:**
- [ ] Configurare credenziali PDND
- [ ] Implementare client API PDND
- [ ] Endpoint `/api/pdnd/verify-durc/:partita_iva`
- [ ] Salvataggio automatico snapshot in `dms_durc_snapshots`
- [ ] Scheduler verifica DURC ogni 24h
- [ ] Alert email per DURC in scadenza

**Stima:** 5-7 giorni

---

## Appendice A: Commit History

### Backend (mihub-backend-rest)

1. **b503f3d** - "feat: FASE 1 - Database upgrade PDND con endpoint migration"
2. **16c0ef1** - "feat: FASE 2 - Backend API completo con imprese PDND e endpoint rating"

### Frontend (dms-hub-app-new)

1. **f3b1ccc** - "feat: Aggiunto monitoraggio endpoint migrate-pdnd in Logs e Debug"

### MIO-hub

1. **8de8b61** - "feat: Aggiunto endpoint migrate-pdnd per FASE 1 MASTER PLAN"

---

## Appendice B: Riferimenti

- **MASTER SYSTEM PLAN:** `/01_architettura/MASTER_SYSTEM_PLAN.md`
- **Schema Database:** Sezione 2.3 del MASTER SYSTEM PLAN
- **API Documentation:** MIO-hub/api/index.json
- **Report FASE 1 & 2:** `/reports/REPORT_FASE1_FASE2_COMPLETATE.md`

---

**Documento generato:** 7 Dicembre 2025  
**Autore:** Manus AI Agent  
**Versione:** 1.0
