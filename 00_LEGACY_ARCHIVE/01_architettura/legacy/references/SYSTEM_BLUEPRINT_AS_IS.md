# 🗺️ SYSTEM BLUEPRINT AS-IS: DMS + MIO-Hub

**Autore:** Manus AI  
**Data:** 21 Novembre 2025, 02:50 CET  
**Versione:** 1.0 (AS-IS)

---

## 📖 INTRODUZIONE

Questo documento è un **blueprint completo** dello stato attuale ("AS-IS") del sistema integrato **DMS (Digital Market System) + MIO-Hub**. È stato creato per fornire una mappa dettagliata di tutti i componenti, architetture, infrastrutture, codice e problemi noti, con l'obiettivo di permettere a qualsiasi sviluppatore (umano o AI) di ricostruire il sistema da zero.

---

## 🎯 OBIETTIVI DEL DOCUMENTO

1. **Fotografare lo stato attuale**: Documentare ogni pezzo del sistema così com'è oggi, inclusi errori e debito tecnico.
2. **Creare una fonte unica di verità**: Eliminare la necessità di cercare informazioni in file sparsi.
3. **Abilitare la manutenzione**: Fornire una base solida per future modifiche e pulizia tecnica.
4. **Facilitare l'onboarding**: Permettere a nuovi membri del team di capire rapidamente il sistema.

---

## 🏗️ ARCHITETTURA GENERALE

Il sistema è composto da 3 macro-componenti principali che comunicano tra loro:

1. **Frontend (Dashboard PA)**: Un'applicazione React + Vite deployata su Vercel, che funge da interfaccia utente per la Pubblica Amministrazione.
2. **Backend (MIO-Hub Orchestrator)**: Un'applicazione Node.js + Express + tRPC che gira su un server Hetzner, responsabile dell'orchestrazione degli agenti AI e della logica business.
3. **Database (PostgreSQL)**: Un database PostgreSQL gestito su Neon, che funge da persistenza dati per tutto il sistema.

### Schema di Comunicazione

```mermaid
graph TD
    A[👤 Utente PA] --> B{Frontend (Vercel)};
    B -->|tRPC (HTTPS)| C{Backend (Hetzner)};
    C -->|SQL| D[Database (Neon)];
    C -->|API REST| E[OpenAI API];
    C -->|API REST| F[Altri Servizi Esterni];
```

---

## 💻 STACK TECNOLOGICO

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
| | Docker | 24.x | Containerization (prevista) |
| **AI** | OpenAI API | - | Modelli GPT-5 |

---


## 🌐 PARTE 2: INFRASTRUTTURA

### 1. Server Hetzner

- **Provider**: Hetzner
- **Hostname**: ubuntu-4gb-nbg1-11
- **IP Pubblico**: 157.90.29.66
- **OS**: Ubuntu 22.04
- **Credenziali**: Chiave SSH `/home/ubuntu/.ssh/hetzner_server_key`

#### Servizi in Esecuzione

- **Backend MIO-Hub**: Applicazione Node.js in ascolto su porta `3000`.
- **Nginx**: Reverse proxy che mappa `https://orchestratore.mio-hub.me` a `http://localhost:3000`.
- **Certbot**: Gestione certificati SSL Let's Encrypt.

#### Configurazione Nginx (`/etc/nginx/sites-enabled/default`)

