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

## Glossario

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

## Riferimenti

- AP - [Everything You NEED to Know About Client Architecture Patterns](https://youtu.be/I5c7fBgvkNY?si=k4i3azJwOtHMDySZ), ultima visita 15/3/2024
- API - [Api Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers), ultima visita 15/3/2024
- AREST - [Raccomandazioni AGID per le API Rest,](https://www.agid.gov.it/sites/default/files/repository_files/04_raccomandazioni_di_implementazione.pdf](https://nextjs.org/docs/app/building-your-application/routing/route-handlers), ultima visita 15/3/2024
- UNAV - [User routing](https://nextjs.org/docs/app/building-your-application/routing)](https://nextjs.org/docs/app/building-your-application/routing/route-handlers), ultima visita 15/3/2024
- DNAV - [User navigation dynamic routing](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes)](https://nextjs.org/docs/app/building-your-application/routing/route-handlers), ultima visita 15/3/2024
- RF - [React Function Components](https://www.robinwieruch.de/react-function-component/), ultima visita 15/3/2024
- ORM - [Prisma ORM](https://www.prisma.io/)](https://www.robinwieruch.de/react-function-component/), ultima visita 15/3/2024

# Stile del codice sorgente

I file e le directory dell'applicazione sviluppata con NextJS versione 14, sono organizzate in modo da ospitare i tre strati dell'architettura three-tier:

* **Navigazione**: la cartella *nextjs/app/* contiene gli elementi struttura di interfaccia utente per la navigazione. Gli elementi a questo livello sono gli unici a poter essere elencati in una sitemap (§UNAV, §DNAV);
* **REST API**: la cartella *nextjs/app/api/* contiene la logica funzionale e la gestione dei dati in MVC (§API);
* **Componenti React** (riusabili): la cartella *nextjs/components/* contiene lo strato di interfaccia per l'iterazione con l'utente finale;
* **Interfacce di classe**: la cartella  *nextjs/types/* contiene le interfacce di classe per le classi e gli oggetti condivisi da API REST e componenti React;
* **Object relation mapper**:  la cartella  *nextjs/prisma/* contiene le definizioni per gli schemi (SQL o NoSQL) e gli script per il versionamento della base dati.

Il codice sorgente del sistema è scritto in **lingua inglese**. E' pertanto necessario un meccanismo di localizzazione linguistica (i18n) per la presentazione delle informazioni nelle interfacce utente.

Si adottano le convezioni a seguire per lo stile di nomi dei file e delle cartelle nel sorgente del microservizio NextJS. 

| Elemento                       | Stile      | Estensione | Esempio                                                      |
| ------------------------------ | ---------- | ---------- | ------------------------------------------------------------ |
| Cartella                       | Kebab case | N/A        | Anagrafica studenti **diventa** students/                    |
| Componente                     | Kebab case | .tsx       | Elenco nuove matricole **diventa** new-students.tsx          |
| Classe e Interfaccia di classe | Kebab case | .ts        | Studente triennale **diventa** bachelor-student.ts           |
| Libreria                       | Kebab case | .ts        | Utilità api **diventa** network-utils.ts                     |
| Script di migrazione           | Snake case | .sql       | Schema per gli studenti **diventa** 20240327000607_init/migrate.sql |

```
nextjs
    +- app/
        +- api/
            +- [id]
                +- route.ts // (DELETE, POST - Update, GET- Detail)
            +- route.ts // (GET - Query, POST - Insert)
        +- student
            +- page.tsx // (GET /student/index.html - elenco degli studenti)     
    +- components/
        +- student/
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

### Componenti React

Si preferisce adottare lo  stile delle componenti React funzionale alle componenti a classe. Per le componenti riusabili è necessario utilizzare i descrittori di tipo per decrivere la segnatura delle funzioni/componenti. I nomi dei component react deve essere in **PascalCase**. I nomi degli attributi, dei metodi  e delle variabili deve essere in **camelCase**. 

```react
type NewProps = {
  notify: () => void;
};

export const NewStudent: React.FC<NewProps> = ({ notify }: NewProps) => {  
  return (
    <>...</>
  );
};

// <NewStudent notify={() => console.log('New student ...')}>
```

### CSS

I nomi delle classi e degli id deve essere in **kebab-case**

```css
#student-list { background-color: green }
.student-avatar { border: 1px solid black }
```

### API REST

Le API di stile REST devono essere conformi alle linee guida AGID (§AREST). I nomi degli attributi devono essere in **camelCase** e gli attributi devono essere in inglese, quando sono localizzabili (eg. codice fiscale non è localizzabile). 

```typescript
export async function GET() {
  return NextResponse.json({
    "items": await prisma.Student.findMany({
        orderBy: [{ id: "asc" }],
      })  
    });
}
```

# Librerie da utilizzare

Segue l'elenco delle componenti che tipicamente compongono il sistema e le librerie che è conveniente impiegare:

- Obiect Relation Mapper - [Prisma.io](https://www.prisma.io/)
- Design delle interfacce utente - [AGID React Dev Kit](https://italia.github.io/design-react-kit/?path=/story/documentazione-welcome--page)

