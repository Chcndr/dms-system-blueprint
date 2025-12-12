# 🎭 Sistema Doppio Canale FRONTSTAGE/BACKSTAGE

**Data Implementazione**: 12 Dicembre 2025  
**Versione**: 1.0  
**Status**: Implementato, in attesa di deploy

---

## 📋 Panoramica

Il **Sistema Doppio Canale** è un'architettura che permette agli agenti AI del MIO Hub di comportarsi in modo diverso a seconda del contesto di comunicazione:

- **FRONTSTAGE** (User ↔ Agente): Comunicazione diretta con l'utente finale
- **BACKSTAGE** (MIO ↔ Agente): Coordinamento interno tra MIO e gli agenti specializzati

---

## 🎯 Obiettivo

**Problema**:
Gli agenti usavano lo stesso tono e stile sia quando parlavano con gli utenti che quando venivano orchestrati da MIO, risultando:
- Troppo tecnici e freddi con gli utenti
- Troppo verbosi e "chiacchieroni" nel coordinamento interno

**Soluzione**:
Implementare due "modalità" di comportamento:
1. **FRONTSTAGE**: Tono conversazionale, amichevole, professionale (come un collega esperto)
2. **BACKSTAGE**: Tono tecnico, esecutivo, sintetico, orientato ai dati/JSON

---

## 🏗️ Architettura

### Flusso di Determinazione Modalità

```
Richiesta → orchestrator.js
    ↓
    Determina conversationId
    ↓
    Chiama callAgent(agent, message, history, toolExecutor, conversationId)
    ↓
llm.js → callAgent()
    ↓
    Analizza conversationId:
    ├─ Contiene "mio-{agent}-coordination"? → BACKSTAGE
    └─ Altrimenti → FRONTSTAGE
    ↓
    Inietta istruzione di modalità nel messaggio
    ↓
    Chiama LLM (Gemini)
    ↓
    Risposta con tono appropriato
```

### Conversation ID e Modalità

| Conversation ID | Modalità | Comportamento |
|----------------|----------|---------------|
| `manus-single` | FRONTSTAGE | Conversazionale |
| `user-manus-direct` | FRONTSTAGE | Conversazionale |
| `mio-manus-coordination` | BACKSTAGE | Tecnico/Esecutivo |
| `zapier-single` | FRONTSTAGE | Conversazionale |
| `user-zapier-direct` | FRONTSTAGE | Conversazionale |
| `mio-zapier-coordination` | BACKSTAGE | Tecnico/Esecutivo |
| `abacus-single` | FRONTSTAGE | Conversazionale |
| `mio-abacus-coordination` | BACKSTAGE | Tecnico/Esecutivo |
| `gptdev-single` | FRONTSTAGE | Conversazionale |
| `mio-gptdev-coordination` | BACKSTAGE | Tecnico/Esecutivo |

---

## 💻 Implementazione Tecnica

### File Modificati

#### 1. `llm.js` - Funzione `callAgent()`

**Modifiche**:
- Aggiunto parametro `conversationId` alla firma della funzione
- Implementata logica di rilevamento modalità
- Iniezione dinamica istruzioni FRONTSTAGE/BACKSTAGE