```nginx
server {
    listen 80;
    server_name orchestratore.mio-hub.me;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name orchestratore.mio-hub.me;

    ssl_certificate /etc/letsencrypt/live/orchestratore.mio-hub.me/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/orchestratore.mio-hub.me/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 2. Vercel

- **Account**: andreas-projects-a6e30e41
- **Progetto**: `dms-hub-app-new`
- **Production URL**: https://dms-hub-app-new.vercel.app
- **Repository Collegato**: `Chcndr/dms-hub-app-new`
- **Branch Production**: `master`

#### Configurazioni Build & Deployment

- **Framework Preset**: Vite
- **Build Command**: `pnpm run build`
- **Output Directory**: `dist`
- **Install Command**: `pnpm install`

### 3. Database Neon

- **Provider**: Neon
- **Tipo**: PostgreSQL Serverless
- **Credenziali**: `DATABASE_URL` (configurata come variabile ambiente in Vercel e Hetzner)

### 4. Credenziali e Variabili Ambiente

| Variabile | Servizio | Utilizzo |
|---|---|---|
| `DATABASE_URL` | Vercel, Hetzner | Connessione a Neon DB |
| `OPENAI_API_KEY` | Vercel, Hetzner | Accesso a OpenAI API |
| `VITE_TRPC_URL` | Vercel | URL backend per tRPC client |
| `UPSTASH_REDIS_REST_TOKEN` | Vercel, Hetzner | Accesso a Upstash Redis |
| `UPSTASH_REDIS_REST_URL` | Vercel, Hetzner | URL Upstash Redis |

---


## 🗃️ PARTE 3: DATABASE (PostgreSQL su Neon)

Lo schema del database è definito usando **Drizzle ORM** e contiene **47 tabelle**. Di seguito la descrizione completa di ogni tabella.

### 1. Tabella: `users`

- **Scopo**: Tabella principale per l'autenticazione e gestione utenti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `openId` | `varchar(64)` | `NOT NULL, UNIQUE` | ID OAuth di Manus |
| `name` | `text` | - | Nome utente |
| `email` | `varchar(320)` | - | Email utente |
| `loginMethod` | `varchar(64)` | - | Metodo di login |
| `role` | `roleEnum` | `DEFAULT 'user'` | Ruolo utente ('user', 'admin') |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |
| `updatedAt` | `timestamp` | `DEFAULT NOW()` | Data ultimo aggiornamento |
| `lastSignedIn` | `timestamp` | `DEFAULT NOW()` | Data ultimo accesso |

### 2. Tabella: `extended_users`

- **Scopo**: Estensione della tabella `users` con dati legati a wallet e sostenibilità.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `NOT NULL, FK(users.id)` | Riferimento a `users` |
| `walletBalance` | `integer` | `DEFAULT 0` | Saldo TCC (Token Carbon Credit) |
| `sustainabilityRating` | `integer` | `DEFAULT 0` | Punteggio sostenibilità (0-100) |
| `transportPreference` | `varchar(50)` | - | Preferenza trasporto (bike, car, bus) |
| `phone` | `varchar(50)` | - | Numero di telefono |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |
| `updatedAt` | `timestamp` | `DEFAULT NOW()` | Data ultimo aggiornamento |

---


### 3. Tabella: `markets`

- **Scopo**: Anagrafica dei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `name` | `varchar(255)` | `NOT NULL` | Nome del mercato |
| `address` | `text` | `NOT NULL` | Indirizzo |
| `city` | `varchar(100)` | `NOT NULL` | Città |
| `lat` | `varchar(20)` | `NOT NULL` | Latitudine |
| `lng` | `varchar(20)` | `NOT NULL` | Longitudine |
| `openingHours` | `text` | - | Orari di apertura (JSON) |
| `active` | `integer` | `DEFAULT 1` | Stato (1=attivo, 0=inattivo) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 4. Tabella: `shops`

- **Scopo**: Anagrafica dei negozi/banchi all'interno dei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `name` | `varchar(255)` | `NOT NULL` | Nome del negozio |
| `category` | `varchar(100)` | - | Categoria (bio, km0, dop) |
| `certifications` | `text` | - | Certificazioni (JSON array) |
| `pendingReimbursement` | `integer` | `DEFAULT 0` | Crediti in attesa di rimborso |
| `totalReimbursed` | `integer` | `DEFAULT 0` | Crediti totali rimborsati |
| `bankAccount` | `varchar(100)` | - | Conto bancario per rimborsi |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 5. Tabella: `transactions`

- **Scopo**: Log di tutte le transazioni di TCC (Token Carbon Credit).

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `type` | `varchar(50)` | `NOT NULL` | Tipo (earn, spend, refund) |
| `amount` | `integer` | `NOT NULL` | Quantità di TCC |
| `euroValue` | `integer` | - | Valore in centesimi di Euro |
| `description` | `text` | - | Descrizione della transazione |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 6. Tabella: `checkins`

- **Scopo**: Log dei check-in degli utenti ai mercati, per calcolare il risparmio di CO₂.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `transport` | `varchar(50)` | - | Mezzo di trasporto usato |
| `lat` | `varchar(20)` | - | Latitudine (anonimizzata) |
| `lng` | `varchar(20)` | - | Longitudine (anonimizzata) |
| `carbonSaved` | `integer` | - | Grammi di CO₂ risparmiati |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 7. Tabella: `carbon_credits_config`

- **Scopo**: Configurazione dei parametri per il calcolo dei TCC.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `baseValue` | `integer` | `NOT NULL` | Valore base in centesimi di Euro |
| `areaBoosts` | `text` | - | Moltiplicatori per area (JSON) |
| `categoryBoosts` | `text` | - | Moltiplicatori per categoria (JSON) |
| `updatedBy` | `varchar(255)` | - | Utente che ha aggiornato la config |
| `updatedAt` | `timestamp` | `DEFAULT NOW()` | Data ultimo aggiornamento |

---

### 8. Tabella: `fund_transactions`

- **Scopo**: Log delle transazioni del fondo cassa.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `type` | `varchar(50)` | `NOT NULL` | Tipo (income, expense) |
| `source` | `varchar(255)` | `NOT NULL` | Fonte della transazione |
| `amount` | `integer` | `NOT NULL` | Importo in centesimi di Euro |
| `description` | `text` | - | Descrizione della transazione |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 9. Tabella: `reimbursements`

- **Scopo**: Gestione dei rimborsi ai negozianti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `credits` | `integer` | `NOT NULL` | Crediti da rimborsare |
| `euros` | `integer` | `NOT NULL` | Valore in centesimi di Euro |
| `status` | `varchar(50)` | `DEFAULT 'pending'` | Stato (pending, processed, failed) |
| `batchId` | `varchar(100)` | - | ID del batch di pagamento |
| `processedAt` | `timestamp` | - | Data di processamento |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 10. Tabella: `civic_reports`

- **Scopo**: Segnalazioni civiche da parte degli utenti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `type` | `varchar(100)` | `NOT NULL` | Tipo di segnalazione |
| `description` | `text` | `NOT NULL` | Descrizione della segnalazione |
| `lat` | `varchar(20)` | - | Latitudine |
| `lng` | `varchar(20)` | - | Longitudine |
| `photoUrl` | `text` | - | URL foto allegata |
| `status` | `varchar(50)` | `DEFAULT 'pending'` | Stato (pending, in_progress, resolved) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 11. Tabella: `products`

- **Scopo**: Anagrafica dei prodotti venduti nei negozi.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `name` | `varchar(255)` | `NOT NULL` | Nome del prodotto |
| `category` | `varchar(100)` | - | Categoria prodotto |
| `certifications` | `text` | - | Certificazioni (JSON array) |
| `price` | `integer` | - | Prezzo in centesimi di Euro |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 12. Tabella: `product_tracking`

- **Scopo**: Tracciabilità dei prodotti (Digital Product Passport).

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `productId` | `integer` | `FK(products.id)` | Riferimento al prodotto |
| `tpassId` | `varchar(255)` | `UNIQUE` | ID T-Pass |
| `originCountry` | `varchar(3)` | - | Paese di origine |
| `originCity` | `varchar(255)` | - | Città di origine |
| `transportMode` | `varchar(50)` | - | Modalità di trasporto |
| `distanceKm` | `integer` | - | Distanza percorsa (km) |
| `co2Kg` | `integer` | - | Grammi di CO₂ emessi |
| `dppHash` | `varchar(255)` | - | Hash del Digital Product Passport |
| `customsCleared` | `integer` | `DEFAULT 0` | Stato sdoganamento (1=si, 0=no) |
| `ivaVerified` | `integer` | `DEFAULT 0` | Stato verifica IVA (1=si, 0=no) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 13. Tabella: `carbon_footprint`

- **Scopo**: Calcolo dell'impronta di carbonio dei prodotti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `productId` | `integer` | `FK(products.id)` | Riferimento al prodotto |
| `lifecycleCo2` | `integer` | - | CO₂ del ciclo di vita (grammi) |
| `transportCo2` | `integer` | - | CO₂ del trasporto (grammi) |
| `packagingCo2` | `integer` | - | CO₂ dell'imballaggio (grammi) |
| `totalCo2` | `integer` | - | CO₂ totale (grammi) |
| `calculatedAt` | `timestamp` | `DEFAULT NOW()` | Data calcolo |

---

### 14. Tabella: `ecocredits`

- **Scopo**: Conversione di TCC in Ecocrediti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `tccConverted` | `integer` | `NOT NULL` | TCC convertiti |
| `ecocreditAmount` | `integer` | `NOT NULL` | Valore in centesimi di Euro |
| `tpasFundId` | `varchar(255)` | - | ID del fondo T-PAS |
| `conversionRate` | `integer` | `NOT NULL` | Tasso di conversione (centesimi) |
| `convertedAt` | `timestamp` | `DEFAULT NOW()` | Data conversione |

---

### 15. Tabella: `audit_logs`

- **Scopo**: Log di audit per tracciare le azioni degli utenti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userEmail` | `varchar(255)` | - | Email dell'utente che ha eseguito l'azione |
| `action` | `varchar(255)` | `NOT NULL` | Azione eseguita (es. 'create_market') |
| `entityType` | `varchar(100)` | - | Tipo di entità modificata (es. 'market') |
| `entityId` | `integer` | - | ID dell'entità modificata |
| `oldValue` | `text` | - | Valore precedente (JSON) |
| `newValue` | `text` | - | Nuovo valore (JSON) |
| `ipAddress` | `varchar(50)` | - | Indirizzo IP dell'utente |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 16. Tabella: `system_logs`

