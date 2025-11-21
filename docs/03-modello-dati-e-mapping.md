# 03. Modello Dati e Mapping

Questo documento descrive il modello dati del nuovo sistema basato su PostgreSQL e il mapping con il sistema legacy.

## Schema Target (PostgreSQL)

Lo schema del database è definito tramite l'ORM Drizzle e comprende 47 tabelle. Di seguito, una descrizione delle tabelle principali.

### 1. Tabella: `users`

- **Scopo**: Gestione degli utenti e autenticazione.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID utente |
| `openId` | `varchar(64)` | `NOT NULL, UNIQUE` | ID OAuth di Manus |
| `name` | `text` | - | Nome utente |
| `email` | `varchar(320)` | - | Email utente |
| `loginMethod` | `varchar(64)` | - | Metodo di login |
| `role` | `roleEnum` | `DEFAULT 'user'` | Ruolo (`user`, `admin`) |

### 2. Tabella: `extended_users`

- **Scopo**: Dati aggiuntivi legati a wallet e sostenibilità.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID |
| `userId` | `integer` | `NOT NULL, FK(users.id)` | Riferimento a `users` |
| `walletBalance` | `integer` | `DEFAULT 0` | Saldo TCC |

### 3. Tabella: `markets`

- **Scopo**: Anagrafica dei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID mercato |
| `name` | `varchar(255)` | `NOT NULL` | Nome del mercato |
| `address` | `text` | `NOT NULL` | Indirizzo |
| `city` | `varchar(100)` | `NOT NULL` | Città |

### 4. Tabella: `shops`

- **Scopo**: Anagrafica dei negozi/banchi.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID negozio |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `name` | `varchar(255)` | `NOT NULL` | Nome del negozio |

### 5. Tabella: `transactions`

- **Scopo**: Log delle transazioni di TCC.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID transazione |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `type` | `varchar(50)` | `NOT NULL` | Tipo (`earn`, `spend`, `refund`) |
| `amount` | `integer` | `NOT NULL` | Quantità di TCC |

## Mapping dal Sistema Legacy (DMS su Heroku)

Il mapping tra il database MySQL del vecchio DMS e il nuovo schema PostgreSQL è in fase di definizione. La fonte principale per questo mapping è il file `Anagrafiche_API_DMS.xlsx` che si trova nella directory `references`.

L'analisi di questo file è un'attività prioritaria della roadmap.