**Codice**:
```javascript
async function callAgent(agent, message, conversationHistory = [], toolExecutor = null, conversationId = null) {
  // 🎯 DOPPIO CANALE: Determina modalità (FRONTSTAGE vs BACKSTAGE)
  let channelMode = 'FRONTSTAGE'; // Default: chat diretta con user
  let modeInstruction = '';
  
  // Analizza conversationId per determinare il canale
  if (conversationId) {
    if (conversationId.includes('mio-') && conversationId.includes('-coordination')) {
      // BACKSTAGE: MIO sta orchestrando questo agente
      channelMode = 'BACKSTAGE';
      modeInstruction = `⚠️ MODALITÀ BACKSTAGE: Stai parlando con l'Orchestratore MIO. Rispondi SOLO con dati tecnici/JSON. Sii sintetico. Non usare toni conversazionali. Esegui i tool richiesti senza chiedere conferme.`;
      console.log(`[LLM] 🔀 DOPPIO CANALE: ${agent} in modalità BACKSTAGE (coordinamento MIO)`);
    } else {
      // FRONTSTAGE: User sta parlando direttamente con l'agente
      channelMode = 'FRONTSTAGE';
      modeInstruction = `👋 MODALITÀ FRONTSTAGE: Stai parlando con l'Utente Finale. Sii utile, chiaro e usa un tono professionale ma umano. Spiega cosa fai e perché.`;
      console.log(`[LLM] 💬 DOPPIO CANALE: ${agent} in modalità FRONTSTAGE (chat diretta user)`);
    }
  } else {
    // Nessun conversationId fornito, assume FRONTSTAGE
    modeInstruction = `👋 MODALITÀ FRONTSTAGE: Stai parlando con l'Utente Finale. Sii utile, chiaro e usa un tono professionale ma umano. Spiega cosa fai e perché.`;
    console.log(`[LLM] 💬 DOPPIO CANALE: ${agent} in modalità FRONTSTAGE (default)`);
  }
  
  // Aggiungi il messaggio corrente CON istruzione di modalità
  const enhancedMessage = modeInstruction ? `${modeInstruction}\n\n${message}` : message;
  messages.push({
    role: 'user',
    content: enhancedMessage
  });
  
  // ... resto della funzione
}
```

#### 2. `orchestrator.js` - Chiamate a `callAgent()`

**Modifiche**: Passare `conversationId` a tutte le chiamate

**A. Routing Zapier** (riga ~686):
```javascript
// 🎯 DOPPIO CANALE: Passa conversationId per determinare modalità
const zapierConvId = (sender === 'mio') ? 'mio-zapier-coordination' : 'user-zapier-direct';
zapierResponse = await callAgent('zapier', message, conversationHistory, zapierToolExecutor, zapierConvId);
```

**B. Tool Executor MIO** (riga ~840):
```javascript
// 🎯 DOPPIO CANALE: Passa coordinationConversationId per modalità BACKSTAGE
agentResponse = await callAgent(targetAgent, taskMessage, [], null, coordinationConversationId);
```

**C. Routing Normale** (riga ~871):
```javascript
// 🎯 DOPPIO CANALE: Passa currentConversationId per determinare modalità
responseMessage = await callAgent(agent, message, conversationHistory, toolExecutor, currentConversationId);
```

---

## 📊 Esempi di Comportamento

### Esempio 1: FRONTSTAGE (User → Manus)

**Input**:
```
User: "Ciao Manus, come sta il server?"
conversationId: "manus-single"
```

**Istruzione Iniettata**:
```
👋 MODALITÀ FRONTSTAGE: Stai parlando con l'Utente Finale. Sii utile, chiaro e usa un tono professionale ma umano. Spiega cosa fai e perché.

Ciao Manus, come sta il server?
```

**Risposta Attesa**:
```
Ciao! Il server sta funzionando bene. Ho controllato PM2 e vedo che il backend è online da 14 minuti, con un consumo di CPU dello 0% e 81.3MB di memoria. L'uptime generale del server è di 7 giorni e 1 ora. Tutto sembra stabile! C'è qualcosa di specifico che vuoi che controlli?
```

### Esempio 2: BACKSTAGE (MIO → Manus)

**Input**:
```
MIO: "Controlla PM2 e uptime"
conversationId: "mio-manus-coordination"
```

**Istruzione Iniettata**:
```
⚠️ MODALITÀ BACKSTAGE: Stai parlando con l'Orchestratore MIO. Rispondi SOLO con dati tecnici/JSON. Sii sintetico. Non usare toni conversazionali. Esegui i tool richiesti senza chiedere conferme.

Controlla PM2 e uptime
```

**Risposta Attesa**:
```json
{
  "pm2": {
    "status": "online",
    "uptime": "14m",
    "cpu": "0%",
    "memory": "81.3MB"
  },
  "server_uptime": "7d 1h 55m"
}
```

### Esempio 3: FRONTSTAGE (User → Zapier)

**Input**:
```
User: "Quali automazioni puoi fare?"
conversationId: "zapier-single"
```

**Risposta Attesa**:
```
Ciao! Posso aiutarti con diverse automazioni. Ecco le azioni disponibili:

📱 **WhatsApp Notifications**
- Inviare messaggi WhatsApp

📧 **Gmail**
- Inviare email
- Leggere email

📅 **Google Calendar**
- Creare eventi
- Aggiornare eventi

📄 **Google Docs**
- Creare documenti
- Aggiornare documenti

Quale di queste vuoi automatizzare? Dimmi pure cosa ti serve!
```

### Esempio 4: BACKSTAGE (MIO → Zapier)

**Input**:
```
MIO: "Elenca azioni disponibili"
conversationId: "mio-zapier-coordination"
```

**Risposta Attesa**:
```json
{
  "actions": [
    {"id": "39e1ab65-24e6-4efb...", "name": "WhatsApp Notifications: Send Message"},
    {"id": "7f2c9a18-53d1-4bca...", "name": "Gmail: Send Email"},
    {"id": "8e4d1b29-64e2-5cdb...", "name": "Google Calendar: Create Event"},
    {"id": "9f5e2c3a-75f3-6dec...", "name": "Google Docs: Create Document"}
  ],
  "total": 6
}
```

---

## 🧪 Testing

### Test Case 1: Chat Singola Manus (FRONTSTAGE)

**Comando**:
```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/mihub/orchestrator \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "manual",
    "targetAgent": "manus",
    "message": "Ciao Manus, come sta il server?"
  }'