- **Scopo**: Log di sistema per debug e monitoraggio.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `app` | `varchar(100)` | `NOT NULL` | Applicazione che ha generato il log |
| `level` | `varchar(50)` | `NOT NULL` | Livello (info, warning, error) |
| `type` | `varchar(100)` | - | Tipo di log |
| `message` | `text` | `NOT NULL` | Messaggio di log |
| `userEmail` | `varchar(255)` | - | Email utente associato |
| `ipAddress` | `varchar(50)` | - | Indirizzo IP |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 17. Tabella: `user_analytics`

- **Scopo**: Dati analitici sul comportamento degli utenti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `transport` | `varchar(50)` | - | Mezzo di trasporto più usato |
| `origin` | `varchar(255)` | - | Città/regione di provenienza |
| `sustainabilityRating` | `integer` | - | Punteggio di sostenibilità |
| `co2Saved` | `integer` | - | Grammi di CO₂ risparmiati |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 18. Tabella: `sustainability_metrics`

- **Scopo**: Metriche di sostenibilità aggregate.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `date` | `timestamp` | `NOT NULL` | Data della metrica |
| `populationRating` | `integer` | `NOT NULL` | Punteggio sostenibilità popolazione |
| `totalCo2Saved` | `integer` | `NOT NULL` | CO₂ totale risparmiato (kg) |
| `localPurchases` | `integer` | `NOT NULL` | Acquisti locali |
| `ecommercePurchases` | `integer` | `NOT NULL` | Acquisti e-commerce |
| `avgCo2Local` | `integer` | `NOT NULL` | CO₂ medio acquisti locali (grammi) |
| `avgCo2Ecommerce` | `integer` | `NOT NULL` | CO₂ medio acquisti e-commerce (grammi) |

