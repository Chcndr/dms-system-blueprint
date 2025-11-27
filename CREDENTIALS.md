# 🔐 Credenziali e Accessi Sistema DMS/MIHUB

**⚠️ DOCUMENTO SENSIBILE - NON CONDIVIDERE PUBBLICAMENTE**

---

## 🖥️ Server Hetzner (Backend MIHUB)

### Informazioni Server

| Campo | Valore |
| :--- | :--- |
| **IP Pubblico** | `157.90.29.66` |
| **Hostname** | `orchestratore.mio-hub.me` |
| **OS** | Ubuntu 22.04.5 LTS |
| **User** | `root` |

### Accesso SSH

**Metodo 1: SSH Key (RACCOMANDATO)**
```bash
ssh -i ~/.ssh/hetzner_server_key root@157.90.29.66
```

**Metodo 2: Password (Backup)**
```bash
ssh root@157.90.29.66
# Password: tkFxkgnT4ieh
```

**Posizione Chiave SSH:**
- Path nel sandbox: `/home/ubuntu/.ssh/hetzner_server_key`
- Tipo: ed25519
- Permessi: `600` (solo lettura per owner)

### Directory Principali

| Directory | Contenuto |
| :--- | :--- |
| `/root/mihub-backend-rest/` | Backend MIHUB REST API |
| `/root/.pm2/` | PM2 process manager |
| `/root/.pm2/logs/` | Log applicazioni PM2 |

---

## 🔑 Secrets e API Tokens

I seguenti secrets sono configurati nel **DMS Hub** → **Configurazioni** → **API & Agent Tokens**:

### GitHub Personal Access Token

- **Nome nel sistema:** `GITHUB_PERSONAL_ACCESS_TOKEN`
- **Scope:** `repo` (accesso completo ai repository)
- **Usato da:** Agente Abacus (via proxy MIHUB)
- **Stato:** ✅ Configurato
- **Ultime 4 cifre:** `airq`
- **Aggiornato:** 27/11/2025, 04:51:26

### Vercel Token

- **Nome nel sistema:** `VERCEL_TOKEN`
- **Scope:** `deploy`
- **Usato da:** Deploy automatici frontend
- **Stato:** ⚠️ Non configurato

### TPER API Key

- **Nome nel sistema:** `TPER_API_KEY`
- **Scope:** `mobility`
- **Usato da:** Integrazioni trasporto pubblico
- **Stato:** ⚠️ Non configurato

**Nota:** I token sono cifrati con AES-256-GCM usando una master key sul server prima di essere salvati nel database.

---

## 🗄️ Database PostgreSQL (Neon)

### Connessione

| Campo | Valore |
| :--- | :--- |
| **Host** | `ep-bold-silence-a2w3xqbw.eu-central-1.aws.neon.tech` |
| **Database** | `neondb` |
| **User** | `neondb_owner` |
| **Password** | `npg_lYG6JQ5Krtsi` |
| **SSL Mode** | `require` |
| **Channel Binding** | `require` |

### Connection String

```
postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-a2w3xqbw.eu-central-1.aws.neon.tech/neondb?sslmode=require
```

### Console Web

- **URL:** https://console.neon.tech
- **Email:** `chcndr@gmail.com`
- **Password:** (verificare con utente)

---

## ☁️ Vercel (Frontend)

### Account

- **URL:** https://vercel.com/dashboard
- **Email:** `chcndr@gmail.com`
- **Password:** (verificare con utente)

### Progetti Deployati

| Progetto | URL | Repository |
| :--- | :--- | :--- |
| **Dashboard PA** | https://dms-hub-app-new.vercel.app | `Chcndr/dms-hub-app-new` |
| **App Cittadini** | https://dms-cittadini.vercel.app | `Chcndr/dms-cittadini` |
| **App Operatore** | https://dms-operatore.vercel.app | `Chcndr/dms-operatore` |

---

## 🐙 GitHub

### Account

- **Username:** `Chcndr`
- **Email:** `chcndr@gmail.com`

### Repository Principali

| Repository | Descrizione |
| :--- | :--- |
| `Chcndr/dms-system-blueprint` | Blueprint e documentazione sistema |
| `Chcndr/mihub-backend-rest` | Backend MIHUB REST API |
| `Chcndr/dms-hub-app-new` | Frontend Dashboard PA |
| `Chcndr/MIO-hub` | Catalogo API e configurazioni |

---

## 🔒 Note di Sicurezza

- **Master Key Secrets:** Configurata in `/root/mihub-backend-rest/.env` come `SECRETS_MASTER_KEY`
- **Backup Credenziali:** Disponibile in `/home/ubuntu/upload/mihub_credentials(3).pdf`
- **Rotazione Token:** I token GitHub e Vercel dovrebbero essere ruotati ogni 90 giorni
- **Accesso SSH:** Preferire sempre la chiave SSH alla password

---

## 📚 Riferimenti

- **Guida Deploy Backend:** `07_guide_operative/DEPLOY_BACKEND_HETZNER.md`
- **Guida Deploy Frontend:** `07_guide_operative/DEPLOY_FRONTEND_VERCEL.md`
- **Architettura Sistema:** `01_architettura/MASTER_SYSTEM_PLAN.md`
- **Credenziali Legacy:** `01_architettura/legacy/root_legacy/CREDENZIALI_E_ACCESSI.md`

---

**Ultimo Aggiornamento:** 27 Novembre 2025  
**Autore:** Manus AI
