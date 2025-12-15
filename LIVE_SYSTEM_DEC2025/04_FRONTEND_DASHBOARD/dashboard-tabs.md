# Dashboard PA - Tabs & Features

> **Documentazione completa di tutti i tab della Dashboard PA**  
> Deploy: `https://dms-hub-app-new.vercel.app/dashboard-pa`  
> Repository: `Chcndr/dms-hub-app-new`

## 📋 Indice

- [Overview Tabs](#overview-tabs)
- [Tabs Operativi (LIVE)](#tabs-operativi-live)
- [Tabs In Sviluppo](#tabs-in-sviluppo)
- [Tabs Pianificati (Roadmap)](#tabs-pianificati-roadmap)
- [Componenti Condivisi](#componenti-condivisi)

---

## 🎯 Overview Tabs

La Dashboard PA è organizzata in **27 tabs** che coprono tutte le funzionalità del sistema DMS Hub. I tab sono raggruppati per area funzionale.

### Stato Sviluppo

| Stato | Descrizione | Numero Tabs |
|-------|-------------|-------------|
| ✅ **LIVE** | Funzionante e operativo | 12 |
| 🚧 **IN SVILUPPO** | Parzialmente implementato | 8 |
| 📋 **PIANIFICATO** | Da sviluppare | 7 |

---

## ✅ Tabs Operativi (LIVE)

### TAB 8: Logs

**Value**: `logs`  
**Stato**: ✅ **LIVE & FUNZIONANTE**  
**Componente**: `GuardianLogsSection`

**Descrizione**: Visualizza i logs del sistema Guardian con 4 sub-tabs:

#### Sub-Tabs:

1. **Tutti i Log**
   - Endpoint: `GET /api/mihub/logs?limit=100`
   - Mostra tutti i 573 logs del sistema
   - Filtri: agent, date range, success
   - Tabella con: Timestamp, Agent, Service ID, Risk Level, Status, Message

2. **System Logs**
   - Endpoint: `GET /api/system-logs`
   - Logs di sistema (non agenti)
   - Attualmente 0 logs (normale)

3. **Guardian Logs**
   - Endpoint: `GET /api/logs/guardian?limit=50`
   - Logs specifici degli agenti Guardian
   - Mostra 72 logs degli agenti (mio, manus, abacus, gptdev, zapier)
   - Aggiornamento automatico ogni 10s

4. **Imprese API**
   - Documentazione endpoint API imprese
   - Mostra: `GET /api/imprese`, `GET /api/imprese/:id`, `GET /api/qualificazioni`

**Features**:
- ✅ Caricamento real-time
- ✅ Filtri avanzati
- ✅ Paginazione
- ✅ Export logs (pianificato)

**File Sorgente**: `client/src/components/GuardianLogsSection.tsx`

---

### TAB 24: MIO Agent

**Value**: `mio`  
**Stato**: ✅ **LIVE & FUNZIONANTE**  
**Componente**: Custom (inline in DashboardPA.tsx)

**Descrizione**: Interfaccia principale per interagire con l'agente MIO e coordinare gli altri agenti.

#### Sezioni:

**A. Chat Principale MIO** (sempre visibile)
- Chat diretta con MIO (GPT-5 Coordinatore)
- Input messaggio + invio
- Visualizzazione conversazione con timestamp (HH:MM formato italiano)
- Label messaggi: "Tu" (utente), "MIO" (orchestrator), "AGENTE" (nome agente)
- Pulsante STOP per interrompere elaborazione in corso
- Endpoint: `POST /api/mihub/orchestrator`
- Context: `MioContext` per gestione stato condiviso

**B. Chat Multi-Agente** (sezione espandibile)

**Default**: Vista 4 Agenti (griglia) - mostrata all'apertura della pagina

**Toggle**: Vista Singola / Vista 4 Agenti

**Vista Singola**:
- Conversazione diretta con UN singolo agente alla volta
- Agenti disponibili: GPT Dev (Sviluppatore), Manus (Esecutivo), Abacus (Analisi), Zapier (Automazioni)
- Features:
  - Chat dedicata full-screen
  - Timestamp (HH:MM) su ogni messaggio
  - Label: "Tu" (utente), "da MIO" (delegazione orchestrator), "[nome agente]" (risposta agente)
  - Pulsante STOP nell'header per interrompere elaborazione
  - Contatore messaggi nell'header
  - Conversation ID: `{agent}-single` (es. `gptdev-single`)
  - Role: `user` (utente parla direttamente all'agente)

**Vista 4 Agenti** (griglia):
- 4 mini-chat in griglia 2x2 per monitoraggio simultaneo
- Agenti: GPT Dev, Manus, Abacus, Zapier
- Features per ogni mini-chat:
  - Visualizzazione ultimi messaggi
  - Timestamp (HH:MM) già presente da prima
  - Input dedicato per inviare messaggi diretti
  - Scroll automatico agli ultimi messaggi
  - Conversation ID: `mio-{agent}-quad` (es. `mio-gptdev-quad`)
  - Role: `user` (utente può interagire direttamente)
- Uso: Coordinamento multi-agente, monitoraggio parallelo task

**C. Attività Agenti Recente (Guardian)**
- Sezione in basso che mostra ultimi 50 eventi agenti
- Endpoint: `GET /api/guardian/logs?limit=50`
- Aggiornamento automatico ogni 10s
- Mostra: Agent, Status (ALLOWED/BLOCKED), Endpoint, Response Time

**D. System Health Check**
- Stato endpoint Guardian API
- Verifica: `/api/logs/initSchema`, `/api/logs/createLog`, `/api/logs/getLogs`, `/api/logs/stats`, `/api/guardian/health`

**Features**:
- ✅ Chat multi-agente funzionante
- ✅ Toggle Vista Singola / Vista 4 Agenti
- ✅ Vista 4 Agenti come default all'apertura
- ✅ Timestamp (HH:MM) in tutti i messaggi
- ✅ Label messaggi corrette (Tu/MIO/Agente)
- ✅ Pulsante STOP in Chat Principale e Vista Singola
- ✅ Logs real-time
- ✅ Health monitoring
- ✅ Conversation persistence (localStorage)
- ✅ Optimistic UI per messaggi utente

**File Sorgente**: 
- `client/src/pages/DashboardPA.tsx` (linee 3865-4570)
- `client/src/contexts/MioContext.tsx` (gestione stato MIO)
- `client/src/components/multi-agent/MultiAgentChatView.tsx` (Vista 4 Agenti)
- `client/src/hooks/useAgentLogs.tsx` (caricamento messaggi agenti)
- `client/src/hooks/useConversationPersistence.tsx` (persistenza conversazioni)

**Changelog Recenti** (Dicembre 2024):
- **14/12/2024**: Fix timestamp Chat Principale MIO + Vista 4 Agenti default
- **14/12/2024**: Fix label messaggi (MIO/Tu/Agente) in tutte le viste
- **14/12/2024**: Aggiunto pulsante STOP in Vista Singola Agente
- **14/12/2024**: Rimosso reset viewMode che causava vista singola all'apertura

---

### TAB 11: Debug & Dev

**Value**: `debug`  
**Stato**: ✅ **LIVE & FUNZIONANTE**  
**Componente**: `DebugSectionReal`

**Descrizione**: Pannello di debug e sviluppo per monitorare lo stato del sistema.

**Features**:
- ✅ Visualizzazione logs real-time
- ✅ Test endpoint API
- ✅ Monitoring performance
- ✅ Error tracking

**File Sorgente**: `client/src/components/DebugSectionReal.tsx`

---

### TAB 20: Integrazioni

**Value**: `integrations`  
**Stato**: ✅ **LIVE & FUNZIONANTE**  
**Componente**: `Integrazioni`

**Descrizione**: Gestione integrazioni con servizi esterni (Zapier, GitHub, PDND, SPID).

**Features**:
- ✅ Lista integrazioni attive
- ✅ Configurazione webhook
- ✅ Test connessioni
- ✅ Logs integrazioni

**File Sorgente**: `client/src/components/Integrazioni.tsx`

---

### TAB 22: Gestione Mercati

**Value**: `mercati`  
**Stato**: ✅ **LIVE & FUNZIONANTE**  
**Componente**: `GestioneMercati`

**Descrizione**: CRUD completo per gestione mercati rionali.

**Features**:
- ✅ Lista mercati
- ✅ Aggiungi/Modifica/Elimina mercato
- ✅ Visualizzazione mappa (GIS)
- ✅ Gestione posteggi

**Endpoint**:
- `GET /api/markets` - Lista mercati
- `POST /api/markets` - Crea mercato
- `PUT /api/markets/:id` - Aggiorna mercato
- `DELETE /api/markets/:id` - Elimina mercato

**File Sorgente**: `client/src/components/GestioneMercati.tsx`

---

### TAB 23: Imprese & Qualificazioni

**Value**: `imprese`  
**Stato**: ✅ **LIVE & FUNZIONANTE**  
**Componente**: `ImpreseQualificazioniPanel`

**Descrizione**: Gestione imprese registrate e qualificazioni.

**Features**:
- ✅ Lista imprese con filtri
- ✅ Dettaglio impresa
- ✅ Gestione qualificazioni
- ✅ Associazione impresa-qualificazione

**Endpoint**:
- `GET /api/imprese` - Lista imprese
- `GET /api/imprese/:id` - Dettaglio impresa
- `GET /api/qualificazioni` - Lista qualificazioni

**File Sorgente**: `client/src/components/ImpreseQualificazioniPanel.tsx`

---

### TAB 1: Overview

**Value**: `overview`  
**Stato**: ✅ **LIVE** (dati mock)

**Descrizione**: Dashboard principale con KPI e metriche aggregate.

**Sezioni**:
- Crescita utenti (grafico trend)
- Statistiche mercati
- Revenue overview
- Attività recente

**Nota**: Attualmente usa dati mock. Da collegare a API reali.

---

### TAB 3: Mercati

**Value**: `markets`  
**Stato**: ✅ **LIVE** (parziale)

**Descrizione**: Visualizzazione mercati con mappa GIS.

**Features**:
- ✅ Mappa interattiva Mercato Grosseto (GIS ufficiale)
- ✅ Lista mercati
- 🚧 Filtri avanzati (in sviluppo)
- 🚧 Statistiche posteggi (in sviluppo)

**Endpoint**: `GET /api/gis/markets`

---

### TAB 9: Agente AI

**Value**: `ai`  
**Stato**: ✅ **LIVE**

**Descrizione**: Chat con Agente AI assistente PA (diverso da MIO Agent).

**Features**:
- ✅ Chat interattiva
- ✅ Risposta automatica
- 🚧 Integrazione con backend (in sviluppo)

---

### TAB 24: Documentazione

**Value**: `docs`  
**Stato**: ✅ **LIVE**

**Descrizione**: Documentazione progetto DMS Hub.

**Sezioni**:
- Architettura sistema
- Guide API
- Troubleshooting
- Changelog

---

### TAB 21: Impostazioni

**Value**: `settings`  
**Stato**: ✅ **LIVE** (basic)

**Descrizione**: Configurazione dashboard PA.

**Features**:
- ✅ Tema (dark/light)
- ✅ Lingua
- 🚧 Notifiche (in sviluppo)
- 🚧 Permessi utente (in sviluppo)

---

## 🚧 Tabs In Sviluppo

### TAB 2: Clienti

**Value**: `users`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Gestione utenti e clienti del sistema.

**Features Implementate**:
- ✅ Visualizzazione mezzi di trasporto (mock data)
- ✅ Statistiche base

**Da Sviluppare**:
- ❌ Lista utenti reale
- ❌ Dettaglio utente
- ❌ CRUD utenti
- ❌ Integrazione con database

**Endpoint Necessari**:
- `GET /api/users` - Lista utenti
- `GET /api/users/:id` - Dettaglio utente
- `POST /api/users` - Crea utente

---

### TAB 4: Prodotti

**Value**: `products`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Catalogo prodotti venduti nei mercati.

**Features Implementate**:
- ✅ Visualizzazione categorie (mock data)
- ✅ Statistiche vendite (mock data)

**Da Sviluppare**:
- ❌ CRUD prodotti
- ❌ Associazione prodotto-venditore
- ❌ Gestione prezzi
- ❌ Inventario

**Endpoint Necessari**:
- `GET /api/products` - Lista prodotti
- `POST /api/products` - Crea prodotto
- `PUT /api/products/:id` - Aggiorna prodotto

---

### TAB 5: Sostenibilità

**Value**: `sustainability`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Metriche sostenibilità ambientale.

**Features Implementate**:
- ✅ Rating sostenibilità (mock data)
- ✅ Grafici emissioni CO2 (mock data)

**Da Sviluppare**:
- ❌ Calcolo reale emissioni
- ❌ Integrazione sensori IoT
- ❌ Report sostenibilità

---

### TAB 6: TPAS

**Value**: `tpas`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Gestione TPAS (Tessera Punti Acquisto Sostenibile).

**Features Implementate**:
- ✅ Visualizzazione e-commerce vs fisico (mock data)
- ✅ Statistiche punti (mock data)

**Da Sviluppare**:
- ❌ Sistema punti reale
- ❌ Redemption premi
- ❌ Integrazione wallet

---

### TAB 7: Carbon Credits

**Value**: `carboncredits`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Gestione crediti carbonio.

**Features Implementate**:
- ✅ Fondo liquidità (mock data)
- ✅ Visualizzazione crediti (mock data)

**Da Sviluppare**:
- ❌ Marketplace crediti
- ❌ Transazioni blockchain
- ❌ Certificazione crediti

---

### TAB 8: Real-Time

**Value**: `realtime`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Monitoraggio attività real-time.

**Features Implementate**:
- ✅ Visualizzazione attività (mock data)
- ✅ Animazioni real-time

**Da Sviluppare**:
- ❌ WebSocket per dati real-time
- ❌ Notifiche push
- ❌ Streaming eventi

---

### TAB 10: Sicurezza

**Value**: `security`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Monitoring sicurezza e accessi.

**Features Implementate**:
- ✅ Statistiche accessi (mock data)
- ✅ Visualizzazione tentativi falliti (mock data)

**Da Sviluppare**:
- ❌ Logs accessi reali
- ❌ Audit trail
- ❌ Gestione permessi

---

### TAB 13: Qualificazione Imprese

**Value**: `businesses`  
**Stato**: 🚧 **IN SVILUPPO**

**Descrizione**: Gestione conformità e qualificazioni imprese.

**Features Implementate**:
- ✅ KPI conformità (mock data)
- ✅ Visualizzazione stato qualificazioni (mock data)

**Da Sviluppare**:
- ❌ Workflow approvazione
- ❌ Documenti allegati
- ❌ Notifiche scadenze

---

## 📋 Tabs Pianificati (Roadmap)

### TAB 14: Segnalazioni & IoT

**Value**: `civic`  
**Stato**: 📋 **PIANIFICATO**

**Descrizione**: Gestione segnalazioni civiche e sensori IoT.

**Features Pianificate**:
- Mappa segnalazioni
- Integrazione sensori
- Workflow gestione segnalazioni
- Dashboard IoT

---

### TAB 15: Utenti Imprese

**Value**: `business-users`  
**Stato**: 📋 **PIANIFICATO**

**Descrizione**: Gestione utenti delle imprese registrate.

**Features Pianificate**:
- Lista utenti imprese
- Permessi e ruoli
- Attività utenti

---

### TAB 16: Controlli/Sanzioni

**Value**: `inspections`  
**Stato**: 📋 **PIANIFICATO**

**Descrizione**: Gestione controlli e sanzioni.

**Features Pianificate**:
- Programmazione controlli
- Verbali digitali
- Gestione sanzioni
- Report conformità

---

### TAB 17: Notifiche

**Value**: `notifications`  
**Stato**: 📋 **PIANIFICATO**

**Descrizione**: Centro notifiche sistema.

**Features Pianificate**:
- Notifiche push
- Email automatiche
- SMS alerts
- Preferenze notifiche

---

### TAB 18: Centro Mobilità

**Value**: `mobility`  
**Stato**: 📋 **PIANIFICATO**

**Descrizione**: Integrazione trasporti pubblici (TPER Bologna).

**Features Pianificate**:
- Orari trasporti real-time
- Pianificatore percorsi
- Integrazione biglietteria
- Statistiche mobilità

---

### TAB 19: Report

**Value**: `reports`  
**Stato**: 📋 **PIANIFICATO**

**Descrizione**: Generazione report e export dati.

**Features Pianificate**:
- Report personalizzabili
- Export PDF/Excel
- Scheduling report automatici
- Dashboard analytics

---

### TAB 25-27: Da Definire

**Stato**: 📋 **PIANIFICATO**

Slot riservati per future funzionalità.

---

## 🧩 Componenti Condivisi

### GuardianLogsSection

**Path**: `client/src/components/GuardianLogsSection.tsx`

**Descrizione**: Componente per visualizzazione logs Guardian con 4 sub-tabs.

**Props**:
```typescript
interface GuardianLogsSectionProps {
  // Nessuna prop, componente standalone
}
```

**Features**:
- Tabs interni (Tutti i Log, System Logs, Guardian Logs, Imprese API)
- Caricamento dati da API
- Filtri e paginazione
- Refresh automatico

---

### DebugSectionReal

**Path**: `client/src/components/DebugSectionReal.tsx`

**Descrizione**: Pannello debug per sviluppatori.

**Features**:
- Console logs
- API testing
- Performance monitoring

---

### Integrazioni

**Path**: `client/src/components/Integrazioni.tsx`

**Descrizione**: Gestione integrazioni esterne.

**Features**:
- Lista integrazioni
- Configurazione webhook
- Test connessioni

---

### GestioneMercati

**Path**: `client/src/components/GestioneMercati.tsx`

**Descrizione**: CRUD mercati con mappa GIS.

**Features**:
- Form creazione/modifica mercato
- Mappa interattiva
- Gestione posteggi

---

### ImpreseQualificazioniPanel

**Path**: `client/src/components/ImpreseQualificazioniPanel.tsx`

**Descrizione**: Gestione imprese e qualificazioni.

**Features**:
- Lista imprese con filtri
- Dettaglio impresa
- Associazione qualificazioni

---

## 📊 Riepilogo Stato Sviluppo

### Tabs per Stato

| Stato | Numero | Percentuale |
|-------|--------|-------------|
| ✅ LIVE | 12 | 44% |
| 🚧 IN SVILUPPO | 8 | 30% |
| 📋 PIANIFICATO | 7 | 26% |
| **TOTALE** | **27** | **100%** |

### Priorità Sviluppo

**Alta Priorità** (Q1 2025):
1. ✅ ~~TAB 8: Logs~~ (COMPLETATO)
2. ✅ ~~TAB 24: MIO Agent~~ (COMPLETATO)
3. 🚧 TAB 2: Clienti (in sviluppo)
4. 🚧 TAB 4: Prodotti (in sviluppo)

**Media Priorità** (Q2 2025):
5. 🚧 TAB 5: Sostenibilità
6. 🚧 TAB 6: TPAS
7. 📋 TAB 14: Segnalazioni & IoT
8. 📋 TAB 16: Controlli/Sanzioni

**Bassa Priorità** (Q3-Q4 2025):
9. 📋 TAB 7: Carbon Credits
10. 📋 TAB 18: Centro Mobilità
11. 📋 TAB 19: Report

---

## 🔄 Flusso Dati

### Architettura Frontend

```
User Interface (Dashboard PA)
    ↓
React Components (Tabs)
    ↓
API Calls (fetch/axios)
    ↓
Backend REST API (Hetzner)
    ↓
Database (Neon PostgreSQL)
```

### State Management

**Attuale**: React Hooks (useState, useEffect)

**Pianificato**: Migrazione a Zustand o Redux per state management globale

---

## 📝 Note Sviluppo

### Convenzioni Codice

**Naming Tabs**:
- Value: lowercase, kebab-case (es. `business-users`)
- Title: Title Case (es. "Utenti Imprese")

**Naming Componenti**:
- PascalCase (es. `GuardianLogsSection`)
- File: PascalCase.tsx (es. `GuardianLogsSection.tsx`)

**Styling**:
- Tailwind CSS per utility classes
- Tema dark custom (`bg-[#1a2332]`, `text-[#e8fbff]`)
- Colori accent: teal (`#14b8a6`), purple (`#8b5cf6`)

### Best Practices

**Componenti**:
- Separare logica da presentazione
- Usare TypeScript per type safety
- Props tipizzate con interface

**API Calls**:
- Gestire loading state
- Gestire error state
- Implementare retry logic

**Performance**:
- Lazy loading per tab non visitati
- Memoization per componenti pesanti
- Debounce per input search

---

**Ultimo Aggiornamento**: 11 Dicembre 2025  
**Versione Dashboard**: 2.0.0  
**Framework**: React 18 + Vite + TypeScript  
**Deploy**: Vercel