---

### 19. Tabella: `notifications`

- **Scopo**: Gestione delle notifiche agli utenti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `title` | `varchar(255)` | `NOT NULL` | Titolo della notifica |
| `message` | `text` | `NOT NULL` | Messaggio della notifica |
| `type` | `varchar(50)` | `NOT NULL` | Tipo (push, email, sms) |
| `targetUsers` | `text` | - | Utenti target (JSON array) |
| `sent` | `integer` | `DEFAULT 0` | Stato invio (1=inviato, 0=non inviato) |
| `scheduledAt` | `timestamp` | - | Data di invio programmata |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 20. Tabella: `mio_conversations`

- **Scopo**: Log delle conversazioni con l'agente MIO.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `agent` | `varchar(100)` | `NOT NULL` | Agente AI coinvolto |
| `mode` | `varchar(50)` | - | Modalità (auto, manual) |
| `userMessage` | `text` | - | Messaggio dell'utente |
| `agentResponse` | `text` | - | Risposta dell'agente |
| `toolUsed` | `varchar(255)` | - | Tool utilizzato dall'agente |
| `toolInput` | `text` | - | Input del tool |
| `toolOutput` | `text` | - | Output del tool |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 21. Tabella: `mio_tasks`

- **Scopo**: Gestione dei task assegnati all'agente MIO.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `description` | `text` | `NOT NULL` | Descrizione del task |
| `status` | `varchar(50)` | `DEFAULT 'pending'` | Stato (pending, in_progress, completed, failed) |
| `agent` | `varchar(100)` | - | Agente assegnato |
| `result` | `text` | - | Risultato del task |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |
| `completedAt` | `timestamp` | - | Data completamento |

---

### 22. Tabella: `mio_projects`

- **Scopo**: Gestione dei progetti MIO.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `name` | `varchar(255)` | `NOT NULL` | Nome del progetto |
| `description` | `text` | - | Descrizione del progetto |
| `status` | `varchar(50)` | `DEFAULT 'active'` | Stato (active, archived) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 23. Tabella: `mio_files`

- **Scopo**: Gestione dei file associati ai progetti MIO.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `projectId` | `integer` | `FK(mio_projects.id)` | Riferimento al progetto |
| `name` | `varchar(255)` | `NOT NULL` | Nome del file |
| `path` | `text` | `NOT NULL` | Percorso del file |
| `content` | `text` | - | Contenuto del file |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |
| `updatedAt` | `timestamp` | `DEFAULT NOW()` | Data ultimo aggiornamento |

---

### 24. Tabella: `inspections`

- **Scopo**: Log delle ispezioni effettuate dalla Polizia Municipale.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio ispezionato |
| `inspectorId` | `integer` | `FK(users.id)` | Riferimento all'ispettore |
| `outcome` | `varchar(100)` | `NOT NULL` | Esito (ok, warning, violation) |
| `notes` | `text` | - | Note dell'ispezione |
| `inspectedAt` | `timestamp` | `DEFAULT NOW()` | Data ispezione |

