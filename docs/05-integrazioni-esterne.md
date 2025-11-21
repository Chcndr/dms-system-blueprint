# 05. Integrazioni Esterne

Questo documento descrive le integrazioni del sistema DMS-HUB con servizi e applicazioni esterne.

## Integrazioni Principali

Il sistema è progettato per interagire con diversi componenti esterni, che sono fondamentali per il suo funzionamento.

### 1. Gestionale Legacy DMS (Heroku)

L'integrazione più critica è quella con il gestionale DMS esistente, ospitato su Heroku. Questo sistema contiene lo storico dei dati e delle logiche di business che devono essere progressivamente migrate nel nuovo DMS-HUB.

- **URL**: `https://lapsy-dms.herokuapp.com`
- **Stato**: Legacy, da trattare in **READ-ONLY**.
- **Obiettivo**: Mappare le funzionalità e i dati per la migrazione, garantendo al contempo la continuità operativa durante la transizione.

### 2. App Mobile per Ambulanti

Il sistema deve integrarsi con l'applicazione mobile utilizzata dagli operatori commerciali per la gestione delle presenze, delle concessioni e di altre attività operative. Le API del DMS-HUB dovranno essere compatibili con i requisiti di tale app.

### 3. Editor GIS

È prevista l'integrazione di un editor GIS (Geographic Information System) per la gestione visuale delle piante dei mercati, l'assegnazione dei posteggi e la pianificazione degli spazi. Questo componente sarà fondamentale per modernizzare la gestione logistica dei mercati.

### 4. API OpenAI

Il MIO-HUB, il sistema di agenti AI, utilizza le API di OpenAI per alimentare i suoi modelli linguistici e le sue capacità di analisi e generazione. Questa integrazione è al centro delle funzionalità di automazione e intelligenza del sistema.

### 5. Zapier

Zapier viene utilizzato come strumento di automazione per connettere il DMS-HUB con centinaia di altre applicazioni web, permettendo di creare workflow automatizzati per compiti come la notifica, la sincronizzazione dei dati e l'automazione dei processi di business.
