# Procedura Deploy Frontend - dms-hub-app-new su Vercel

**Data:** 2025-11-27
**Autore:** Manus AI

Questa guida descrive i passaggi per deployare una nuova versione del frontend `dms-hub-app-new` su Vercel.

## Deploy Automatico

Vercel è configurato per il **deploy automatico** ad ogni push sul branch `master` del repository [Chcndr/dms-hub-app-new](https://github.com/Chcndr/dms-hub-app-new).

Non è richiesta alcuna azione manuale per il deploy.

## Passaggi di Sviluppo

1. **Clonare il repository**
   ```bash
   gh repo clone Chcndr/dms-hub-app-new
   ```

2. **Creare un nuovo branch per le modifiche**
   ```bash
   git checkout -b feat/nome-nuova-funzionalita
   ```

3. **Effettuare le modifiche al codice**

4. **Testare in locale**
   ```bash
   npm install
   npm run dev
   ```

5. **Commit e push del branch**
   ```bash
   git add .
   git commit -m "feat: descrizione modifiche"
   git push origin feat/nome-nuova-funzionalita
   ```

6. **Creare una Pull Request su GitHub**
   - Aprire il repository su GitHub
   - Creare una Pull Request dal nuovo branch verso `master`
   - Attendere l'approvazione e il merge

7. **Deploy Automatico**
   - Una volta che la Pull Request è stata approvata e mergiata su `master`, Vercel avvierà automaticamente il processo di build e deploy.
   - Il deploy sarà disponibile all'URL: https://dms-hub-app-new.vercel.app

## Monitoraggio Deploy

È possibile monitorare lo stato del deploy dalla dashboard di Vercel, nella sezione relativa al progetto `dms-hub-app-new`.
