# DMS System Blueprint - MIO Hub

> **📚 Biblioteca Tecnica Completa del Sistema MIO Hub**  
> Documentazione aggiornata al sistema funzionante in produzione

## 🎯 Cos'è questo Repository

Questo repository è il **"Cervello"** del progetto MIO Hub. Contiene tutta la documentazione tecnica, architetturale e operativa del sistema Digital Market System (DMS) basato su orchestrazione multi-agente AI.

**Operazione "Specchio Reale"** (11 Dicembre 2025): Questo repository è stato completamente riorganizzato per riflettere **esattamente** il sistema funzionante in produzione, separando documentazione legacy da sistema live.

---

## 📂 Struttura Repository

```
dms-system-blueprint/
│
├── LIVE_SYSTEM_DEC2025/          ✅ SISTEMA FUNZIONANTE
│   ├── 01_ARCHITECTURE/          → Architettura "8 Isole"
│   ├── 02_BACKEND_CORE/          → API, LLM Engine, Tools
│   ├── 03_DATABASE_SCHEMA/       → Schema PostgreSQL Neon
│   └── 04_FRONTEND_DASHBOARD/    → Dashboard PA (27 tabs)
│
├── ROADMAP_2025/                 🗺️ SVILUPPI FUTURI
│   ├── Q1_2025/                  → Gen-Mar 2025
│   ├── Q2_2025/                  → Apr-Giu 2025
│   └── Q3_Q4_2025/               → Lug-Dic 2025
│
└── 00_LEGACY_ARCHIVE/            📦 ARCHIVIO STORICO
    ├── 01_architettura/          → Documentazione vecchia
    ├── 02_backend/               → Guide legacy
    └── docs/                     → Documentazione teorica
```

---

## 🚀 Quick Start

### Per AI Agents (come MIO)

Se sei un AI Agent e devi capire come funziona il sistema:

1. **Inizia da qui**: [LIVE_SYSTEM_DEC2025/README.md](./LIVE_SYSTEM_DEC2025/README.md)
2. **Studia l'architettura**: [01_ARCHITECTURE/system-overview.md](./LIVE_SYSTEM_DEC2025/01_ARCHITECTURE/system-overview.md)
3. **Impara le API**: [02_BACKEND_CORE/api-map.md](./LIVE_SYSTEM_DEC2025/02_BACKEND_CORE/api-map.md)
4. **Comprendi il database**: [03_DATABASE_SCHEMA/schema.md](./LIVE_SYSTEM_DEC2025/03_DATABASE_SCHEMA/schema.md)

### Per Sviluppatori

Se sei uno sviluppatore umano:

