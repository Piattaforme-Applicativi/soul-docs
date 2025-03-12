---
title: Stile del codice
description: Linee guida per la scrittura del codice sorgente degli applicativi
published: true
date: 2025-03-12T17:56:48.046Z
tags: 
editor: markdown
dateCreated: 2025-03-12T17:56:48.046Z
---

# Linee guida per la scrittura del codice sorgente

I file e le directory dell'applicazione sviluppata con NextJS versione 14, sono organizzate in modo da ospitare i tre strati dell'architettura three-tier:

* **Navigazione**: la cartella *nextjs/app/* contiene gli elementi struttura di interfaccia utente per la navigazione. Gli elementi a questo livello sono gli unici a poter essere elencati in una sitemap (§UNAV, §DNAV);
* **Componenti React (Client e Server)** (riusabili): la cartella *nextjs/components/* contiene lo strato di interfaccia per l'iterazione con l'utente finale;
* **Interfacce di classe**: la cartella  *nextjs/types/* contiene le interfacce di classe per le classi e gli oggetti condivisi da API REST e componenti React;
* **Object relation mapper**:  la cartella  *nextjs/prisma/* contiene le definizioni per gli schemi (SQL o NoSQL) e gli script per il versionamento della base dati.

Il codice sorgente del sistema è scritto in **lingua inglese**. E' pertanto necessario un meccanismo di localizzazione linguistica (i18n) per la presentazione delle informazioni nelle interfacce utente.

Si adottano le convezioni a seguire per lo stile di nomi dei file e delle cartelle nel sorgente del microservizio NextJS. 

| Elemento                       | Stile      | Estensione | Esempio                                                      |
| ------------------------------ | ---------- | ---------- | ------------------------------------------------------------ |
| Cartella                       | Kebab case | N/A        | Anagrafica studenti **diventa** students/                    |
| Componente (React client)      | Kebab case | .tsx       | Elenco nuove matricole **diventa** new-students.tsx          |
| Classe e Interfaccia di classe | Kebab case | .ts        | Studente triennale **diventa** bachelor-student.ts           |
| Libreria                       | Kebab case | .ts        | Utilità api **diventa** network-utils.ts                     |
| Script di migrazione           | Snake case | .sql       | Schema per gli studenti **diventa** 20240327000607_init/migrate.sql |

```
nextjs
    +- app/
        +- student
            +- page.tsx // (GET /student/index.html - elenco degli studenti)     
    +- components/
        +- student/
                +- actions.ts // (export async function listsStudents())
                +- list.tsx
                +- ....
    +- prisma/
        +- migrations/
                +- ${datetime}_${features}
                        +- migration.sql
        +- schema.prisma
    +- types/
        +- student.ts (interface IStudent, class Student)
```

## Convenzioni per la scrittura del codice

Lo stile di programmazione si adegua alle convezioni a seguire

### DDL e SQL

I nomi delle tabelle, dei campi, delle viste, dei trigger, delle chiavi e degli indici devono essere in **snake_case**

```sql
-- migrate.sql
CREATE TABLE "student" (
    "id" SERIAL NOT NULL,
    "first_name" CHARACTER VARYING(50) NOT NULL,
    "surname" CHARACTER VARYING(50) NOT NULL,
    "codice_fiscale" CHARACTER VARYING(16) NOT NULL,
    CONSTRAINT "student_pkey" PRIMARY KEY ("id")
);
```

### Schema prisma

I nomi dei model e degli attributi devono essere in **camelCase**

```
// Prisma
model student {
    id Int @id @unique @default(autoincrement())
    firstName @map("first_name") String
    surname @map("surname") String
    codiceFiscale @map("codice_fiscale") String
}
```

### Classi e interfacce typescript

I nomi delle classi e delle interfacce typescript deve essere in **PascalCase**. I nomi degli attributi, dei metodi e delle variabili deve essere in **camelCase**.

```typescript
export default interface IStudent {
  id: number;
  firstName: string;
  surname: string;
  codiceFiscale: string;
}
```

### Componenti React (Client)

Si preferisce adottare lo  stile delle componenti React funzionale alle componenti a classe. Per le componenti riusabili è necessario utilizzare i descrittori di tipo per decrivere la segnatura delle funzioni/componenti. I nomi dei component react deve essere in **PascalCase**. I nomi degli attributi, dei metodi  e delle variabili deve essere in **camelCase**. 

```react
type NewProps = {
  notify: () => void;
};

export default function NewStudent({ notify }: NewProps) {  
  return (
    <>...</>
  );
};

// <NewStudent notify={() => console.log('New student ...')}>
```

### Backend for frontend

La versione 14 di NextJS ha consolidato l'utilizzo delle [server actions](https://nextjs.org/docs/14/app/building-your-application/data-fetching/server-actions-and-mutations). La convenzione per la scrittura del codice sorgente del backend del sistema è di utilizzare le server-actions. Cercheremo di illustrare brevemente i vantagggi di questo tipo di approccio. Le server-actions in ambiente NextJS possono svolgere una doppia funzione:

* API REST (Like) - semplificano la scrittura del codice  per l'interazione con il server nelle componenti React di tipo client;
* Funzioni di recupero dati - le funzioni che eseguono data fetch possono essere riutilizzate nelle componenti React di tipo server per recuperare i dati in modo efficiente (in modo simultaneo - [async concurrency](https://en.wikipedia.org/wiki/Concurrency_(computer_science)))

Disegno che spiega come programmare in modo semi monolitico in NextJS

Parlare anche di:

* esecuzione asincrona e parallela server side NextJS
* problema del primo caricamento in React



### CSS

I nomi delle classi e degli id deve essere in **kebab-case**

```css
#student-list { background-color: green }
.student-avatar { border: 1px solid black }
```

# Librerie da utilizzare

Segue l'elenco delle componenti che tipicamente compongono il sistema e le librerie che è conveniente impiegare:

- Obiect Relation Mapper - [Prisma.io](https://www.prisma.io/)
- Design delle interfacce utente - [AGID React Dev Kit](https://italia.github.io/design-react-kit/?path=/story/documentazione-welcome--page)

# Glossario

| Termine o acronimo | Descrizione                                                  |
| ------------------ | ------------------------------------------------------------ |
| Camel case         | La notazione a cammello prevede di scrivere in minuscolo e senza spazi i "nomi": delle variabili; degli attributi degli oggetti; dei metodi; delle funzioni. Ogni parola che compone il "nome" deve riportare l'iniziale maiuscola. L'iniziale del nome così composto deve essere in minuscolo (eg. camelCasel) |
| Pascal case        | La notazione di Pascal è equivalente alla notazione a cammello con l'eccezzione che l'iniziale del nome è in maiuscolo (eg. PascalCase) |
| Snake case         | La notazione a serpente prevede di scrivere, in minuscolo e rimpiazzando gli spazi con il carattere _ , i "nomi": delle variabili; degli attributi degli oggetti; dei metodi; delle funzioni (eg. snake_case) |
| Kebab case         | La notazione a kebab prevede di scrivere, in minuscolo e rimpiazzando gli spazi con il carattere _ , i "nomi": delle variabili; degli attributi degli oggetti; dei metodi; delle funzioni (eg. snake-case) |
| Three-tier         | Con l'espressione architettura three-tier ("a tre strati") è indicata una particolare architettura software per l'esecuzione di un'applicazione web che prevede la suddivisione dell'applicazione in tre diversi moduli o strati dedicati rispettivamente all' interfaccia utente, alla logica funzionale e alla gestione dei dati persistenti. |
| Sitemap            | Una sitemap, o site map, o semplicemente mappa, è una pagina Web che elenca gerarchicamente tutte le pagine di un sito web. Nata per facilitare la navigazione dell'utente all'interno del sito, ha poi avuto una notevole importanza nell'attività di scansione della Rete da parte dei crawler dei motori di ricerca. |
| MVC                | Model-view-controller in informatica, è un pattern architetturale molto diffuso nello sviluppo di sistemi software, in particolare nell'ambito della programmazione orientata agli oggetti e in applicazioni web, in grado di separare la logica di presentazione dei dati dalla logica di business |
| i18n               | L'internazionalizzazione o la localizzazione linguistica è la capacità del software di adattarsi alla lingua dell'utente con il quale stà interagendo |

# Riferimenti

- AP - [Everything You NEED to Know About Client Architecture Patterns](https://youtu.be/I5c7fBgvkNY?si=k4i3azJwOtHMDySZ), ultima visita 15/3/2024
- API - [Api Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers), ultima visita 15/3/2024
- UNAV - [User routing](https://nextjs.org/docs/app/building-your-application/routing)](https://nextjs.org/docs/app/building-your-application/routing/route-handlers), ultima visita 15/3/2024
- DNAV - [User navigation dynamic routing](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes)](https://nextjs.org/docs/app/building-your-application/routing/route-handlers), ultima visita 15/3/2024
- RF - [React Function Components](https://www.robinwieruch.de/react-function-component/), ultima visita 15/3/2024
- ORM - [Prisma ORM](https://www.prisma.io/)](https://www.robinwieruch.de/react-function-component/), ultima visita 15/3/2024

