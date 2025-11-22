# MASTER SYSTEM PLAN - DMS / MIO-HUB

**Versione:** 1.0
**Data:** 22 Novembre 2025
**Autore:** Manus AI

## 1. Visione e Obiettivi

Questo documento descrive l'architettura completa e la roadmap di implementazione del sistema integrato **DMS / MIO-HUB**, con l'obiettivo di creare una piattaforma unificata per la gestione dei mercati digitali, partendo dal progetto pilota del **Comune di Grosseto**.

Il sistema è composto da:
- **DMS HUB (Backend):** Servizi core, API, integrazioni (Hetzner).
- **Dashboard PA (Frontend):** Interfaccia di gestione per operatori comunali.
- **DMS Legacy (Heroku):** Gestionale originale, fonte di dati e logiche da migrare.

L'obiettivo finale è che la **Dashboard PA sostituisca completamente il DMS Legacy**, diventando l'unico punto di accesso per la gestione dei mercati.

## 2. Architettura Generale

### 2.1. Componenti Chiave

| Componente | Tecnologia | Scopo |
|---|---|---|
| **DMS HUB (Backend)** | Node.js, TypeScript, tRPC, Drizzle ORM, PostgreSQL | Servizi core, API, integrazioni |
| **Dashboard PA (Frontend)** | React, TypeScript, Vite, Mantine UI | Interfaccia di gestione |
| **Database** | PostgreSQL (Hetzner) | Dati mercati, posteggi, imprese, wallet, log |
| **GIS** | PostGIS, G3W-SUITE | Gestione dati geospaziali (mappe, posteggi) |
| **Autenticazione** | SPID/CIE (OIDC) | SSO per operatori e ufficiali |
| **Pagamenti** | PagoPA / E-FIL | Gestione pagamenti e wallet |
| **Integrazioni PA** | PDND, SSET, ABACO, MAGGIOLI | Scambio dati con Pubblica Amministrazione |

### 2.2. Flusso Dati Principale

1.  **Dati Mercato/GIS:** Sincronizzati da PostGIS al backend DMS HUB.
2.  **Dashboard PA:** Interroga le API del backend DMS HUB per visualizzare e gestire i dati.
3.  **Integrazioni Esterne:** Il backend DMS HUB orchestra le chiamate a servizi esterni (PagoPA, PDND, ABACO, MAGGIOLI).
4.  **Autenticazione:** Gestita tramite SPID/CIE, con ruoli e permessi definiti nel backend.

## 3. Moduli Funzionali e Roadmap

### Fase 1: Fondamenta e GIS (✅ FATTO)

-   **Gestione Mercati:** Creazione anagrafica mercati, posteggi, imprese.
-   **Mappa GIS:** Standardizzazione su `MarketMapComponent` con dati reali da PostGIS.
-   **Log & Debug:** Pagina dedicata per il monitoraggio.
-   **Integrazioni (struttura):** Creata pagina con struttura per future integrazioni.

### Fase 2: HUB e Pagamenti (🟡 IN CORSO)

-   **Gestione HUB:** Consolidamento schema backend e API per HUB, negozi e servizi. Collegamento del frontend alle API reali.
-   **Pagamenti & Wallet:** Implementazione gestione pagamenti via PagoPA/E-FIL e wallet per ogni impresa.

### Fase 3: Integrazioni PA (❌ NON AVVIATO)

-   **Integrazioni Reali:** Collegamento a SPID/CIE, PDND, ABACO, MAGGIOLI per avere dati reali su verifiche PA, verbali, posizioni tributarie.

### Fase 4: Centro Mobilità Scalabile (❌ NON AVVIATO)

-   **Architettura Scalabile:** Implementazione architettura per provider di mobilità configurabili per ogni mercato, con fallback al Centro Mobilità Nazionale.

## 4. Dettaglio Implementazione Grosseto

Per una mappatura dettagliata tra i moduli del Piano Lavori DMS Grosseto e l'implementazione attuale, si rimanda al documento:

[**Implementazione Comune di Grosseto**](./projects/01-grosseto-implementation.md)

## 5. Documentazione di Riferimento

-   [Architettura Scalabile Centro Mobilità](./Architettura%20Scalabile%20Centro%20Mobilit%C3%A0.md)
-   [Piano di Implementazione Dashboard PA](./Piano%20di%20Implementazione%20Dashboard%20PA.md)
