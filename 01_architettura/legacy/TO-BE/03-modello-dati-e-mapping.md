# 03 - Modello Dati e Mapping (TO-BE)

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Stato:** In Lavorazione  
**Obiettivo:** Definire lo schema dati target in PostgreSQL e mappare le tabelle del sistema legacy DMS su Heroku verso questo nuovo schema.

---

## 1. Schema Dati Target (PostgreSQL)

Questo schema rappresenta la "single source of truth" per il sistema DMS-HUB. È progettato per essere normalizzato e scalabile.

### Tabella: `markets` (Mercati)
| Colonna | Tipo | Descrizione |
|---|---|---|
| `id` | `uuid` (Primary Key) | ID univoco del mercato |
| `name` | `varchar(255)` | Nome del mercato (es. "Mercato di Cervia") |
| `city` | `varchar(255)` | Città in cui si svolge il mercato |
| `start_date` | `date` | Data di inizio validità del mercato |
| `end_date` | `date` | Data di fine validità del mercato |
| `created_at` | `timestamp` | Data di creazione record |
| `updated_at` | `timestamp` | Data di ultimo aggiornamento |

### Tabella: `market_schedules` (Programmazione Mercati)
| Colonna | Tipo | Descrizione |
|---|---|---|
| `id` | `uuid` (Primary Key) | ID univoco della programmazione |
| `market_id` | `uuid` (Foreign Key to `markets.id`) | Riferimento al mercato |
| `frequency` | `varchar(50)` | Cadenza (es. 'daily', 'weekly', 'monthly') |
| `day_of_week` | `integer` | Giorno della settimana (1-7) per cadenza settimanale |
| `day_of_month` | `integer` | Giorno del mese per cadenza mensile |
| `start_time` | `time` | Orario di inizio |
| `end_time` | `time` | Orario di fine |

### Tabella: `stalls` (Piazzole)
| Colonna | Tipo | Descrizione |
|---|---|---|
| `id` | `uuid` (Primary Key) | ID univoco della piazzola |
| `market_id` | `uuid` (Foreign Key to `markets.id`) | Riferimento al mercato di appartenenza |
| `stall_number` | `varchar(50)` | Numero o codice della piazzola (es. "F001P001") |
| `location_geojson` | `jsonb` | Coordinate GeoJSON della piazzola sulla mappa |
| `created_at` | `timestamp` | Data di creazione record |
| `updated_at` | `timestamp` | Data di ultimo aggiornamento |

### Tabella: `vendors` (Ambulanti)
| Colonna | Tipo | Descrizione |
|---|---|---|
| `id` | `uuid` (Primary Key) | ID univoco dell'ambulante |
| `business_name` | `varchar(255)` | Ragione sociale dell'ambulante |
| `created_at` | `timestamp` | Data di creazione record |
| `updated_at` | `timestamp` | Data di ultimo aggiornamento |

### Tabella: `concessions` (Concessioni)
| Colonna | Tipo | Descrizione |
|---|---|---|
| `id` | `uuid` (Primary Key) | ID univoco della concessione |
| `stall_id` | `uuid` (Foreign Key to `stalls.id`) | Riferimento alla piazzola |
| `vendor_id` | `uuid` (Foreign Key to `vendors.id`) | Riferimento all'ambulante |
| `start_date` | `date` | Data di inizio validità della concessione |
| `end_date` | `date` | Data di fine validità della concessione |
| `status` | `varchar(50)` | Stato (es. 'active', 'expired', 'revoked') |
| `created_at` | `timestamp` | Data di creazione record |
| `updated_at` | `timestamp` | Data di ultimo aggiornamento |

