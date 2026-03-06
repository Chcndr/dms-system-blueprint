# 🏛️ MioHub - Master Blueprint

> **Versione:** 8.17.0 (Fix Interoperabilità Legacy + Fix verbale_code)
> **Data:** 06 Marzo 2026
> **Autore:** Sistema documentato da Manus AI & Claude AI
> **Stato:** PRODUZIONE

---

## 📋 INDICE

1. [Panoramica Sistema](#panoramica-sistema)
2. [Architettura Completa](#architettura-completa)
3. [Repository GitHub](#repository-github)
4. [Servizi e Componenti](#servizi-e-componenti)
5. [MIO Agent - Sistema Multi-Agente](#mio-agent---sistema-multi-agente)
6. [Knowledge Base DMS](#knowledge-base-dms)
7. [Guardian - Sistema di Monitoraggio](#guardian---sistema-di-monitoraggio)
8. [Database e Storage](#database-e-storage)
9. [API Endpoints](#api-endpoints)
10. [SSO SUAP - Modulo SCIA](#sso-suap---modulo-scia)
11. [Deploy e CI/CD](#deploy-e-cicd)
12. [Architettura di Autenticazione (v2.0 - Firebase)](#architettura-di-autenticazione-v20---firebase)
13. [Credenziali e Accessi](#credenziali-e-accessi)
14. [Troubleshooting](#troubleshooting)
15. [Regole per Agenti AI](#regole-per-agenti-ai)

---

## 📝 CHANGELOG RECENTE

### Sessione 06 Marzo 2026 (v8.16.0 → v8.17.0)

**Backend (mihub-backend-rest) — 6 commit:**

- ✅ **Fix Critico `verbale_code`:** Sostituita la generazione del `verbale_code` (che usava `Math.random()` e causava collisioni `duplicate key`) con una **sequenza PostgreSQL `verbale_code_seq`**. Questo garantisce unicità atomica e thread-safe per tutti i verbali generati, sia manualmente che dal sistema CROM.
- ✅ **Fix Critico A1 - Stato Concessione Legacy:** Corretta la mappatura dello stato concessione nel sync-out verso il Legacy. Ora `attiva` viene tradotto in `A` (codice `CHAR(1)`) invece che `ATTIVA` (stringa lunga), risolvendo un bug che rompeva la sync.
- ✅ **Fix A3 - Tracciabilità Operatore Presenze:** Aggiunto il mapping `suser_id` → `users.id` tramite email. Ora le presenze importate dal Legacy sono correttamente collegate all'operatore che le ha registrate sul campo.
- ✅ **Fix B1 - Arricchimento Dati Mercato:** La trasformazione dei mercati dal Legacy ora estrae la provincia dalla stringa `mkt_city` (es. "Firenze (FI)" → `province: "FI"`), arricchendo i dati su MioHub.
- ✅ **Fix B2 - Dati Reali Posteggi/Incassato:** Le sessioni importate dal Legacy ora hanno i valori reali di `posteggi_occupati` e `totale_incassato`, calcolati con `COUNT` e `SUM` invece di essere hardcodati a 0.
- ✅ **Fix B4 - Timestamp Presenze Robusto:** Refactorata la fragile logica di combinazione data+ora in una funzione utility `combineDateAndTime()` sicura e testabile, che gestisce vari formati TIME, ISO e valori null.

**Ricognizione Sistema Legacy:**
- ✅ **Report Completo:** Prodotto un report di 527 righe (`DMS_LEGACY_RECONNAISSANCE.md`) che mappa l'intera architettura di interoperabilità, i 24 endpoint, le 12 tabelle, le 9 stored functions e i 17 transformer. Sono stati identificati 3 problemi critici e 5 medi, ora risolti.
- ✅ **Script di Verifica:** Creato lo script `scripts/verify_legacy_db.py` per eseguire 7 verifiche formali direttamente sul DB Legacy per confermare lo schema e i dati.

---

## 🏛️ INTEGRAZIONE DMS LEGACY (Heroku) — PROGETTO COMPLETO v3.1 (Aggiornato 06/03/2026)

> **Versione progetto:** 3.1.0
> **Data:** 06 Marzo 2026
> **Principio fondamentale:** MioHub si adatta al formato del Legacy. Quando ci connettiamo, i dati sono già nel formato che il Legacy si aspetta — stessi nomi colonne, stessi tipi, stessi valori. Il Legacy non deve cambiare nulla.

### 1. Visione Strategica

**MioHub è il CERVELLO** — elabora, decide, autorizza. Si connette a SUAP, PagoPA, PDND, ANPR. Gestisce login imprese (SPID/CIE), concessioni, canone, more, mappa GIS, wallet TCC, 23 controlli SCIA, verbali automatici.

**DMS Legacy è il BRACCIO** — opera sul campo, raccoglie dati grezzi. L'app tablet registra presenze fisiche, uscite, deposito spazzatura, scelte alla spunta.

**Il dialogo:** MioHub riceve il dato grezzo dal campo → lo lavora → restituisce il dato elaborato al Legacy per dirgli cosa deve fare con ogni impresa.

| Ruolo | Sistema | Cosa fa |
|---|---|---|
| **CERVELLO** | MioHub | Login SPID/CIE, SUAP, PagoPA, PDND, concessioni, canone, more, mappa GIS, wallet, controlli, verbali |
| **BRACCIO** | DMS Legacy | App tablet spunta, presenze fisiche, uscite, spazzatura, scelte spunta |

### 2. Architettura DMS Legacy

| Componente | Dettagli |
|---|---|
| **Piattaforma** | Heroku (app `lapsy-dms`) |
| **URL Gestionale** | `https://lapsy-dms.herokuapp.com/index.html` |
| **Credenziali Gestionale** | `checchi@me.com` / `Dms2022!` (accesso frontend) |
| **Backend** | Node.js + Express — **thin layer** sopra stored functions |
| **Database** | PostgreSQL su AWS RDS (`eu-west-1`) — **25 tabelle, 117 stored functions** |
| **URI Database** | `postgres://u4gjr63u7b0f3k:p813...scl.eu-west-1.rds.amazonaws.com:5432/d18d7n7ncg8ao7` |
| **Real-time** | Socket.IO namespace `/ac.mappe` per aggiornamento mappe tablet |
| **Pattern** | Ogni API chiama una stored function: `Express → SELECT funzione(json) → PostgreSQL` |
| **CRUD** | Funzioni `_crup`: se ID è NULL → INSERT, se valorizzato → UPDATE |

### 3. Diagramma Architettura Bidirezionale

![Architettura Bidirezionale MioHub ↔ DMS Legacy](https://files.manuscdn.com/user_upload_by_module/session_file/310519663287267762/fSuZPPQqEMGIJtjI.png)

### 4. Flusso Dati Bidirezionale (Aggiornato)

#### 4.1 MioHub → Legacy (SYNC OUT)

| Dato che mandiamo | Tabella Legacy | Colonne Legacy (formato esatto) | Nostra sorgente |
|---|---|---|---|
| **Anagrafica imprese** | `amb` | `amb_ragsoc`, `amb_piva`, `amb_cfisc`, `amb_email`, `amb_phone`, `amb_addr_via`, `amb_addr_civ`, `amb_addr_cap`, `amb_addr_city`, `amb_addr_prov` | `imprese` (dati verificati SUAP/SPID) |
| **Saldo wallet** | `amb` | `amb_saldo_bors` (numeric 8,2) | `wallets.balance` |
| **Punteggio graduatoria** | `amb` | `amb_punti_grad_dfl` (integer) | `graduatoria_presenze.punteggio` |
| **Fido impresa** | `amb` | `amb_fido` (numeric 8,2) | `imprese.fido` |
| **Mercati** | `mercati` | `mkt_desc`, `mkt_city`, `mkt_addr`, `mkt_lat`, `mkt_lng`, `mkt_prezzo`, `mkt_dal`, `mkt_al` | `markets` |
| **Posteggi con mappa** | `piazzole` | `pz_numero`, `pz_mq`, `pz_lat`, `pz_lng`, `pz_height`, `pz_width`, `pz_alimentare`, `pz_enabled` | `stalls` + `geometry_geojson` |
| **Concessioni** | `conc_std` | `conc_dal`, `conc_al`, `conc_stato` (`A`/`S`/`R`/`C`), `conc_importo`, `conc_alimentare`, `amb_id`, `mkt_id`, `pz_id` | `concessions` |
| **Autorizzazioni spunta** | `spuntisti` | `sp_dal`, `sp_al`, `sp_stato`, `sp_importo`, `amb_id`, `mkt_id` | `wallets` (type=SPUNTA) |
| **Utenti/operatori** | `suser` | `suser_email`, `suser_nome`, `suser_cognome`, `suser_phone`, `suser_role`, `suser_enabled`, `suser_badge` (CIE) | `users` |
| **Regolarità impresa** | via `conc_stato` | `A` = può operare, `R`/`S` = bloccata | Calcolata da 23 controlli SCIA |

#### 4.2 Legacy → MioHub (SYNC IN)

| Dato che riceviamo | Tabella Legacy | Colonne Legacy | Nostra destinazione |
|---|---|---|---|
| **Presenza ingresso** | `presenze` | `pre_ingresso` (time), `amb_id`, `pz_id`, `ist_id` | `vendor_presences.checkin_time` |
| **Uscita** | `presenze` | `pre_uscita` (time) | `vendor_presences.checkout_time` |
| **Deposito spazzatura** | `presenze` | `pre_spazzatura` (boolean) | `vendor_presences.orario_deposito_rifiuti` |
| **Presenza rifiutata** | `presenze` | `pre_rifiutata` (boolean) | Flag nella nostra presenza |
| **Note operatore** | `presenze` | `pre_note` (text) | `vendor_presences.note` |
| **Prezzo calcolato** | `presenze` | `pre_prezzo` (numeric 8,2) | `vendor_presences.importo_addebitato` |
| **Tipo presenza** | `presenze` | `pre_tipo` (varchar) | `vendor_presences.tipo_presenza` |
| **Operatore che registra** | `presenze` | `suser_id` (integer) | `vendor_presences.operatore_id` (via email match) |
| **Giornata mercato** | `istanze` | `ist_id`, `ist_stato` | `market_sessions` |
| **Posti scelti spunta** | `presenze` | `pz_id` (posteggio scelto dallo spuntista) | `vendor_presences.stall_id` |

### 5. Endpoint MioHub Implementati (Aggiornato)

Tutti gli endpoint sono prefissati con `/api/integrations/dms-legacy/` e sono **TUTTI ATTIVI**.

| Tipo | # | Descrizione |
|---|---|---|
| **EXPORT** | 9 | Lettura diretta DB Legacy (mercati, ambulanti, concessioni, etc.) |
| **SYNC OUT** | 9 | Scrittura su DB Legacy via stored functions `_crup` (imprese, mercati, concessioni, etc.) |
| **SYNC IN** | 2 | Ricezione presenze e giornate dal campo |
| **UTILITY** | 4 | Health check, status, sync manuale, sync CRON |

---


*Il resto del documento rimane invariato.*