1. **Leggi la panoramica**: [LIVE_SYSTEM_DEC2025/README.md](./LIVE_SYSTEM_DEC2025/README.md)
2. **Setup ambiente**: Vedi guide in `00_LEGACY_ARCHIVE/07_guide_operative/`
3. **Consulta la roadmap**: [ROADMAP_2025/README.md](./ROADMAP_2025/README.md)
4. **Contribuisci**: Vedi [CONTRIBUTING.md](#contributing) (da creare)

---

## 🏗️ Sistema MIO Hub

### Panoramica

Il **MIO Hub** è un sistema multi-agente orchestrato che coordina 5 agenti AI specializzati per gestire il Digital Market System (DMS). Il sistema è basato su un'architettura "8 Isole" che separa le conversazioni dirette utente-agente dalle conversazioni di coordinamento backstage.

### Componenti Principali

**Backend Node.js**
- Server: Hetzner 157.90.29.66
- Runtime: Node.js 22.13.0
- Process Manager: PM2
- LLM Engine: Google Gemini 2.5-flash
- API: 30+ endpoint REST

**Database PostgreSQL**
- Provider: Neon (Serverless PostgreSQL)
- Region: EU-Central-1 (Frankfurt)
- Tabelle: 20+ (conversazioni, logs, business data)
- Logs: 573 operazioni tracciate

**Frontend Dashboard PA**
- Deploy: Vercel
- Framework: React 18 + Vite + TypeScript
- Tabs: 27 (12 live, 8 in sviluppo, 7 pianificati)
- URL: https://dms-hub-app-new.vercel.app/dashboard-pa

### Agenti AI

1. **MIO** - Orchestratore principale che coordina gli altri agenti
2. **Manus** - Operatore esecutivo (SSH, deploy, file operations)
3. **Abacus** - Analista dati (SQL, statistiche, report)
4. **GPT Dev** - Sviluppatore (code, debug, Git)
5. **Zapier** - Automazioni (webhook, integrazioni)

---

## 📊 Stato Progetto

### Metriche Live (11 Dicembre 2025)

**Backend**:
- ✅ Uptime: 99.5%
- ✅ Tempo risposta medio: 2.8s
- ✅ Success rate: 78.7% (451/573 operazioni)

**Database**:
- ✅ Logs totali: 573
- ✅ Conversazioni attive: 8 isole
- ✅ Messaggi salvati: 2000+

**Dashboard**:
- ✅ Tabs live: 12/27 (44%)
- 🚧 Tabs in sviluppo: 8/27 (30%)
- 📋 Tabs pianificati: 7/27 (26%)

### Roadmap 2025

**Q1 2025** (Gen-Mar):
- Completare TAB 2 (Clienti) e TAB 4 (Prodotti)
- Integrazione PDND
- Ottimizzazione performance (<2s response time)

**Q2 2025** (Apr-Giu):
- Completare TAB 5 (Sostenibilità) e TAB 6 (TPAS)
- Sistema segnalazioni IoT
- 1000+ utenti attivi

**Q3-Q4 2025** (Lug-Dic):
- Marketplace Carbon Credits (blockchain)
- Integrazione TPER (mobilità)
- Scalabilità 10.000+ utenti

---

## 📖 Documentazione

### LIVE_SYSTEM_DEC2025

Documentazione del sistema funzionante in produzione:

**01_ARCHITECTURE**
- `system-overview.md` - Panoramica architettura "8 Isole"
- `data-flow.md` - Flusso dati nel sistema
- `deployment.md` - Architettura deployment

**02_BACKEND_CORE**
- `api-map.md` - Mappa completa endpoint API
- `llm-engine.md` - Funzionamento LLM Engine (Gemini)
- `tools-system.md` - Sistema tools agenti

**03_DATABASE_SCHEMA**
- `schema.md` - Schema completo PostgreSQL
- `queries.md` - Query comuni e ottimizzazioni
- `migrations.md` - Storia migrazioni

**04_FRONTEND_DASHBOARD**
- `dashboard-tabs.md` - Tutti i 27 tabs con stato sviluppo
- `components.md` - Componenti riutilizzabili
- `state-management.md` - Gestione stato

### ROADMAP_2025

Piano sviluppo 2025 organizzato per quarter:

- `README.md` - Panoramica roadmap completa
- `Q1_2025/` - Obiettivi Q1 (Gen-Mar)
- `Q2_2025/` - Obiettivi Q2 (Apr-Giu)
- `Q3_Q4_2025/` - Obiettivi Q3-Q4 (Lug-Dic)

### 00_LEGACY_ARCHIVE

Documentazione storica e guide legacy:

- `01_architettura/` - Architettura teorica vecchia
- `02_backend/` - Guide backend legacy
- `07_guide_operative/` - Guide deploy e troubleshooting
- `docs/` - Documentazione varia

---

## 🔧 Setup & Deploy

### Prerequisiti

- Node.js 22.13.0+
- PostgreSQL client (per accesso database)
- SSH access al server Hetzner (per deploy backend)
- Vercel CLI (per deploy frontend)

### Backend Deploy

```bash
# SSH al server
ssh root@157.90.29.66

# Vai alla directory backend
cd /root/mihub-backend-rest

# Pull ultime modifiche
git pull origin main

# Installa dipendenze
npm install

# Restart PM2
pm2 restart mihub-backend
pm2 logs mihub-backend
```

### Frontend Deploy

```bash
# Clone repository
git clone https://github.com/Chcndr/dms-hub-app-new.git
cd dms-hub-app-new/client

# Installa dipendenze
npm install

# Build
npm run build

# Deploy Vercel (automatico su push main)
vercel --prod
```

### Database Access

```bash
# Connetti a Neon PostgreSQL
psql "postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-a2p3zcfr.eu-central-1.aws.neon.tech/neondb?sslmode=require"

# Lista tabelle
\dt

# Query esempio
SELECT COUNT(*) FROM mio_agent_logs;
```

---

## 🤝 Contributing

### Come Contribuire

1. **Fork** il repository
2. **Crea branch** per la tua feature (`git checkout -b feature/amazing-feature`)
3. **Commit** le modifiche (`git commit -m 'Add amazing feature'`)
4. **Push** al branch (`git push origin feature/amazing-feature`)
5. **Apri Pull Request**

### Convenzioni

**Commit Messages**:
- `feat:` Nuova feature
- `fix:` Bug fix
- `docs:` Documentazione
- `refactor:` Refactoring
- `test:` Test

**Branch Naming**:
- `feature/nome-feature`
- `fix/nome-bug`
- `docs/nome-doc`

---

## 📝 Changelog

### [2.0.0] - 2025-12-11 - Operazione "Specchio Reale"

**Added**:
- ✅ Documentazione LIVE_SYSTEM_DEC2025 completa
- ✅ ROADMAP_2025 organizzata per quarter
- ✅ Endpoint `/api/guardian/logs` per dashboard
- ✅ Endpoint `/api/mihub/logs` per tab "Tutti i Log"
- ✅ Sezione "Attività Agenti Recente (Guardian)" funzionante

**Changed**:
- 🔄 Riorganizzazione completa repository
- 🔄 Spostamento documentazione legacy in archivio
- 🔄 Aggiornamento README principale

**Fixed**:
- ✅ Tab "Logs" ora visualizza tutti i 573 logs
- ✅ Tab "Guardian Logs" mostra 72 logs agenti
- ✅ Sezione Guardian nella pagina MIO Agent funzionante

---

## 🔗 Link Utili

### Produzione

- **Dashboard PA**: https://dms-hub-app-new.vercel.app/dashboard-pa
- **Backend API**: https://api.mio-hub.me
- **Database**: Neon Console (richiede login)

### Repository

- **Backend**: https://github.com/Chcndr/mihub-backend-rest
- **Frontend**: https://github.com/Chcndr/dms-hub-app-new
- **Blueprint**: https://github.com/Chcndr/dms-system-blueprint

### Documentazione

- **PDND**: https://docs.pdnd.italia.it
- **Neon PostgreSQL**: https://neon.tech/docs
- **Google Gemini**: https://ai.google.dev/docs

---

## 📞 Contatti

**Team MIO Hub**
- Email: [da definire]
- Slack: [da definire]
- Issues: https://github.com/Chcndr/dms-system-blueprint/issues

---

## 📄 Licenza

[Da definire]

---

**Ultimo Aggiornamento**: 11 Dicembre 2025  
**Versione**: 2.0.0  
**Maintainer**: Team MIO Hub  
**Status**: ✅ Operativo in Produzione