### Tabella: `attendances` (Presenze)
| Colonna | Tipo | Descrizione |
|---|---|---|
| `id` | `uuid` (Primary Key) | ID univoco della presenza |
| `market_id` | `uuid` (Foreign Key to `markets.id`) | Riferimento al mercato |
| `stall_id` | `uuid` (Foreign Key to `stalls.id`) | Riferimento alla piazzola occupata |
| `vendor_id` | `uuid` (Foreign Key to `vendors.id`) | Riferimento all'ambulante presente |
| `concession_id` | `uuid` (Nullable Foreign Key to `concessions.id`) | Riferimento alla concessione (se presente) |
| `date` | `date` | Data della presenza |
| `check_in_time` | `timestamp` | Orario di ingresso |
| `check_out_time` | `timestamp` | Orario di uscita |
| `fee_amount` | `decimal(10, 2)` | Importo pagato (es. spazzatura) |
| `attendance_type` | `varchar(50)` | Tipo di presenza (es. 'concession_holder', 'spuntista') |

---

## 2. Mapping dal Sistema Legacy (Heroku)

Questa sezione mappa le tabelle e i campi del database del gestionale DMS legacy su Heroku allo schema target PostgreSQL. **Nota: questa è una mappatura preliminare basata sull'analisi funzionale. Sarà necessario un accesso diretto al DB di Heroku per una mappatura campo-per-campo definitiva.**

| Tabella Sorgente (Heroku) | Tabella Target (PostgreSQL) | Campi Sorgente → Campi Target | Regole di Mapping e Note |
|---|---|---|---|
| **(Tabella Mercati)** | `markets` | `Descrizione` → `name`<br>`Città` → `city`<br>`Dal` → `start_date`<br>`Al` → `end_date` | L'ID dovrà essere migrato e mappato. |
| **(Tabella Programmazione)** | `market_schedules` | `Cadenza` → `frequency`<br>`Gg sett.` → `day_of_week`<br>... | Da associare al `market_id` corretto. |
| **(Tabella Piazzole)** | `stalls` | `Numero` → `stall_number` | Da associare al `market_id` corretto. Il campo `location_geojson` sarà da popolare in un secondo momento. |
| **(Tabella Ambulanti)** | `vendors` | `Ragione Sociale` → `business_name` | L'ID dovrà essere migrato e mappato. |
| **(Tabella Concessioni)** | `concessions` | `Mercato` → `(lookup markets.id)`<br>`Ambulante` → `(lookup vendors.id)`<br>`Piazzola` → `(lookup stalls.id)`<br>`Dal` → `start_date`<br>`Al` → `end_date`<br>`Stato` → `status` | Richiede lookup per mappare le descrizioni testuali agli ID `uuid` del nuovo schema. |
| **(Tabella Presenze)** | `attendances` | `Ambulante` → `(lookup vendors.id)`<br>`Piazzola` → `(lookup stalls.id)`<br>`Ingresso` → `check_in_time`<br>`Uscita` → `check_out_time`<br>... | Richiede lookup e logica per determinare `attendance_type`. |
| **(Tabella Spuntisti)** | `attendances` | `Ambulante` → `(lookup vendors.id)`<br>`Valido dal` → `date`<br>... | I dati degli spuntisti confluiranno nella tabella `attendances` con `attendance_type = 'spuntista'`. |

---

## 3. Riferimenti da `Anagrafiche_API_DMS.xlsx`

Il file `Anagrafiche_API_DMS.xlsx` (attualmente mancante) è fondamentale per completare questa mappatura. Si presume che contenga:

- **ID univoci** per mercati, ambulanti e piazzole, che permetteranno di creare un mapping stabile tra il sistema legacy e il nuovo.
- **Dettagli anagrafici** aggiuntivi non presenti nell'interfaccia del gestionale.
- **Regole di business** per il calcolo di tariffe o altre logiche.

**Azione Prioritaria:** Recuperare il file `Anagrafiche_API_DMS.xlsx` per finalizzare questo documento.

---

## 4. Prossimi Passi

1.  **Ottenere accesso in sola lettura al database di Heroku** per analizzare lo schema reale e i tipi di dati.
2.  **Recuperare il file `Anagrafiche_API_DMS.xlsx`**.
3.  **Finalizzare la mappatura campo-per-campo** in questo documento, specificando tipi di dati, trasformazioni e regole di normalizzazione.
4.  **Scrivere gli script di migrazione** (ETL) per trasferire i dati dal sistema legacy al nuovo database PostgreSQL.
