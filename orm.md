---
title: Dati dell'applicazione
description: Modellazione dell'ER, accesso ai dati via ORM e versionamento della banca dati
published: true
date: 2025-05-06T10:14:03.822Z
tags: 
editor: markdown
dateCreated: 2025-05-06T10:14:03.822Z
---

# Dati dell'applicazione
Le applicazioni sviluppate a partire dallo [Starter Kit](https://github.com/Piattaforme-Applicativi/soul-starter-kit) devono gestire i dati dell'utente. Per i casi in cui è necessario dare garanzie di **A**tomicità, **C**oerenza, **I**solamento e **D**urabilità è possibile utilizzare l'RDBMS PostgreSQL. Adottando PostgreSQL è necessario prevedere una fase iniziale di modellazione della base dati. Lo [Starter Kit](https://github.com/Piattaforme-Applicativi/soul-starter-kit) offre un meccanismo di versionamento della banca dati, che dà allo sviluppatore la possibilità di perfezionare il modello per ampliare del dominio durante i successivi rilasci del prodotto software. Prisma ORM è la libreria Typescript adottata per comunicare con la banca dati e gestirne il versionamento. 

## Flusso di lavoro

A fronte della fase di analisi il team di sviluppo è in possesso di suffcienti informazioni per disegnare il modello Entità Relazione che descrive il dominio dell'applicazione. Dopo aver descritto il modello concettuale, il team di sviluppo traduce il modello astratto in un modello reale nella base dati RDBMS PostgresSQL. Per aggiornare l'istanza del database, il team di sviluppo genera i file con estensione .sql da eseguire in sequenza per migrare ad una versione successiva dello schema PostgresSQL. All'aggiornamento dell'istanza PostgreSQL segue l'aggiornamento del file descrittore di Prisma ORM.  A seguito dell'aggiornamento dello schema prisma, è utile generare nuovamente il client prisma nell'ambiente di sviluppo per propagare le modifiche in NextJS.

![](diagrammi/er.svg)

### Compiti dello sviluppatore

Segue l'elenco delle attività che derivano dalla raccolta di nuovi requisiti funzionali. In tutti i casi nei quali i la base dati del dominio cambia, lo sviluppatore deve eseguire le seguenti operazioni nell'ambiente di sviluppo.

| Codice | Nome del compito                           | Descrizione del compito                                      |
| :----: | ------------------------------------------ | ------------------------------------------------------------ |
|   S1   | Analisi del dominio e modellazione dell'ER | Lo sviluppatore formalizza il modello ER che descrive il dominio |
|   S2   | Aggiornamento istanza RDBMS                | Lo sviluppatore genera uno script .sql vuoto *nextjs/prisma/migrations/{timestamp}-{nuova-parte-db}*. All'interno dello script vuoto lo sviluppatore trascrive gli statement DDL e SQL per aggiornare l'istanza del database, con le modifiche nella nuova versione dell'ER al passo precedente. |
|   S3   | Aggiornamento schema Prisma ORM            | Lo sviluppatore aggiorna l'istanza del database per eseguendo lo script .sql creato in precedenza. A seguito dell'aggiornamento dell'istanza lo sviluppatore aggiorna lo schema Prisma ORM in *nextjs/prisma/schema.prisma* adeguando i nomi dei campi da Snake case a Camel case per rispettare linee guida per la scrittura del codice sorgente |
|   S4   | Generazione prisma client                  | Lo sviluppatore genera il client/driver prisma a partire dallo schema. Il file schema.prisma è un metadescrittore va pertanto generata la versione  in formato .js eseguibile dall'interprete javascript. |

Shell dei comandi da eseguire nel microservizio NextJS:

```bash
# Formalizzato il nuovo ER
# Accedo all'istanza nextjs
docker exec -it todo-nextjs sh

# S2 - Aggiornamento istanza RDBMS 
npx prisma migrate dev --create-only --name nuova-parte-db
# created: nextjs/prisma/migrations/{timestamp}-{nuova-parte-db}.sql
# Edit {timestamp}-{nuova-parte-db}.sql: Aggiungo nel file gli statement DDL e SQL per aggiornare l'istanza del database
npx prisma migrate deploy

# S3 - Aggiornamento schema Prisma ORM
npx prisma db pull
# Edit nextjs/prisma/schema.prisma: modifica manuale allo schema 

# S4 - Generazione prisma client
npx prisma generate
```