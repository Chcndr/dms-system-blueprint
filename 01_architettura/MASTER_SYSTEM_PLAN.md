# MASTER SYSTEM PLAN - DMS HUB + MIO HUB

**Ultima Modifica:** 29 Novembre 2025
**Versione:** 3.0 (Post-Consolidamento)

---

## 1. Architettura Generale

Il sistema DMS/MIO-hub è un ecosistema complesso che integra diversi componenti per fornire una piattaforma unificata per la gestione dei mercati digitali e l'orchestrazione di agenti AI.

### Documentazione di Riferimento

- **[Infrastruttura AS-IS](./AS-IS/INFRASTRUTTURA_DMS_MIOHUB.md)**: Fotografia completa e verificata dei componenti reali, server, configurazioni e problemi aperti.
- **[Stato Endpoint](./AS-IS/ENDPOINTS_DMS_MIOHUB.md)**: Tabella di verità operativa per tutti gli endpoint DMS/MIO-hub, con stato reale e note sugli errori.
- **[Credenziali e Accessi](../CREDENZIALI_E_ACCESSI.md)**: Gestione sicura di tutte le credenziali di accesso ai servizi.

---

## 2. Roadmap di Stabilizzazione

Questa roadmap, basata sull'analisi AS-IS, definisce i passi necessari per stabilizzare l'infrastruttura e i servizi.

### Fase 1 – Congelare l’Esistente (SUBITO)

1.  **Blueprint come Unica Verità**: Questo documento e i file collegati sono l'unica fonte di verità. Nessuna modifica all'infrastruttura può essere fatta senza prima aggiornare questo blueprint.
2.  **Audit Endpoint Completato**: Lo stato di tutti gli endpoint è stato verificato e documentato. Gli endpoint critici per la PA (mercati, posteggi, operatori, GIS) sono stati identificati.
3.  **Regole Agenti AI**: Gli agenti (MIO, Manus, Abacus) devono seguire regole rigide, utilizzando solo endpoint reali e dati verificati, senza simulazioni.

### Fase 2 – Mettere in Linea la Macchina Tecnica

1.  **Fix Endpoint Rotti**: Risolvere gli errori SQL negli endpoint `/api/dmsHub/stalls/listByMarket` e `/api/dmsHub/vendors/list`.
2.  **Fix Configurazione Frontend**: Correggere l'URL dell'API nel file `.env.production` del frontend per puntare a `https://mihub.157-90-29-66.nip.io`.
3.  **Automazione Deploy**: Implementare un webhook GitHub → Hetzner per automatizzare il deploy del backend, eliminando la necessità di interventi manuali via SSH.
4.  **Multi-agente “Vero”**: Collegare i log `internalTraces` alla vista a 4 agenti nella Dashboard PA per avere un monitoraggio reale delle operazioni interne.

### Fase 3 – Funzioni Prodotto (Stabili e Documentate)

1.  **Anello Base PA**: Stabilizzare e rendere pienamente funzionanti gli endpoint per la gestione di Mercati, Posteggi e Operatori, con dati reali provenienti da Neon.
2.  **Cronologia e Task MIO**: Implementare la persistenza delle conversazioni e dei task dell'agente MIO su Neon.
3.  **Integrazioni Esterne**: Procedere con le integrazioni (PDND, PagoPA, ecc.) solo dopo che il nucleo DMS/MIHUB è stabile e completamente documentato.

---

## 3. Regole di Aggiornamento

Ogni modifica a URL, server, processi PM2, endpoint o al modulo GIS **deve** essere accompagnata da un aggiornamento di questo blueprint (master plan + file AS-IS) nello stesso Pull Request. 
