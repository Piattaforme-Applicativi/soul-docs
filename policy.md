---
title: Principi guida per lo sviluppo
description: Principi guida per lo sviluppo delle applicazioni rilasciate in Unipd
published: true
date: 2025-06-27T09:08:19.402Z
tags: 
editor: markdown
dateCreated: 2025-06-27T08:57:46.640Z
---

# Principi generali

ASIT adotta una politica di sviluppo software open source basata sui principi di riuso, trasparenza ed efficienza. Questa policy è rivolta a tutti i collaboratori dell’Università di Padova che si occupano di sviluppo software, sia personale interno dell’Ateneo, sia fornitori esterni. 
Il presente documento intende essere una linea guida ASIT, dove si esplicitano le norme adottate sia nella fase di progettazione e sviluppo dei servizi digitali, sia le indicazioni specifiche del contesto di design system UI, finalizzate a mantenere coerenza visiva e funzionale tra team di design e sviluppo (working agreements).
Le linee guida ASIT sono prettamente orientate alle linee guida AGID, includendo anche ulteriori contributi che esulano dalle indicazioni nazionali, ma che risultano più efficaci per le finalità implementative di ASIT.
Relativamente al dispiegamento e al versionamento, si fa riferimento agli ambienti cloud di Ateneo, con rilascio negli ambienti di staging e produzione in modo automatizzato secondo azioni CI/CD definite da ASIT nell’ambiente collaborativo GitHub.
Oltre agli aspetti strettamente correlati alla progettazione e allo sviluppo, le policy contengono anche indicazioni relative ai temi della sicurezza, del monitoraggio e del miglioramento continuo, per garantire che il software sia sicuro, conforme e affidabile.

## Modello di DevOps

### Workflow di dispiegamento e versionamento
- Utilizzo di strumenti DevOps per l'integrazione e il deployment continuo (CI/CD).
- Adozione di tecnologie cloud per scalabilità e flessibilità.
- Standardizzazione degli ambienti di sviluppo e produzione.

La policy di sviluppo si fonda su un workflow CI/CD automatizzato principalmente tramite GitHub Actions, utilizzando GitHub come piattaforma di repository. La CD si avvale in modo importante di Docker per la containerizzazione e Terraform per l'IaC, con il deployment gestito da un orchestrator (ECS/EKS). La gestione delle configurazioni è centralizzata e sicura con AWS SSM Parameter Store. Il monitoraggio è affidato allo stack Grafana. Strumenti come Apache APISIX, Casbin/OPA e un sistema di tagging supportano l'infrastruttura e la sicurezza.

### Gestione del Codice e Collaborazione
- Utilizzo di repository pubblici (es. GitHub, GitLab) per la gestione del codice.
- Adozione di un modello di branching chiaro (es. GitFlow).
- Revisione del codice tramite pull request e code review.

### Licenze Open Source e Documentazione
- Rilascio del software con licenze open source compatibili OSI (es. MIT, Apache 2.0).
- Documentazione completa e accessibile del codice e delle API.
- Pubblicazione del codice sorgente su piattaforme accessibili.

## Sicurezza, Monitoraggio e Miglioramento Continuo
### Principi e pratiche
Nello sviluppo di soluzioni software dedicate ai processi di Ateneo, devono essere adottati principi e pratiche in linea con il Sistema di Gestione per la Sicurezza delle Informazioni (SGSI). In particolare, si richiede di garantire:

#### Sicurezza delle Informazioni (Information Security by Design)
* Confidenzialità, integrità e disponibilità delle informazioni devono essere garantite fin dalle fasi iniziali di progettazione (Secure by Design);
* Devono essere applicate le misure di sicurezza tecniche e organizzative adeguate, in base all’analisi del rischio;
* È richiesto l’uso di cifratura, controlli di accesso, audit trail, e altre misure coerenti con le buone pratiche di sicurezza;
* Le dipendenze di terze parti (librerie, moduli, API) devono essere verificate e mantenute aggiornate per prevenire vulnerabilità note.

#### Monitoraggio e Logging
I sistemi devono implementare meccanismi di logging sicuro e monitoraggio degli eventi rilevanti per la sicurezza, in conformità con le politiche di audit dell’Ateneo. I log devono essere:
* Protetti contro alterazioni non autorizzate;
* Conservati secondo i tempi stabiliti dalla normativa vigente e dalle policy interne;
* Analizzati periodicamente per individuare comportamenti anomali o incidenti di sicurezza.

