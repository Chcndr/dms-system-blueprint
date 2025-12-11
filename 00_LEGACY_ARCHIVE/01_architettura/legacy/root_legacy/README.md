# DMS System Blueprint

**Repository ufficiale** per l'architettura e la documentazione del sistema DMS / MIO-HUB per la gestione dei mercati ambulanti.

Questo repository è la **fonte di verità ufficiale** per:
- Architettura AS-IS (stato attuale del sistema)
- Architettura TO-BE (progetto finale del sistema)
- Mapping tra sistemi legacy e nuovo sistema
- Roadmap tecnica e decisioni implementative

---

## 📘 Documenti Ufficiali

### 1. **[MASTER SYSTEM PLAN](docs/MASTER_SYSTEM_PLAN.md)**

Questo è il **documento principale** che unifica visione, architettura, piano di migrazione e integrazione Pepe GIS.

### 2. **[Implementation Plan - Checklist Operativa](docs/Implementation%20Plan.md)**

Questo è il **backlog operativo** con le fasi di implementazione e i task concreti.

### 3. **[Report Stato Lavori](docs/REPORT_STATO_LAVORI.md)**

Questo è lo **stato corrente del progetto**, cosa è pronto e cosa manca.

### 4. **[Integrazione Pepe GIS / Editor v3](docs/Integrazione%20Pepe%20GIS%20_%20Editor%20v3.md)**

Questo documento descrive la **prima integrazione da completare** (PRIORITÀ ASSOLUTA).

---

## Struttura del Repository

```
dms-system-blueprint/
├── docs/
│   ├── 00-MASTER_SYSTEM_PLAN.md           # 📘 DOCUMENTO PRINCIPALE
│   ├── Implementation Plan.md              # 📋 BACKLOG OPERATIVO
│   ├── REPORT_STATO_LAVORI.md              # 📊 STATO CORRENTE
│   ├── Integrazione Pepe GIS _ Editor v3.md # 🗺️ PRIMA INTEGRAZIONE
│   ├── Schema DB Pepe GIS.md               # Schema DB completo
│   ├── API Pepe GIS.md                     # Documentazione API
│   ├── MASTER SYSTEM PLAN v2.md            # Piano completo (versione estesa)
│   ├── AS-IS/                              # Analisi dello stato attuale
│   └── TO-BE/                              # Progetto finale del sistema
└── references/                             # File di riferimento
    ├── SYSTEM_BLUEPRINT_AS_IS.md
    ├── ANALISI_DMS_LEGACY_COMPLETA.md
    ├── pepe-gis/
    │   └── editor-v3-sample.json           # JSON di esempio dall'Editor v3
    └── screenshot_*.webp
```

---

## Decisioni Architetturali Non Negoziabili

1. **Database Unico**: PostgreSQL su Neon (niente più PlanetScale/MySQL)
2. **DMS Core API** (Hetzner): Unico backend, tutta la business logic vive qui
3. **Frontend** (Next.js su Vercel): Solo UI, parla SOLO con DMS Core API, niente accesso diretto a DB
4. **DMS Legacy** (Heroku): Sistema esterno da integrare e poi dismettere
5. **Pepe GIS**: Priorità assoluta per la gestione delle mappe dei mercati

---

## Roadmap (Sintesi)

- **Fase 0**: Base tecnica pulita ✅ COMPLETATA
- **Fase 1**: Integrazione Pepe GIS ✅ IMPLEMENTATO (Branch: feature/gis-market-map-v1, In attesa di deploy)
- **Fase 2**: Copia e allineamento del gestionale DMS (Heroku)
- **Fase 3**: Nuovo gestionale dentro la Dashboard / MIO-HUB
- **Fase 4**: Feature avanzate (CO₂, wallet, automazioni)

*Per i dettagli completi, vedere [Implementation Plan](docs/Implementation%20Plan.md).*

---

## Sistemi Coinvolti

| Sistema | Hosting | Ruolo | Stato |
|---------|---------|-------|-------|
| **DMS Core API** | Hetzner | Backend principale, business logic | TO-BE |
| **Dashboard / MIO-HUB** | Vercel | Interfaccia web amministrativa | TO-BE |
| **DMS Legacy** | Heroku | Gestionale esistente | AS-IS (da integrare) |
| **Database** | Neon | PostgreSQL unico | TO-BE |
| **Pepe GIS** | - | Motore mappe mercati | TO-BE (Fase 1) |

---

## Come Usare Questo Repository

Questo repository è il **manuale di bordo del sistema**. Deve essere aggiornato man mano che:
- Si analizza il gestionale DMS legacy
- Si scoprono dettagli tecnici
- Si modifica architettura e codice

**Regola d'oro**: Se cambi una decisione tecnica importante:
1. Aggiorna prima il `00-MASTER_SYSTEM_PLAN.md`
2. Poi adegua il codice
3. Aggiorna il `REPORT_STATO_LAVORI.md`

---

## 📝 Documenti Aggiuntivi

### **CREDENZIALI_E_ACCESSI.md** 🔒
Documento sensibile con tutte le credenziali e accessi:
- Server Hetzner (SSH, PM2, Nginx)
- Database PostgreSQL (Neon)
- Frontend Vercel
- Repository GitHub
- Endpoint API
- Comandi utili e checklist deploy

**⚠️ ATTENZIONE:** Proteggere adeguatamente questo file.

### **REPORT_STATO_MAPPA_GIS_v2_21Nov2025.md**
Report completo dell'integrazione Pepe GIS / Editor v3:
- Implementazione endpoint backend `GET /api/gis/market-map`
- Implementazione pagina frontend `/market-gis`
- Test eseguiti e risultati
- Stato attuale: ✅ Implementato e testato
- Prossimi passi per deploy

---

## 🚀 Stato Implementazione GIS (Aggiornamento 21 Nov 2025)

### ✅ Completato
- **Endpoint Backend**: `GET /api/gis/market-map` - Serve dati GeoJSON da Editor v3
- **Pagina Frontend**: `/market-gis` - Visualizzazione interattiva con Leaflet
- **Componenti**: Contorno mercato (Polygon) + Piazzole (Circle) + Popup dettagli
- **Build**: Backend e frontend senza errori
- **Test**: End-to-end superati

### ⏸️ In Attesa
- Deploy backend su Hetzner
- Deploy frontend su Vercel
- Test in produzione

### 💾 Repository e Branch
- **Backend**: `Chcndr/mihub` - Branch `feature/gis-market-map-v1`
- **Frontend**: `Chcndr/dms-hub-app-new` - Branch `feature/gis-market-map-v1`
- **Blueprint**: `Chcndr/dms-system-blueprint` - Branch `main`

---

## Contatti e Manutenzione

Questo repository è mantenuto come documentazione vivente del progetto DMS / MIO-HUB.

Per domande o chiarimenti, fare riferimento ai documenti ufficiali.

**Ultimo aggiornamento:** 21 Novembre 2025 - 18:25 GMT+1  
**Maintainer:** Manus AI
