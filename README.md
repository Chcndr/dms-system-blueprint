# DMS System Blueprint

**Repository ufficiale** per l'architettura e la documentazione del sistema DMS / MIO-HUB per la gestione dei mercati ambulanti.

Questo repository è la **fonte di verità ufficiale** per:
- Architettura AS-IS (stato attuale del sistema)
- Architettura TO-BE (progetto finale del sistema)
- Mapping tra sistemi legacy e nuovo sistema
- Roadmap tecnica e decisioni implementative

---

## 📘 Documento Principale

**[MASTER SYSTEM PLAN v2 - DMS / MIO-HUB](docs/MASTER%20SYSTEM%20PLAN%20v2%20-%20DMS%20_%20MIO-HUB.md)**

Questo è il **documento unico e ufficiale** (v2) che definisce l'intero ecosistema. È il punto di riferimento per tutti gli stakeholder (umani e AI).

**Stato:** APPROVED  
**Owner:** Chcndr  
**Executor:** Manus / ABACUS

---

## Priorità Assoluta: Pepe GIS

La **Fase 1** del piano è l'integrazione di Pepe GIS nella Dashboard:
- Import della pianta del mercato dall'Editor v3
- Salvataggio geometrie piazzole in Postgres (Neon) come GeoJSON/JSONB
- API per servire la mappa alla dashboard
- Pagina "Mappa Mercato / GIS" su DMS-HUB / MIO-HUB

---

## Struttura del Repository

```
dms-system-blueprint/
├── docs/
│   ├── MASTER SYSTEM PLAN v2 - DMS _ MIO-HUB.md  # 📘 PIANO UNICO
│   ├── AS-IS/              # Analisi dello stato attuale
│   │   ├── 00-overview.md
│   │   ├── 01-sistemi-e-ambienti.md
│   │   ├── 02-dominio-funzionale.md
│   │   ├── 03-modello-dati-e-mapping.md
│   │   ├── 04-architettura-tecnica.md
│   │   ├── 05-integrazioni-esterne.md
│   │   ├── 06-roadmap.md
│   │   └── 07-analisi-codice-esistente.md
│   └── TO-BE/              # Progetto finale del sistema
│       ├── MASTER-SYSTEM-DESIGN.md
│       └── 03-modello-dati-e-mapping.md
└── references/             # File di riferimento
    ├── SYSTEM_BLUEPRINT_AS_IS.md
    ├── ANALISI_DMS_LEGACY_COMPLETA.md
    └── screenshot_*.webp
```

---

## Principi Architetturali Non Negoziabili

1. **Un solo DB per il nuovo mondo**: Postgres (Neon) - eliminare PlanetScale/MySQL
2. **API-first**: Frontend parla SOLO con Core API, niente accessi diretti al DB
3. **Legacy isolato**: DMS Heroku è sistema esterno da integrare
4. **GIS di prima classe**: Le piazzole hanno geometria, non sono solo righe in tabella

---

## Roadmap (Sintesi)

- **Fase 0**: Allineamento tecnico (pulizia DB, Drizzle solo Postgres)
- **Fase 1**: **PEPE GIS** (PRIORITÀ ASSOLUTA)
- **Fase 2**: Analisi & copia logiche DMS legacy (Heroku)
- **Fase 3**: Nuovo Core gestionale
- **Fase 4**: Allineamento App ambulanti
- **Fase 5**: Pulizia finale e sunset del legacy

*Per i dettagli completi, vedere il [MASTER SYSTEM PLAN v2](docs/MASTER%20SYSTEM%20PLAN%20v2%20-%20DMS%20_%20MIO-HUB.md).*

---

## Sistemi Coinvolti

| Sistema | Hosting | Ruolo | Stato |
|---------|---------|-------|-------|
| **DMS Core API** | Hetzner | Backend principale, business logic | TO-BE |
| **Dashboard / MIO-HUB** | Vercel | Interfaccia web amministrativa | TO-BE |
| **DMS Legacy** | Heroku | Gestionale esistente | AS-IS (da integrare) |
| **App Mobile DMS** | - | App per ambulanti | AS-IS (da migrare) |
| **Database** | Neon | PostgreSQL unico | TO-BE |
| **Pepe GIS** | - | Motore mappe mercati | TO-BE |

---

## Regole per il Repository

- Tutte le decisioni architetturali vanno qui, non sparse in chat
- Quando si scopre qualcosa di nuovo (tabella, regola di business):
  - Aggiornare i documenti nel repo
  - Nessun "piano parallelo": il MASTER SYSTEM PLAN v2 è il riferimento unico

---

## Contatti e Manutenzione

Questo repository è mantenuto come documentazione vivente del progetto DMS / MIO-HUB.

Per domande o chiarimenti, fare riferimento al documento principale.