#### Gestione degli Incidenti e delle Vulnerabilità
Devono essere predisposte procedure per la segnalazione, gestione e tracciamento degli incidenti di sicurezza. È richiesto un processo strutturato di vulnerability management, comprensivo di:
* Scansione periodica del software;
* Trattazione e risoluzione tempestiva delle vulnerabilità identificate;
* Documentazione delle attività correttive effettuate.

#### Miglioramento Continuo e Conformità
Il software deve essere soggetto a revisioni periodiche, anche post rilascio, per assicurare la continua conformità alle policy di sicurezza e alle normative applicabili. Il processo di sviluppo deve includere:
* Code review e test di sicurezza (es. penetration test, SAST/DAST);
* Meccanismi di feedback e analisi post-mortem in caso di incidenti;
* Tutte le attività devono contribuire al ciclo di miglioramento continuo (Plan-Do-Check-Act) previsto dal SGSI.

### Pratiche operative e controlli ISO/IEC 27001
Di seguito si dà evidenza dei riferimenti all’Allegato A della ISO/IEC 27001 per ciascun punto (rif. ISO/IEC 27001:2022 - Allegato A).

#### Principi di Sicurezza Applicativa
[Controlli ISO: A.5.23, A.8.25, A.8.26, A.8.28]
* Progettazione secondo principi Secure by Design e Secure by Default;
* Accesso ai dati basato su minimo privilegio e separazione dei ruoli (A.8.2 - Identity management);
* Autenticazione sicura, preferibilmente federata e/o multi-fattore (A.8.4 - Authentication information);
* Protezione dei dati sensibili mediante cifratura in transito e a riposo (A.8.24 - Use of cryptography);
* Adozione di strumenti per l’analisi del codice e la gestione delle dipendenze (A.8.25 - Secure development life cycle).

#### Logging e Monitoraggio
[Controlli ISO: A.5.17, A.8.15, A.8.16, A.8.20]
* Implementazione di log per operazioni critiche (accessi, modifiche, errori, ecc.);
* Log protetti da modifiche non autorizzate, con dettagli minimi standardizzati;
* Integrazione con sistemi di monitoraggio centralizzato e alerting, dove previsto;
* Log conservati e analizzati secondo le policy di audit dell’Ateneo (A.5.17 - Monitoring activities, A.8.16 - Monitoring systems).

#### Gestione delle Vulnerabilità e Patch
[Controlli ISO: A.8.29, A.5.25, A.8.8]
Monitoraggio attivo delle vulnerabilità (CVE) dei componenti di terze parti.
* Applicazione tempestiva di patch secondo le priorità definite (A.8.29 - Information security testing in development and acceptance);
* Uso consigliato di strumenti di Software Composition Analysis (SCA);
* Evidenza documentata delle attività di aggiornamento e mitigazione (A.5.25 - Technical compliance review).

#### Gestione degli Incidenti
[Controlli ISO: A.5.24, A.5.28, A.5.31]
* Obbligo di notifica immediata in caso di incidente di sicurezza che impatti i sistemi o i dati trattati;
* Collaborazione con l’Ateneo per analisi, contenimento, e reporting dell’incidente;
* Mantenimento di un registro degli incidenti a disposizione per eventuali audit (A.5.24 - Information security event reporting);

#### Ciclo di Vita Sicuro e Miglioramento Continuo
[Controlli ISO: A.5.30, A.8.25, A.8.26, A.8.27]
Adozione di un ciclo di sviluppo sicuro (SSDLC), comprensivo di:
* Analisi del rischio in fase di progettazione;
* Code review, test automatizzati e test di sicurezza (A.8.27 - Secure coding);
* Rilascio controllato e tracciabile (A.8.26 - Secure system architecture and engineering principles);
* Verifiche post-rilascio e piani di miglioramento documentati;
* Disponibilità a sottoporsi a audit tecnici e test di sicurezza indipendenti (A.5.30 - ICT readiness for business continuity).

## Design System UI di Unipd
L'Università di Padova adotterà un proprio design system UI ispirato alle linee guida AgID, ma con contenuti e componenti aggiuntivi personalizzati. Questo design system garantirà:
- Coerenza visiva e funzionale tra le applicazioni sviluppate;
- Accessibilità e usabilità per tutti gli utenti;
- Componenti riutilizzabili e documentati per velocizzare lo sviluppo.
Design System per dotare ASIT UNIPD di un proprio linguaggio visivo, aderente alle linee guida di identità visiva prodotte da ACOM

## Piano di Implementazione e Revisione
- Definizione di un piano di implementazione graduale della policy;
- Revisione periodica della policy per adattarla alle nuove esigenze e tecnologie;
- Coinvolgimento attivo di tutto il personale tecnico-amministrativo e docente nel processo di miglioramento.

