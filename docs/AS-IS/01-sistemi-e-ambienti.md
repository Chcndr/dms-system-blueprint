_# 01. Sistemi e Ambienti

Questo documento descrive l'infrastruttura tecnica, gli ambienti di hosting e i servizi su cui si basa l'ecosistema DMS-HUB.

## Architettura AS-IS

L'architettura attuale è distribuita su tre provider principali: Hetzner per il backend, Vercel per il frontend e Neon per il database.

### 1. Server Backend (Hetzner)

Il cuore del sistema, l'orchestratore MIO-Hub, risiede su un server cloud Hetzner.

- **Provider**: Hetzner
- **IP Pubblico**: `157.90.29.66`
- **OS**: Ubuntu 22.04
- **Hostname**: `ubuntu-4gb-nbg1-11`

#### Servizi in Esecuzione

- **DMS Core API**: Applicazione Node.js/TypeScript in esecuzione tramite Docker container `mihub-api`.
  - **Repository**: `Chcndr/mihub` (monorepo)
  - **App**: `apps/api`
  - **Path sul server**: `/opt/mihub/apps/api`
  - **Comando**: `node dist/server.js`
  - **Porta interna**: `3000`
- **Nginx**: Reverse proxy che mappa il dominio `orchestratore.mio-hub.me` alla porta `3000`, gestendo anche il traffico HTTPS.
- **Certbot**: Tool per la gestione e il rinnovo automatico dei certificati SSL Let's Encrypt.
- **Backend REST Legacy**: Applicazione Node.js in `/root/mihub-backend-rest` (processo separato, da valutare se dismettere).

#### Configurazione Nginx

La configurazione di Nginx (`/etc/nginx/sites-enabled/default`) instrada tutto il traffico in ingresso sulla porta 443 (HTTPS) verso il servizio Node.js locale.

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

### 2. Frontend (Vercel)

L'interfaccia utente per la Pubblica Amministrazione è un'applicazione React (Vite) ospitata su Vercel.

- **Progetto**: `dms-hub-app-new`
- **URL Produzione**: [https://dms-hub-app-new.vercel.app](https://dms-hub-app-new.vercel.app)
- **Repository**: `Chcndr/dms-hub-app-new`
- **Branch di Produzione**: `master`

### 3. Database (Neon)

Il database principale è un'istanza PostgreSQL serverless gestita da Neon, che garantisce scalabilità e gestione semplificata.

- **Provider**: Neon
- **Tipo**: PostgreSQL Serverless
- **Connessione**: Avviene tramite la variabile d'ambiente `DATABASE_URL`.

### 4. Sistema Legacy (Heroku)

Il gestionale DMS originale è ospitato su Heroku e rappresenta un componente legacy da integrare.

- **Applicazione**: `lapsy-dms`
- **URL**: [https://lapsy-dms.herokuapp.com](https://lapsy-dms.herokuapp.com)
- **Credenziali**: `checchi@me.com` / `Dms2022!`
- **Database**: MySQL (legacy)

Questo sistema deve essere considerato **READ-ONLY**. L'obiettivo è mappare le sue funzionalità e i suoi dati per la migrazione verso il nuovo DMS-HUB, non di modificarlo.
