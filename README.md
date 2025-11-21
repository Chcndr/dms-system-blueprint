# DMS System Blueprint

**Repository ufficiale** per l'architettura e la documentazione del sistema DMS / MIO-HUB per la gestione dei mercati ambulanti.

Questo repository è la **fonte di verità ufficiale** per:
- Architettura AS-IS (stato attuale del sistema)
- Architettura TO-BE (progetto finale del sistema)
- Mapping tra sistemi legacy e nuovo sistema
- Roadmap tecnica e decisioni implementative

---

## 📘 Documento Principale

**[MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md](MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md)**

Questo è il **documento unico e ufficiale** che unifica visione, architettura, modello dati e roadmap. È il punto di riferimento per tutti gli stakeholder (umani e AI).

---

## Struttura del Repository

```
dms-system-blueprint/
├── MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md  # 📘 DOCUMENTO PRINCIPALE
├── docs/
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

## Decisioni Architetturali Non Negoziabili

1. **Database Unico**: PostgreSQL (niente più PlanetScale/MySQL)
2. **DMS Core API** (Hetzner): Unico cervello del sistema, tutta la business logic vive qui
3. **Frontend** (Next.js su Vercel): Solo UI, parla SOLO con DMS Core API, niente accesso diretto a DB
4. **DMS Legacy** (Heroku): Sistema esterno da integrare, non più il centro del mondo
5. **App DMS Ambulanti**: Nel TO-BE parla con Core API, non con il gestionale legacy
6. **Pepe GIS**: Motore per la creazione e visualizzazione delle mappe dei mercati

---

## Roadmap (Sintesi)

- **Fase 0**: Base tecnica pulita (standardizzazione PostgreSQL, pulizia configurazioni)
- **Fase 1**: Integrazione Pepe GIS come primo caso d'uso forte (PRIORITÀ)
- **Fase 2**: Copia e allineamento del gestionale DMS (Heroku)
- **Fase 3**: Nuovo gestionale dentro la Dashboard / MIO-HUB
- **Fase 4**: Feature avanzate (CO₂, wallet, automazioni)

*Per i dettagli completi, vedere il [MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md](MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md).*

---

## Sistemi Coinvolti

| Sistema | Hosting | Ruolo | Stato |
|---------|---------|-------|-------|
| **DMS Core API** | Hetzner | Backend principale, business logic | TO-BE |
| **Dashboard / MIO-HUB** | Vercel | Interfaccia web amministrativa | TO-BE |
| **DMS Legacy** | Heroku | Gestionale esistente | AS-IS (da integrare) |
| **App Mobile DMS** | - | App per ambulanti | AS-IS (da migrare) |
| **Database** | Hetzner/Neon | PostgreSQL unico | TO-BE |
| **Pepe GIS** | - | Motore mappe mercati | TO-BE |

---

## Come Usare Questo Repository

Questo repository è il **manuale di bordo del sistema**. Deve essere aggiornato man mano che:
- Si analizza il gestionale DMS legacy
- Si scoprono dettagli tecnici
- Si modifica architettura e codice

**Regola d'oro**: Se cambi una decisione tecnica importante:
1. Aggiorna prima il `MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md`
2. Poi adegua il codice

---

## Contatti e Manutenzione

Questo repository è mantenuto come documentazione vivente del progetto DMS / MIO-HUB.

Per domande o chiarimenti, fare riferimento al documento principale.