---

### 25. Tabella: `violations`

- **Scopo**: Log delle violazioni riscontrate durante le ispezioni.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `inspectionId` | `integer` | `FK(inspections.id)` | Riferimento all'ispezione |
| `type` | `varchar(255)` | `NOT NULL` | Tipo di violazione |
| `description` | `text` | - | Descrizione della violazione |
| `fineAmount` | `integer` | - | Importo della multa in centesimi |
| `paid` | `integer` | `DEFAULT 0` | Stato pagamento (1=pagato, 0=non pagato) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 26. Tabella: `webhooks`

- **Scopo**: Gestione dei webhook per le integrazioni esterne.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `service` | `varchar(100)` | `NOT NULL` | Servizio esterno (es. Stripe, Zapier) |
| `event` | `varchar(255)` | `NOT NULL` | Evento che triggera il webhook |
| `url` | `text` | `NOT NULL` | URL del webhook |
| `active` | `integer` | `DEFAULT 1` | Stato (1=attivo, 0=inattivo) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 27. Tabella: `webhook_logs`

- **Scopo**: Log delle chiamate ai webhook.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `webhookId` | `integer` | `FK(webhooks.id)` | Riferimento al webhook |
| `statusCode` | `integer` | - | Codice di stato della risposta |
| `requestPayload` | `text` | - | Payload della richiesta (JSON) |
| `responseBody` | `text` | - | Corpo della risposta (JSON) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 28. Tabella: `gis_geometries`

- **Scopo**: Geometrie GIS per la mappatura dei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `type` | `varchar(50)` | `NOT NULL` | Tipo di geometria (polygon, point) |
| `geometry` | `text` | `NOT NULL` | Dati geometria (GeoJSON) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 29. Tabella: `gis_layers`

- **Scopo**: Layer GIS per la visualizzazione sulla mappa.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `name` | `varchar(255)` | `NOT NULL` | Nome del layer |
| `source` | `text` | `NOT NULL` | Fonte dati del layer (URL) |
| `type` | `varchar(50)` | `NOT NULL` | Tipo di layer (raster, vector) |
| `visible` | `integer` | `DEFAULT 1` | Visibilità (1=visibile, 0=nascosto) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 30. Tabella: `dms_hub_bookings`

- **Scopo**: Prenotazioni dei posteggi nei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `date` | `timestamp` | `NOT NULL` | Data della prenotazione |
| `status` | `varchar(50)` | `DEFAULT 'confirmed'` | Stato (confirmed, cancelled) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 31. Tabella: `dms_hub_market_geometry`

- **Scopo**: Geometrie specifiche dei posteggi nei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `geometry` | `text` | `NOT NULL` | Dati geometria (GeoJSON) |
| `metadata` | `text` | - | Metadati (JSON) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 32. Tabella: `dms_hub_market_metadata`

- **Scopo**: Metadati aggiuntivi per i mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `key` | `varchar(255)` | `NOT NULL` | Chiave del metadato |
| `value` | `text` | - | Valore del metadato |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 33. Tabella: `dms_hub_market_stalls`

- **Scopo**: Anagrafica dei posteggi (stalli) nei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `stallNumber` | `varchar(50)` | `NOT NULL` | Numero del posteggio |
| `geometry` | `text` | - | Geometria del posteggio (GeoJSON) |
| `status` | `varchar(50)` | `DEFAULT 'available'` | Stato (available, occupied, reserved) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 34. Tabella: `dms_hub_tariffs`

- **Scopo**: Tariffe per l'occupazione del suolo pubblico.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `description` | `text` | `NOT NULL` | Descrizione della tariffa |
| `amount` | `integer` | `NOT NULL` | Importo in centesimi |
| `unit` | `varchar(50)` | `NOT NULL` | Unità di misura (es. al giorno, al mq) |
| `validFrom` | `timestamp` | - | Data inizio validità |
| `validTo` | `timestamp` | - | Data fine validità |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 35. Tabella: `dms_hub_payments`

- **Scopo**: Log dei pagamenti per l'occupazione del suolo pubblico.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `bookingId` | `integer` | `FK(dms_hub_bookings.id)` | Riferimento alla prenotazione |
| `amount` | `integer` | `NOT NULL` | Importo in centesimi |
| `paymentMethod` | `varchar(100)` | - | Metodo di pagamento |
| `status` | `varchar(50)` | `DEFAULT 'completed'` | Stato (completed, failed) |
| `transactionId` | `varchar(255)` | - | ID della transazione di pagamento |
| `paidAt` | `timestamp` | `DEFAULT NOW()` | Data pagamento |

