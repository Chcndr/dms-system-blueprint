# 🧭 Playbook Ufficiale Deploy DMS HUB

Questo documento definisce il flusso standard per ogni deploy relativo all'ecosistema DMS / MIHUB. L'obiettivo è garantire procedure coerenti, sicure e tracciabili, evitando deploy "a intuito".

## 0. Regola d'oro: Leggere sempre il Blueprint prima

Prima di qualunque deploy, è **obbligatorio** consultare il [repository blueprint DMS](https://github.com/Chcndr/dms-system-blueprint) per verificare:

- **Repository Associati:**
  - Frontend Vercel: `dms-hub-app-new`
  - Backend Hetzner: `mihub-backend-rest`
- **Stato Endpoint:** Quali endpoint devono essere attivi su Hetzner, quali sull'orchestratore e quali sono solo mock.
- **Credenziali e Host:** Informazioni di accesso a server e servizi.

## 1️⃣ Deploy FRONTEND su Vercel (dms-hub-app-new)

**Obiettivo:** Ogni merge sul branch `master` del repository frontend deve riflettersi sul sito Vercel.

### Passi Standard

1.  **Verifica Progetto Vercel:**
    -   **Progetto:** `dms-hub-app-new`
    -   **Branch Collegato:** `master`

2.  **Deploy Automatico (da GitHub):**
    -   Effettuare `merge` o `commit` sul branch `master`.
    -   Vercel triggera automaticamente un nuovo deploy.
    -   **Verifica Build:** Dal pannello Vercel, controllare i log della build per escludere errori.

3.  **Re-deploy Manuale:**
    -   Utile per modifiche a configurazione, DNS o variabili d'ambiente.
    -   Da Vercel → `dms-hub-app-new` → `Deployments` → clic sul branch → `Redeploy`.

### Check Post-Deploy (Obbligatori)

-   **Dashboard PA:** La pagina deve aprirsi senza errori JavaScript in console.
-   **API Dashboard:** La sezione `Integrazioni` → `API Dashboard` deve caricare correttamente la lista degli endpoint (validando `api/index.json`).
-   **Logs:** I tab `Logs` e `Guardian Logs` devono mostrare dati reali e non mock.

## 2️⃣ Deploy BACKEND MIHUB su Hetzner (mihub-backend-rest)

**Endpoint:** `https://mihub.157-90-29-66.nip.io/api`

### Playbook

1.  **SSH su Hetzner:**
    -   Usare le credenziali documentate nel blueprint.
    -   `ssh root@157.90.29.66`
    -   `cd /root/mihub-backend-rest`

2.  **Aggiornare Codice:**
    -   `git status` (verificare che non ci siano file locali non tracciati)
    -   `git pull`

3.  **Install / Build:**
    -   Se `package.json` è cambiato: `npm install` (o `yarn install`).
    -   Se il progetto ha una fase di build: `npm run build`.

4.  **Restart Servizio (PM2):**
    -   `pm2 list` (identificare il processo, es. `mihub-backend-rest`)
    -   `pm2 restart mihub-backend-rest`
    -   `pm2 logs mihub-backend-rest --lines 50` (controllare errori)

5.  **Health Check Backend:**
    -   `curl https://mihub.157-90-29-66.nip.io/api/gis/health`
    -   `curl https://mihub.157-90-29-66.nip.io/api/dmsHub/markets/list`
    -   `curl https://mihub.157-90-29-66.nip.io/api/markets/1/companies`

**Regola:** Ogni modifica a file in `routes/` richiede sempre il ciclo completo: `git pull` → `npm install` (se serve) → `pm2 restart` → `curl` test.

## 3️⃣ Migrazioni DB Neon (schema DMS)

**Regola:** Nessuna migrazione "al volo" senza file `.sql` versionato nel blueprint.

### Procedura

1.  **File di Migrazione:**
    -   Creare un file `YYYYMMDD_add_xxx.sql` nella cartella `/sql` del blueprint.
    -   Includere descrizione della migrazione e script di rollback.

2.  **Esecuzione:**
    -   `psql "$DATABASE_URL" -f 20251125_add_pdnd_tables.sql` (eseguire con le credenziali corrette).

3.  **Verifica Post-Migrazione:**
    -   Eseguire una query di controllo (es. `SELECT count(*) FROM stalls WHERE is_active = true;`).
    -   Aggiornare il blueprint con lo stato "eseguito".

## 4️⃣ Allineamento Guardian / API Dashboard (api/index.json)

**Regola:** Ogni nuovo endpoint REST in MIHUB deve essere registrato in `api/index.json`.

### Procedura

1.  **Modifica `api/index.json`:**
    -   Aprire il file nel repository del blueprint.
    -   Aggiungere la definizione del nuovo endpoint (path, method, category, status, etc.).

2.  **Commit e Push.**

3.  **Test:**
    -   **API Dashboard:** Verificare che l'endpoint compaia nella lista.
    -   **Guardian Logs:** Verificare che le chiamate all'endpoint vengano tracciate.

**Nota:** Se `api/index.json` non è aggiornato, il sistema restituirà l'errore `This endpoint is not available.`

## 5️⃣ Log & Debug

Ogni nuova feature o endpoint deve essere visibile e testabile nelle sezioni di logging e debug.

### Checklist

-   **Nuovi Endpoint:**
    -   Devono essere loggati dal middleware (già presente).
    -   Devono avere una categoria coerente in `api/index.json`.
-   **Nuovi Moduli:**
    -   Aggiornare la sezione `Debug / API Playground` per consentire test da UI.
    -   Aggiornare il blueprint con esempi di payload.
