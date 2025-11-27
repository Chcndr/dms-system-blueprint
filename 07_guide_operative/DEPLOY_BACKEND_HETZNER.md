# Procedura Deploy Backend - mihub-backend-rest su Hetzner

**Data:** 2025-11-27
**Autore:** Manus AI

Questa guida descrive i passaggi per deployare una nuova versione del backend `mihub-backend-rest` sul server Hetzner.

## Prerequisiti

- Accesso SSH al server Hetzner (IP: `157.90.29.66`)
- Repository `mihub-backend-rest` clonato in locale
- Modifiche testate e committate su branch `master`

## Passaggi di Deploy

1. **Connessione SSH al server**
   ```bash
   ssh root@157.90.29.66
   ```

2. **Navigare nella directory del progetto**
   ```bash
   cd /root/mihub-backend-rest
   ```

3. **Pull delle modifiche da GitHub**
   ```bash
   git pull origin master
   ```

4. **Installare/aggiornare dipendenze**
   ```bash
   npm install
   ```

5. **Riavviare il servizio PM2**
   ```bash
   pm2 restart mihub-backend-rest
   ```

6. **Verificare lo stato del servizio**
   ```bash
   pm2 status mihub-backend-rest
   ```
   Assicurarsi che lo stato sia `online`.

7. **Controllare i log per eventuali errori**
   ```bash
   pm2 logs mihub-backend-rest --lines 100
   ```

## Rollback in caso di problemi

Se il deploy causa problemi, è possibile tornare al commit precedente:

1. **Trovare l'hash del commit precedente**
   ```bash
   git log -n 2
   ```

2. **Eseguire il checkout al commit stabile**
   ```bash
   git checkout <hash_commit_precedente>
   ```

3. **Riavviare il servizio PM2**
   ```bash
   pm2 restart mihub-backend-rest
   ```
