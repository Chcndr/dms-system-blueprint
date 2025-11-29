# Accesso Server Hetzner - Backend Mercati/GIS

## Informazioni Server
- **Host**: `157.90.29.66`
- **User**: `root`
- **Servizi**: Backend GIS, Orchestratore MIO, Database PostgreSQL

## Chiave SSH Ufficiale

### Path Chiave
```
/home/ubuntu/.ssh/hetzner_deploy_key
```

### Fingerprint
```
SHA256:Gsnr7z2Zy+ehoj0dvlTFHiFntLRbeP267ZlZJs2nNDk
```

### Tipo Chiave
- **Algoritmo**: ED25519
- **Dimensione**: 256 bit
- **Comment**: `hetzner-server-key`

### Data Creazione/Ultima Rotazione
**29 Novembre 2025**

## Comando Standard di Accesso

```bash
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66
```

### Test Connessione
```bash
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66 "pwd && whoami"
```

**Output Atteso**:
```
/root
root
```

## Chiave Pubblica sul Server

La chiave pubblica è salvata in `/root/.ssh/authorized_keys` sul server:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHxlJmQiMAoFNMDlXxay+SVSfi806N8FI+Ds8KTpeNSr hetzner-server-key
```

## Procedura di Rotazione Chiave

### 1. Generazione Nuova Chiave
```bash
ssh-keygen -t ed25519 -f ~/.ssh/hetzner_deploy_key_new -C "hetzner-deploy-key-$(date +%Y%m%d)" -N ""
```

### 2. Aggiunta Chiave Pubblica al Server
```bash
# Opzione A: Via SSH con chiave vecchia
cat ~/.ssh/hetzner_deploy_key_new.pub | ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66 "cat >> /root/.ssh/authorized_keys"

# Opzione B: Via Console Hetzner (modalità rescue)
# 1. Abilita modalità rescue dalla console web
# 2. Riavvia il server
# 3. Accedi con password rescue
# 4. Monta il disco principale
# 5. Aggiungi chiave pubblica a /mnt/root/.ssh/authorized_keys
# 6. Riavvia normalmente
```

### 3. Test Nuova Chiave
```bash
ssh -i ~/.ssh/hetzner_deploy_key_new root@157.90.29.66 "pwd && whoami"
```

### 4. Sostituzione Chiave
```bash
mv ~/.ssh/hetzner_deploy_key ~/.ssh/hetzner_deploy_key.old
mv ~/.ssh/hetzner_deploy_key_new ~/.ssh/hetzner_deploy_key
mv ~/.ssh/hetzner_deploy_key_new.pub ~/.ssh/hetzner_deploy_key.pub
```

### 5. Aggiornamento Documentazione
- Aggiornare fingerprint in questo documento
- Aggiornare data rotazione
- Aggiornare configurazione in Dashboard PA (Cassetto Chiavi)

### 6. Rimozione Chiave Vecchia dal Server
```bash
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66 "sed -i '/hetzner-server-key-OLD/d' /root/.ssh/authorized_keys"
```

## Gestione Secret

### Variabile d'Ambiente (Backend)
Se necessario per automazioni, salvare la chiave privata come secret:

```bash
# Codifica in base64
cat ~/.ssh/hetzner_deploy_key | base64 -w 0 > hetzner_key_b64.txt

# Salva in variabile d'ambiente backend
HETZNER_SSH_KEY_B64="<contenuto_base64>"
```

### Cassetto Chiavi Dashboard PA
La chiave è documentata nella Dashboard PA → Impostazioni → API & Tokens:

- **Nome**: `HETZNER_SSH_KEY`
- **Scope**: `infra`
- **Label**: "Hetzner SSH – Backend Mercati/GIS"
- **Descrizione**: Include host, user, path e fingerprint

## Servizi sul Server

### Backend GIS
- **Path**: `/root/mihub-backend-rest`
- **Processo PM2**: `mihub-backend`
- **Porta**: `3000`
- **Endpoint**: `https://mihub.157-90-29-66.nip.io/api/gis/*`

### Orchestratore MIO
- **Path**: `/root/mihub-backend-rest`
- **Processo PM2**: `mihub-backend`
- **Porta**: `3000`
- **Endpoint**: `https://orchestratore.mio-hub.me/api/mihub/orchestrator/*`

### Database PostgreSQL
- **Host**: Neon Cloud (non locale)
- **Connection String**: Vedere `CREDENZIALI E ACCESSI - DMS_MIO-HUB.pdf`

## Comandi Utili

### Verifica Stato Servizi
```bash
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66 "pm2 status"
```

### Visualizza Log
```bash
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66 "pm2 logs --lines 50"
```

### Deploy Backend
```bash
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66 "cd /root/mihub-backend-rest && git pull origin master && pm2 restart all"
```

### Verifica Endpoint GIS
```bash
curl -i https://mihub.157-90-29-66.nip.io/api/gis/health
```

## Troubleshooting

### Connessione Richiede Password
**Problema**: Il comando SSH chiede la password invece di usare la chiave.

**Causa**: La chiave pubblica non è presente in `/root/.ssh/authorized_keys` sul server.

**Soluzione**:
```bash
# Verifica chiave sul server
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66 "grep 'hetzner' /root/.ssh/authorized_keys"

# Se manca, aggiungila via console Hetzner (modalità rescue)
```

### Permessi Chiave Errati
**Problema**: SSH rifiuta la chiave con errore "permissions are too open".

**Soluzione**:
```bash
chmod 600 ~/.ssh/hetzner_deploy_key
chmod 644 ~/.ssh/hetzner_deploy_key.pub
```

### Chiave Non Trovata
**Problema**: SSH non trova la chiave in `~/.ssh/hetzner_deploy_key`.

**Soluzione**:
```bash
# Verifica presenza chiave
ls -la ~/.ssh/hetzner_deploy_key*

# Se manca, rigenerala seguendo la procedura di rotazione
```

## Note Importanti

⚠️ **NON salvare la chiave privata in repository Git**

⚠️ **NON condividere la chiave privata via email/chat**

⚠️ **Ruotare la chiave ogni 6 mesi** (prossima rotazione: Maggio 2026)

✅ **Usare SOLO questa chiave** per accesso Hetzner (niente password, niente altre chiavi)

✅ **Documentare ogni rotazione** in questo file e nel Cassetto Chiavi Dashboard PA

✅ **Testare sempre la nuova chiave** prima di rimuovere la vecchia
