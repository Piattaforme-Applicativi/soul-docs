---
title: Prima app CRUD
description: Creare una prima applicazione CRUD
published: true
date: 2025-05-09T14:32:36.129Z
tags: 
editor: markdown
dateCreated: 2025-05-08T09:59:51.833Z
---

# Creare un'applicazione CRUD
E' utile illustrare il processo di implementazione di un'applicazione di tipo CRUD per evidenziare alcuni aspetti importanti durante l'implementazione come:

* L'organizzazione dei file e delle cartelle per un nuovo modulo;
* La localizzazione linguistica dei messaggi, delle etichette e dei nomi dei campi;
* Il caching delle componenti;
* Il render client-side e server-side delle componenti;
* L'accesso alla base dati con l'ORM.

## Workflow implementazione di un nuovo modulo

Il modo più efficiente per avviare lo sviluppo di un nuovo modulo software in SOUL consiste nell'implementazione iniziale delle interfacce di classe e dei tipi dato trattati dal nuovo modulo. La creazione della pagina di elenco degli elementi, consente di definire la struttura di base del modulo e di facilitare l'integrazione con le funzionalità successive.

Particolare attenzione deve essere riservata alla progettazione del form di inserimento, che rappresenta il punto in cui si concentra la maggior parte del tempo di sviluppo. È infatti essenziale curare l’esperienza utente, garantendo un’interazione fluida e un buon livello di usabilità.

Una volta completata la logica di inserimento, si può procedere con la realizzazione della pagina di dettaglio dell’elemento, che solitamente eredita parte delle strutture già definite.

Per quanto riguarda l’eliminazione di un elemento, è fondamentale che sia sempre accompagnata da un messaggio di conferma, al fine di evitare cancellazioni accidentali.

Infine, la form di modifica viene di norma sviluppata partendo da una copia di quella di inserimento, adattandola alle specifiche esigenze di aggiornamento del dato.

Per accedere ad ogni interfaccia utente è necessario realizzare un nuova rotta secondo quanto definito dalle [linee guida per la crezione di una pagina NextJS](https://nextjs.org/docs/app/getting-started/layouts-and-pages). La progettazione e il disegno di ogni interfaccia utente, è accompagnata dalla creazione di nuove [server actions NextJS che recuparano i dati dal database](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations).

![Nuovo modulo di un applicativo SOUL](diagrammi/crud.svg)

Un nuovo modulo CRUD prevede la creazione di nuovi componenti e rotte NextJS, con forma e struttura presentate nelle [linee guida per la scrittura del codice](/stile-codice).

# Esempio di implementazione di un nuovo modulo

L'esempio di creazione del nuovo modulo CRUD viene fatta a partire dalla modellazione dei dati già presentata alla sezione [esempio di modellazione della base dati](/orm#esempio-di-modellazione). Il nuovo modulo dell'applicativo tratterà una domanda di prenotazione di un mezzo. La nuova domanda di prenotazione di risorsa presentata da un utente conterrà: l' indirizzo email dell'utente; la tipologia di risorsa che deve essere prenotata  (auto, bicicletta, monopattino), il tempo di utilizzo della risorsa.

## Tipi e interfacce di classe

Le interfacce di classe e i tipi dato sono necessari a migliorare la leggibilità del codice. Inoltre l'eseprienza di sviluppo è migliorata per gli IDE di nuova generazione dotati di analizzatori di codice statico e di meccanismi di autosuggerimento.

```typescript
// nextjs/types/request.ts

import { resourceType } from "@prisma/client";

export default interface Request {
  id: number;
  createdBy: String;
  resourceType: resourceType;
  bookingDate: Date;
  bookingSlot?:  String;
  createdAt: Date;
  updatedAt: Date;
}
```

## Lista degli elementi

La nuova pagina che presenta l'interfaccia web all'utente che elenca le richieste, sarà raggiungibile all'indirizzo */request/*. Per creare la nuova rotta va creato un nuovo **React Server Component** nella posizione *nextjs/app/secure/request/page.tsx*

```jsx
// nextjs/app/secure/request/page.tsx
...
import { permissionType } from "@prisma/client";
import UserNotAuthorized from "@/components/common/user-not-authorized";
import ListRequests from "@/components/request/list";

export default async function Page() {
  const user = await getSessionPayload();
	...
  return (
    {!isAuthorized(user, permissionType.requestManage) ? (
        <UserNotAuthorized />
      ) : (
        <ListRequests/ >
      )}
  );
}
```

Deve essere creata poi la server action che recupera le requests dal database. 

```jsx
// nextjs/components/request/actions.ts

import Request from "@/types/request";

export async function listRequests(): Promise<Request[]> {
  try {
    return (
      await prisma.request.findMany({
        orderBy: {
          createdAt: "asc",
        }
      })
    ).map((r) => r as Request);
  } catch (e) {
    console.error(`Failed to find the request: ${e}`);
    return [];
  }
}
```

