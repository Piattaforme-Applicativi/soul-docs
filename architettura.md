---
title: Archittettura
description: Architettura delle applicazioni
published: true
date: 2025-04-29T13:08:22.149Z
tags: 
editor: markdown
dateCreated: 2025-03-31T07:59:50.309Z
---

# Architettura dei sistemi

Le applicazioni sviluppate a partire dallo [Starter Kit](https://github.com/Piattaforme-Applicativi/soul-starter-kit) ereditano un' **architettura monolitica** basata su container. L'architettura monolitica è indicata per progetti che hanno le seguenti caratteristiche:

- MVP (Minimum Viable Product), prototipi o app con team piccoli;
- Progetti interni, CRUD semplici, gestionali;
- Quando serve velocità di sviluppo e facilità di deploy;
- Se il dominio non è ancora ben definito (adottando l'architettura monolitica, non è necessario prevedere e implementare forme di astrazione per accettare nuovi cambiamenti).

Lo starter kit può risultare poco efficace nei casi in cui il progetto prevede:

- Team grandi e distribuiti;
- Scalabilità indipendente per moduli diversi;
- Domini complessi o con responsabilità ben separate (es. e-commerce, piattaforme SaaS multi-tenant).

Lo starter kit SOUL prevede l'utlizzo di tre container:

* **NextJS** - Applicazione web per l'interazione con l'utente finale;
* **PostgreSQL** - Database relazionale per la persistenza dei dati;
* **Cronjob** - Sistema batch per il download programmato dei dati da fonti esterne (opzionale).

![Componenti pila software SOUL](/diagrammi/componenti-architettura.svg)



## NextJS

Gli applicativi web sviluppati con il framework NextJS versione 14 (App Router), permettono di costruire sistemi con architettura di tipo client-server e multi tier. 

Il framework eredita le funzionalità di React, pertanto è possibile utilizzare i componenti React lato client e lato server. Gli applicativi web sviluppati con questa tecnologia sono di tipo client-server e prevedono che l'utente interagisca con l'applicazione dispiegata nel server Node/NextJS attraverso il browser.

### Architettura standard a due livelli

L'archittettura standard di un'applicazione SOUL è organizzata a **due livelli**:

* Nel **primo livello** sono gestite: le richieste che l'utente invia al server; la presentazione del codice HTML restituito dal sever; le operazioni che modificano l'interfaccia utente lato client (browser);
* Nel **secondo livello** sono gestite: l'esecuzione del codice Javascript lato server; la logica di presentazione e la logica di business; la comunicazione con il database attraverso un ORM.

![Design pattern architetturale standard](/diagrammi/two-tier.svg)

### Architettura a tre livelli

Al fine di perseguire qualità del prodotto finale quali affidabilità, efficienza delle prestazioni, manutenibilità e sicurezza, il sistema prevede l'utilizzo di un container che ha il compito di sincronizzare i dati condivisi dall'Ateneo nella banca dati "locale". Il sistema può tuttavia essere modificato per consumare sorgenti dati esterne in modalità "quasi tempo reale" (semi real-time). A seguire riportiamo alcuni scenari in cui questo tipo di strategia risulta vantaggiosa:

* L'applicazione non può sincronizzare localmente i dati del sistema "terzo" a causa della loro **dimensione/volume**;
* L'applicazione richiede il **dato "fresco"**, ovvero l'informazione appena aggiornata;
* Il sistema esterno **non permette la copia "locale"** dei dati.

In questi casi è necessario prevedere un'architettura a tre livelli. Il terzo livello ha il compito di fornire accesso alle risorse esterne via API o preferibilemente via API Gateway.

![Design pattern architetturale a tre livelli](/diagrammi/three-tier.svg)