---

### 36. Tabella: `dms_hub_graduatorie`

- **Scopo**: Graduatorie per l'assegnazione dei posteggi.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `year` | `integer` | `NOT NULL` | Anno della graduatoria |
| `data` | `text` | - | Dati della graduatoria (JSON) |
| `publishedAt` | `timestamp` | - | Data di pubblicazione |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 37. Tabella: `dms_hub_scadenze`

- **Scopo**: Scadenze amministrative e fiscali.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `description` | `text` | `NOT NULL` | Descrizione della scadenza |
| `dueDate` | `timestamp` | `NOT NULL` | Data di scadenza |
| `status` | `varchar(50)` | `DEFAULT 'pending'` | Stato (pending, completed) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 38. Tabella: `dms_hub_wallet`

- **Scopo**: Wallet degli utenti per i pagamenti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `balance` | `integer` | `DEFAULT 0` | Saldo in centesimi |
| `currency` | `varchar(3)` | `DEFAULT 'EUR'` | Valuta |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |
| `updatedAt` | `timestamp` | `DEFAULT NOW()` | Data ultimo aggiornamento |

---

### 39. Tabella: `dms_hub_wallet_transactions`

- **Scopo**: Transazioni del wallet.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `walletId` | `integer` | `FK(dms_hub_wallet.id)` | Riferimento al wallet |
| `type` | `varchar(50)` | `NOT NULL` | Tipo (deposit, withdrawal, payment) |
| `amount` | `integer` | `NOT NULL` | Importo in centesimi |
| `description` | `text` | - | Descrizione della transazione |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 40. Tabella: `dms_hub_suap_practices`

- **Scopo**: Pratiche SUAP.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `practiceNumber` | `varchar(100)` | `NOT NULL` | Numero di pratica |
| `status` | `varchar(50)` | `DEFAULT 'pending'` | Stato (pending, approved, rejected) |
| `submittedAt` | `timestamp` | - | Data di invio |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 41. Tabella: `dms_hub_pdnd_logs`

- **Scopo**: Log delle interazioni con la Piattaforma Digitale Nazionale Dati (PDND).

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `service` | `varchar(255)` | `NOT NULL` | Servizio PDND chiamato |
| `request` | `text` | - | Richiesta (JSON) |
| `response` | `text` | - | Risposta (JSON) |
| `statusCode` | `integer` | - | Codice di stato della risposta |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 42. Tabella: `dms_hub_pagopa_transactions`

- **Scopo**: Transazioni PagoPA.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `paymentId` | `integer` | `FK(dms_hub_payments.id)` | Riferimento al pagamento |
| `iuv` | `varchar(255)` | `NOT NULL` | Identificativo Univoco di Versamento |
| `status` | `varchar(50)` | `DEFAULT 'pending'` | Stato (pending, paid, failed) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 43. Tabella: `dms_hub_sso_logs`

- **Scopo**: Log degli accessi tramite SSO.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `userId` | `integer` | `FK(users.id)` | Riferimento all'utente |
| `provider` | `varchar(100)` | `NOT NULL` | Provider SSO (es. SPID, CIE) |
| `ipAddress` | `varchar(50)` | - | Indirizzo IP |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 44. Tabella: `dms_hub_police_reports`

- **Scopo**: Report della Polizia Municipale.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `inspectorId` | `integer` | `FK(users.id)` | Riferimento all'ispettore |
| `title` | `varchar(255)` | `NOT NULL` | Titolo del report |
| `content` | `text` | - | Contenuto del report |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 45. Tabella: `dms_hub_app_ambulante_data`

- **Scopo**: Dati specifici per l'app degli ambulanti.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `shopId` | `integer` | `FK(shops.id)` | Riferimento al negozio |
| `key` | `varchar(255)` | `NOT NULL` | Chiave del dato |
| `value` | `text` | - | Valore del dato |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 46. Tabella: `dms_hub_editor_v3_projects`

- **Scopo**: Progetti dell'editor v3 per la creazione delle piante dei mercati.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `marketId` | `integer` | `FK(markets.id)` | Riferimento al mercato |
| `name` | `varchar(255)` | `NOT NULL` | Nome del progetto |
| `data` | `text` | - | Dati del progetto (JSON) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

### 47. Tabella: `dms_hub_gis_editor_data`

- **Scopo**: Dati dell'editor GIS.

