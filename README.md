# DMS System Blueprint

**Repository ufficiale** per l'architettura e la documentazione del sistema DMS (Dealer Management System) per la gestione dei mercati ambulanti.

Questo repository è la **fonte di verità ufficiale** per:
- Architettura AS-IS (stato attuale del sistema)
- Architettura TO-BE (progetto finale del sistema)
- Mapping tra sistemi legacy e nuovo sistema
- Roadmap tecnica e decisioni implementative

---

## Struttura del Repository

```
dms-system-blueprint/
├── docs/
│   ├── AS-IS/              # Analisi dello stato attuale
│   │   ├── 00-overview.md
│   │   ├── 01-sistemi-e-ambienti.md
│   │   ├── 02-dominio-funzionale.md
│   │   ├── 03-modello-dati-e-mapping.md
│   │   ├── 04-architettura-tecnica.md
│   │   ├── 05-integrazioni-esterne.md
│   │   └── 06-roadmap.md
│   └── TO-BE/              # Progetto finale del sistema
│       ├── MASTER-SYSTEM-DESIGN.md
│       ├── 03-modello-dati-e-mapping.md
│       └── 06-roadmap-tecnica.md
└── references/             # File di riferimento
    ├── SYSTEM_BLUEPRINT_AS_IS.md
    ├── ANALISI_DMS_LEGACY_COMPLETA.md
    ├── Anagrafiche_API_DMS.xlsx (da aggiungere)
    └── screenshot_*.webp
```

---

## Documenti Chiave

### AS-IS (Stato Attuale)
- **00-overview.md**: Panoramica generale del sistema attuale
- **01-sistemi-e-ambienti.md**: Infrastruttura (Hetzner, Vercel, Neon, Heroku)
- **02-dominio-funzionale.md**: Entità e concetti chiave del dominio
- **03-modello-dati-e-mapping.md**: Schema database attuale
- **04-architettura-tecnica.md**: Backend tRPC e Frontend React
- **05-integrazioni-esterne.md**: Sistemi esterni e integrazioni
- **06-roadmap.md**: Piano di modernizzazione

### TO-BE (Progetto Finale)
- **MASTER-SYSTEM-DESIGN.md**: Architettura finale del sistema con decisioni tecniche non negoziabili
- **03-modello-dati-e-mapping.md**: Schema Postgres target e mapping da sistemi legacy
- **06-roadmap-tecnica.md**: Checklist implementativa con step concreti

### References
- **ANALISI_DMS_LEGACY_COMPLETA.md**: Analisi funzionale dettagliata del gestionale DMS su Heroku
- **SYSTEM_BLUEPRINT_AS_IS.md**: Documento completo dello stato AS-IS
- **Screenshot**: Immagini delle schermate del sistema legacy

---

## Decisioni Architetturali Non Negoziabili

1. **Database Unico**: PostgreSQL (niente più PlanetScale/MySQL)
2. **DMS Core API** (Hetzner): Unico cervello del sistema, tutta la business logic vive qui
3. **Frontend** (Next.js su Vercel): Solo UI, parla SOLO con DMS Core API, niente accesso diretto a DB
4. **DMS Legacy** (Heroku): Sistema esterno da integrare, non più il centro del mondo
5. **App DMS Ambulanti**: Nel TO-BE parla con Core API, non con il gestionale legacy
6. **GIS/Mappe**: Layer sopra i dati, agganciato a ID stabili nel DB

---

## Come Usare Questo Repository

Questo repository è il **manuale di bordo del sistema**. Deve essere aggiornato man mano che:
- Si analizza il gestionale DMS legacy
- Si scoprono dettagli tecnici
- Si modifica architettura e codice

**Regola d'oro**: Se cambi una decisione tecnica importante:
1. Aggiorna prima il `MASTER-SYSTEM-DESIGN.md`
2. Poi adegua il codice

---

## Sistemi Coinvolti

| Sistema | Hosting | Ruolo | Stato |
|---------|---------|-------|-------|
| **DMS Core API** | Hetzner | Backend principale, business logic | TO-BE |
| **DMS-HUB Frontend** | Vercel | Interfaccia web amministrativa | TO-BE |
| **DMS Legacy** | Heroku | Gestionale esistente | AS-IS (da integrare) |
| **App Mobile DMS** | - | App per ambulanti | AS-IS (da migrare) |
| **Database** | Neon/Hetzner | PostgreSQL unico | TO-BE |

---

## Contatti e Manutenzione

Questo repository è mantenuto come documentazione vivente del progetto DMS.

Per domande o chiarimenti, fare riferimento ai documenti nella directory `docs/TO-BE/`.
