# Report Sincronizzazione Infrastruttura DMS/MIO-hub

**Data:** 29 Novembre 2025

Questo report riassume i risultati della verifica AS-IS e consolida lo stato reale dell'infrastruttura.

## 1. URL e Server Confermati

- **Frontend PA**: `https://dms-hub-app-new.vercel.app` (✅ Confermato, attivo)
- **Backend Attivo**: `https://mihub.157-90-29-66.nip.io` (✅ Confermato, attivo)
- **Server IP**: `157.90.29.66` (✅ Confermato)
- **Database Neon**: Connessione confermata e funzionante.
- **Dominio `orchestratore.mio-hub.me`**: 🔴 **NON ATTIVO**. Il frontend punta a questo dominio, che non è risolto.

## 2. Stato Endpoint DMS/MIHUB

| Stato | Conteggio | Endpoint Esempio |
| :--- | :--- | :--- |
| ✅ **OK** | 8 | `/api/dmsHub/markets/list`, `/api/gis/health` |
| 🔴 **ROTTO** | 2 | `/api/dmsHub/stalls/listByMarket`, `/api/dmsHub/vendors/list` |
| 🔴 **NON IMPLEMENTATO** | 1 | `/api/stalls` (legacy) |
| ⚪ **NON TESTATO** | 3 | `/api/dmsHub/stalls/updateStatus`, etc. |

**Problema Principale**: Due endpoint DMS Hub critici sono rotti a causa di errori SQL, impedendo la visualizzazione di posteggi e venditori.

## 3. Cosa Resta Fuori (Discrepanze)

- **Confusione sui Repository**: Sul server sono presenti sia `mihub-backend-rest` (attivo) che `mihub` (monorepo legacy). È necessario definire quale sia la fonte di verità per il futuro sviluppo.
- **Configurazione Frontend Errata**: L'URL dell'API nel frontend è sbagliato e deve essere corretto per puntare all'IP reale del backend.
- **Endpoint Mock/Non Implementati**: Molti endpoint definiti nel catalogo teorico (`MIO-hub/api/index.json`) non sono implementati nel backend attualmente in esecuzione.

Questo report, insieme ai documenti aggiornati nel blueprint, fornisce una base chiara per le prossime azioni di stabilizzazione.