| Campo | Tipo | Constraint | Descrizione |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY` | ID numerico auto-incrementato |
| `projectId` | `integer` | `FK(dms_hub_editor_v3_projects.id)` | Riferimento al progetto editor |
| `geometry` | `text` | `NOT NULL` | Geometria (GeoJSON) |
| `metadata` | `text` | - | Metadati (JSON) |
| `createdAt` | `timestamp` | `DEFAULT NOW()` | Data creazione |

---

## 🚀 PARTE 4: BACKEND API (tRPC)

Il backend è un'applicazione Node.js che usa Express come web server e tRPC per definire e esporre le API. La logica è suddivisa in router, ognuno responsabile di un dominio specifico.

### 1. Struttura dei Router

Il router principale (`server/routers.ts`) aggrega i seguenti sub-router:

- `dmsHubRouter`: Gestione mercati, posteggi, prenotazioni, pagamenti.
- `mihubRouter`: Gestione agenti AI, conversazioni, task.
- `integrationsRouter`: Gestione webhook e integrazioni esterne.
- `eventBus`: Gestione eventi di sistema.

### 2. Endpoint API (Procedure tRPC)

Di seguito la lista completa delle procedure tRPC esposte, raggruppate per router.

#### 2.1. `dmsHubRouter`

| Procedura | Input | Output | Descrizione |
|---|---|---|---|
| `getMarkets` | - | `Market[]` | Ottiene la lista di tutti i mercati. |
| `getMarketById` | `id: number` | `Market` | Ottiene un mercato specifico. |
| `createMarket` | `Market` | `Market` | Crea un nuovo mercato. |
| `updateMarket` | `Market` | `Market` | Aggiorna un mercato esistente. |
| `deleteMarket` | `id: number` | `void` | Elimina un mercato. |
| `getStallsByMarket` | `marketId: number` | `Stall[]` | Ottiene i posteggi di un mercato. |
| `createStall` | `Stall` | `Stall` | Crea un nuovo posteggio. |
| `updateStall` | `Stall` | `Stall` | Aggiorna un posteggio. |
| `deleteStall` | `id: number` | `void` | Elimina un posteggio. |
| `getBookingsByMarket` | `marketId: number` | `Booking[]` | Ottiene le prenotazioni di un mercato. |
| `createBooking` | `Booking` | `Booking` | Crea una nuova prenotazione. |
| `cancelBooking` | `id: number` | `void` | Annulla una prenotazione. |
| `getPaymentsByBooking` | `bookingId: number` | `Payment[]` | Ottiene i pagamenti di una prenotazione. |
| `createPayment` | `Payment` | `Payment` | Crea un nuovo pagamento. |

---

#### 2.2. `mihubRouter`

| Procedura | Input | Output | Descrizione |
|---|---|---|---|
| `getConversations` | `userId: number` | `Conversation[]` | Ottiene le conversazioni di un utente. |
| `createConversation` | `Conversation` | `Conversation` | Crea una nuova conversazione. |
| `getTasks` | `userId: number` | `Task[]` | Ottiene i task di un utente. |
| `createTask` | `Task` | `Task` | Crea un nuovo task. |
| `getProjects` | - | `Project[]` | Ottiene la lista di tutti i progetti. |
| `createProject` | `Project` | `Project` | Crea un nuovo progetto. |
| `getFilesByProject` | `projectId: number` | `File[]` | Ottiene i file di un progetto. |
| `createFile` | `File` | `File` | Crea un nuovo file. |
| `updateFile` | `File` | `File` | Aggiorna un file. |
| `deleteFile` | `id: number` | `void` | Elimina un file. |
| `orchestrator` | `mode: string, targetAgent: string, message: string` | `AgentResponse` | Invia un messaggio all'orchestratore. |

---

#### 2.3. `integrationsRouter`

| Procedura | Input | Output | Descrizione |
|---|---|---|---|
| `getWebhooks` | - | `Webhook[]` | Ottiene la lista di tutti i webhook. |
| `createWebhook` | `Webhook` | `Webhook` | Crea un nuovo webhook. |
| `updateWebhook` | `Webhook` | `Webhook` | Aggiorna un webhook. |
| `deleteWebhook` | `id: number` | `void` | Elimina un webhook. |
| `getWebhookLogs` | `webhookId: number` | `WebhookLog[]` | Ottiene i log di un webhook. |

---

#### 2.4. `eventBus`

| Procedura | Input | Output | Descrizione |
|---|---|---|---|
| `publishEvent` | `topic: string, payload: any` | `void` | Pubblica un evento sul bus. |
| `subscribeToEvent` | `topic: string, callbackUrl: string` | `void` | Sottoscrive un webhook a un evento. |

---

## 🎨 PARTE 5: FRONTEND (React + Vite)

Il frontend è un'applicazione React moderna costruita con Vite. La logica è organizzata in componenti, pagine e servizi.

### 1. Struttura dei Componenti

La directory `client/src/components` contiene tutti i componenti riutilizzabili, tra cui:

- `Dashboard`: Il layout principale della dashboard.
- `Chat`: Il componente per la chat con gli agenti AI.
- `Map`: Il componente per la visualizzazione GIS.
- `Table`: Un componente generico per visualizzare dati in tabelle.

### 2. Pagine e Routing

Il routing è gestito da `react-router-dom`. Le pagine principali sono:

- `/`: Homepage
- `/dashboard-pa`: Dashboard della Pubblica Amministrazione
- `/login`: Pagina di login

### 3. State Management

Lo stato globale è gestito tramite una combinazione di:

- **React Context**: Per lo stato dell'utente e le configurazioni.
- **tRPC React Query**: Per il fetching e il caching dei dati dal backend.

---

## 🤖 PARTE 6: AGENTI AI

Il sistema MIO-Hub è costruito attorno a un ecosistema di agenti AI specializzati, coordinati da un orchestratore.

### 1. MIO (Coordinatore)

- **Scopo**: Agente principale che interagisce con l'utente, interpreta le richieste e delega i task agli agenti specializzati.
- **Modello**: GPT-5
- **Modalità**: Auto (delega automaticamente), Manual (l'utente sceglie l'agente)

### 2. Agenti Specializzati

- **MIO Dev**: Agente specializzato in operazioni di sviluppo software (GitHub, Vercel, etc.).
- **Manus**: Agente general-purpose per task di ricerca, scrittura, analisi.
- **Abacus**: Agente specializzato in analisi dati e creazione di grafici.
- **Zapier**: Agente per l'automazione di workflow tramite Zapier.

### 3. Orchestratore

- **Scopo**: Riceve le richieste da MIO e le instrada all'agente corretto. Gestisce il flusso di comunicazione tra gli agenti.
- **Endpoint**: `/mihub/orchestrator`

### 4. Guardian

- **Scopo**: Sistema di sicurezza che monitora le azioni degli agenti e richiede conferma per operazioni ad alto rischio.
- **Configurazione**: `api/index.json`, `agents/permissions.json`, `config/api-guardian.json`

---

## ⚠️ PARTE 7: PROBLEMI NOTI

Il sistema, nella sua configurazione attuale, presenta alcuni problemi noti che richiedono attenzione.

### 1. Cache Vercel

- **Problema**: Vercel ha una cache molto aggressiva che impedisce l'aggiornamento del bundle JavaScript anche dopo nuovi deploy. Questo causa la visualizzazione di versioni vecchie del frontend.
- **Soluzione temporanea**: Forzare l'invalidazione della cache tramite commit vuoti con timestamp, o modificando file chiave per cambiare l'hash del bundle.
- **Soluzione definitiva**: Creare un nuovo progetto Vercel per bypassare completamente la cache.

### 2. Mismatch MySQL vs Postgres

- **Problema**: Il codice contiene ancora riferimenti a tipi e funzioni specifiche di MySQL, nonostante il database sia PostgreSQL. Questo causa errori di tipo in fase di build.
- **Soluzione**: Rimuovere tutti i riferimenti a MySQL e usare solo tipi e funzioni compatibili con PostgreSQL.

### 3. Errori TypeScript

- **Problema**: Il build su Vercel mostra ancora errori TypeScript, anche se il build locale passa. Questo è probabilmente dovuto a una configurazione di `tsconfig.json` non ottimale o a problemi di cache.
- **Soluzione**: Rivedere la configurazione di TypeScript e assicurarsi che sia consistente tra locale e Vercel.

### 4. Configurazioni Inconsistenti

- **Problema**: Ci sono configurazioni diverse tra l'ambiente di produzione e quello di sviluppo, in particolare per i comandi di build e le directory di output.
- **Soluzione**: Unificare le configurazioni e usare variabili d'ambiente per gestire le differenze tra ambienti.

---

## 🚀 PARTE 8: DEPLOYMENT

Le procedure di deploy sono diverse per il frontend e il backend.

### 1. Frontend (Dashboard PA su Vercel)

- **Trigger**: Automatico a ogni push sul branch `master` del repository `dms-hub-app-new`.
- **Comando di build**: `pnpm run build`
- **Directory di output**: `dist`
- **Variabili d'ambiente**: `VITE_TRPC_URL` (punta al backend su Hetzner)

### 2. Backend (MIO-Hub su Hetzner)

- **Procedura**: Manuale, tramite SSH.
- **Comandi**:
  1. `git pull origin master`
  2. `docker-compose down`
  3. `docker-compose up -d --build`
- **Migrazioni database**: Eseguire `pnpm drizzle-kit migrate` prima del deploy se ci sono modifiche allo schema.

---