```

**Verifica**:
- Log deve mostrare: `[LLM] 💬 DOPPIO CANALE: manus in modalità FRONTSTAGE`
- Risposta deve essere conversazionale e amichevole

### Test Case 2: Vista 4 Agenti (BACKSTAGE)

**Comando**:
```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/mihub/orchestrator \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "manual",
    "targetAgent": "mio",
    "message": "Manus, controlla PM2"
  }'
```

**Verifica**:
- Log deve mostrare: `[LLM] 🔀 DOPPIO CANALE: manus in modalità BACKSTAGE`
- MIO deve delegare a Manus tramite tool `delegate_to_agent`
- Risposta di Manus deve essere tecnica e sintetica

---

## 📈 Benefici

### Per gli Utenti
- ✅ Agenti più amichevoli e facili da capire
- ✅ Spiegazioni chiare di cosa fanno e perché
- ✅ Tono professionale ma umano
- ✅ Migliore esperienza utente complessiva

### Per il Sistema
- ✅ Comunicazioni interne più efficienti
- ✅ Riduzione "chiacchiere" nel coordinamento
- ✅ Risposte più rapide tra MIO e agenti
- ✅ Formato dati strutturato (JSON) per parsing automatico

### Per lo Sviluppo
- ✅ Implementazione non invasiva (iniezione dinamica)
- ✅ Nessuna modifica ai system prompt esistenti
- ✅ Facile da debuggare (logging chiaro)
- ✅ Estensibile a nuovi agenti

---

## 🔧 Troubleshooting

### Problema: Agente non cambia tono

**Causa**: `conversationId` non viene passato correttamente

**Soluzione**:
1. Verificare che `orchestrator.js` passi `conversationId` a `callAgent()`
2. Controllare i log per vedere quale modalità viene rilevata
3. Verificare che `conversationId` segua il formato corretto

### Problema: Log non mostra "DOPPIO CANALE"

**Causa**: Deploy non ancora avvenuto o PM2 non riavviato

**Soluzione**:
```bash
ssh root@157.90.29.66
cd /root/mihub-backend-rest
git pull origin master
pm2 restart mihub-backend --update-env
pm2 logs mihub-backend --lines 50
```

---

## 📝 Note Implementative

### Perché Iniezione Dinamica?

**Alternativa considerata**: Creare due system prompt per ogni agente (`manus` e `manus_coordination`)

**Problema**:
- Duplicazione massiva del codice
- Difficile mantenere sincronizzati i prompt
- Rischio di corrompere il file `llm.js` (troppo grande)

**Soluzione scelta**: Iniezione dinamica
- Modifiche minimali al codice
- Nessuna duplicazione
- Facile da testare e debuggare
- Non tocca l'integrazione Gemini

### Compatibilità Retroattiva

Il sistema è **retrocompatibile**:
- Se `conversationId` non viene passato → assume FRONTSTAGE
- Chiamate esistenti continuano a funzionare
- Nessun breaking change

---

## 🚀 Deploy

### Checklist Pre-Deploy

- [x] Modifiche committate su GitHub
- [x] Test locali eseguiti
- [x] Blueprint aggiornato
- [ ] PM2 riavviato su Hetzner
- [ ] Test in produzione
- [ ] Verifica logs

### Comandi Deploy

```bash
# 1. Pull modifiche
cd /root/mihub-backend-rest
git pull origin master

# 2. Restart PM2
pm2 restart mihub-backend --update-env

# 3. Verifica logs
pm2 logs mihub-backend --lines 50 | grep "DOPPIO CANALE"

# 4. Test endpoint
curl -X POST https://mihub.157-90-29-66.nip.io/api/mihub/orchestrator \
  -H "Content-Type: application/json" \
  -d '{"mode":"manual","targetAgent":"manus","message":"test"}'
```

---

## 📚 Riferimenti

- **Commit**: `d0f3e63`
- **File modificati**:
  - `src/modules/orchestrator/llm.js` (+28 righe)
  - `src/modules/orchestrator/orchestrator.js` (+6 righe)
- **Data**: 12 Dicembre 2025
- **Autore**: Manus AI Agent
- **Issue**: Sistema Doppio Canale FRONTSTAGE/BACKSTAGE

---

**Versione Documento**: 1.0  
**Ultima Modifica**: 12 Dicembre 2025  
**Status**: Implementato, in attesa di deploy
