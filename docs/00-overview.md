# 00. Overview

Questo documento fornisce una panoramica generale del sistema DMS / DMS-HUB / MIO-HUB, della sua visione futura e dei principi guida dell'architettura.

## Contesto

Il sistema nasce dall'esigenza di modernizzare il gestionale DMS esistente, creando un nuovo ecosistema cloud-native, modulare e scalabile. Il nuovo sistema, chiamato **DMS-HUB**, deve:

- Replicare e migliorare le funzionalità del gestionale DMS storico.
- Essere compatibile con l'app DMS degli ambulanti.
- Integrare un editor GIS per la gestione delle piante dei mercati.
- Essere gestito e manutenuto tramite un ecosistema di agenti AI (MIO-HUB).

## Visione Futura (TO-BE)

L'architettura target prevede un sistema disaccoppiato, basato su microservizi e API, con:

- **Un unico DB principale PostgreSQL** come fonte di verità per tutti i dati.
- **Un DMS Core API** su Hetzner come unico punto di accesso ai dati e alla logica di business.
- **Un frontend moderno** su Vercel (React + Vite) per la Dashboard PA.
- **Un sistema di agenti AI (MIO-HUB)** per la gestione, il monitoraggio e lo sviluppo del sistema.
- **Il gestionale DMS storico** trattato come sistema legacy da integrare e progressivamente dismettere.

## Principi Guida

Il sistema si basa su alcuni principi architetturali fondamentali che guidano tutte le decisioni di design e implementazione.

**Single Source of Truth**: Tutti i dati risiedono in un unico database PostgreSQL. Non esistono database separati o fonti di dati duplicate. Questo garantisce consistenza e integrità dei dati.

**API First**: Tutta la logica di business è esposta tramite API ben definite e fortemente tipizzate (tRPC). Questo permette una chiara separazione tra frontend e backend e facilita l'integrazione con sistemi esterni.

**Separation of Concerns**: Frontend, backend e agenti AI sono componenti separati e indipendenti, ciascuno con responsabilità ben definite. Questo favorisce la manutenibilità e la scalabilità del sistema.

**Infrastructure as Code**: L'infrastruttura è definita e gestita tramite codice (es. Docker, Terraform), permettendo riproducibilità e versionamento della configurazione.

**AI-driven Development**: Lo sviluppo e la manutenzione sono supportati e automatizzati da agenti AI, che assistono nella generazione di codice, nella documentazione e nell'analisi del sistema.

## Stack Tecnologico

Il sistema utilizza tecnologie moderne e consolidate per garantire prestazioni, scalabilità e manutenibilità.

| Componente | Tecnologia | Versione | Note |
|---|---|---|---|
| **Frontend** | React | 18.x | UI library |
| | Vite | 5.x | Build tool |
| | TypeScript | 5.x | Linguaggio |
| | tRPC Client | 11.x | Comunicazione API |
| | Tailwind CSS | 3.x | Styling |
| **Backend** | Node.js | 18.x | Runtime |
| | Express | 4.x | Web server |
| | tRPC Server | 11.x | API layer |
| | Drizzle ORM | 0.44.x | Database access |
| | TypeScript | 5.x | Linguaggio |
| **Database** | PostgreSQL | 16.x | Gestito da Neon |
| **Infrastruttura** | Vercel | - | Hosting frontend |
| | Hetzner | Ubuntu 22.04 | Hosting backend |
| | Nginx | 1.18.x | Reverse proxy |
| | Docker | 24.x | Containerization |
| **AI** | OpenAI API | - | Modelli GPT |

## Problemi Noti

Il sistema, nella sua configurazione attuale, presenta alcuni problemi noti che richiedono attenzione.

**Cache Vercel**: Vercel ha una cache molto aggressiva che impedisce l'aggiornamento del bundle JavaScript anche dopo nuovi deploy. Questo causa la visualizzazione di versioni vecchie del frontend. La soluzione definitiva è stata la creazione di un nuovo progetto Vercel.

**Mismatch MySQL vs Postgres**: Il codice conteneva ancora riferimenti a tipi e funzioni specifiche di MySQL, nonostante il database sia PostgreSQL. Questo problema è stato risolto nel commit `d717837`.

**Errori TypeScript**: Il build su Vercel mostra ancora errori TypeScript, anche se il build locale passa. Questo è probabilmente dovuto a una configurazione di `tsconfig.json` non ottimale o a problemi di cache.

**Configurazioni Inconsistenti**: Ci sono configurazioni diverse tra l'ambiente di produzione e quello di sviluppo, in particolare per i comandi di build e le directory di output. È necessario unificare le configurazioni e usare variabili d'ambiente per gestire le differenze tra ambienti.
