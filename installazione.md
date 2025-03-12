---
title: Installare e configurare l'ambiente
description: Installare e configurare l'ambiente con l'aiuto dello Starter Kit
published: true
date: 2025-03-12T15:42:01.189Z
tags: 
editor: markdown
dateCreated: 2025-03-12T14:40:30.089Z
---

# Installazione dell'ambiente

L'ambiente di sviluppo SOUL è stato progettato con l'intenzione di raggiungere i seguenti obiettivi:

* Semplificare le attività di configurazione dell'ambiente di lavoro dello sviluppatore;
* Rendere lo sviluppatore autonomo fin da subito, posticipando le attività di integrazione al momento del dispiegamento negli ambienti di staging e produzione (per esempio, la richiesta di integrazione con il sistema di Single Sign-On può essere fatta in un secondo momento);
* Migliorare la compatibilità e la portabilità negli ambienti di staging e produzione.

Per raggiungere questi obiettivi il framework è stato progettato a micro-servizi utilizzando il sistema di virtualizzazione *open-source* Docker.

La comunità SOUL mette a disposizione uno [Starter Kit](https://github.com/Piattaforme-Applicativi/soul-starter-kit) per iniziare a sviluppare con queste tecnologie. 

## Requisiti di sistema

* 16 GB RAM (consigliato)
* 4 GB di spazio libero su disco (circa 800 MB per l'applicazione)
* Sistema operativo con supporto per Docker v27 (MacOS, GNU/Linux, Windows)

## Requisiti software

Di seguito riportiamo gli strumenti per sviluppare con il framework SOUL:

- Terminale con shell bash, zsh oppure sh
- Docker: https://www.docker.com/community-edition#/download
- Client GIT: https://git-scm.com/downloads
- GitHub Desktop Client: https://docs.github.com/en/desktop/installing-and-authenticating-to-github-desktop/installing-github-desktop (opzionale)

## Download dello Starter Kit 

Dopo aver installato l'ambiente [Docker](https://www.docker.com/get-started/) è sufficiente clonare lo [Starter Kit](https://github.com/Piattaforme-Applicativi/soul-starter-kit) utilizzando un client GIT come [GitHub Desktop](https://docs.github.com/en/desktop/installing-and-authenticating-to-github-desktop/installing-github-desktop).

Se invece vuoi clonare il repository con un client GIT a linea di comando devi prima creare un [token di accesso personale (classic) per GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#personal-access-tokens-classic).  Per creare il token puoi seguire questi passi:

1. Accedi a GitHub e vai su **Impostazioni**.
2. Vai su **Impostazioni sviluppatore** > **Token di accesso personale** > **Token (classico)**.
3. Clicca su **Genera nuovo token (classico)**.
4. Seleziona la scadenza del token (scegli **"Nessuna scadenza"** solo se necessario).
5. Sotto ambiti (scopes), seleziona almeno:
   - **repo** -> per accedere ai repository privati e pubblici.
   - **workflow** -> se devi interagire con GitHub Actions.
6. Clicca **Genera token**.
7. Copia il token (GitHub non lo mostrerà più dopo la creazione).

Una volta che hai memorizzato il token puoi procedere con il comando di clone del repository

```bash
# Username: tuo utente GitHub
# Password: il token che hai memorizzato
git clone https://github.com/Piattaforme-Applicativi/soul-starter-kit.git $myapp
```

## Personalizzazione dell'ambiente

Lo starter kit ti permette di iniziare da subito a sviluppare l'applicativo. Tuttavia prima di cominciare lo sviluppo di un nuovo applicativo, è buona norma modificare i file di avvio dell'ambiente per meglio identificare e differenziare la tua applicazione. 

Per fare questo è sufficiente modificare i file(s):

* docker-compose.yml
* docker-coolify.yml
* nextjs/package.json
* nextjs/package-lock.json

modificando la stringa "todo" che identifica il nome dell'applicazione presente nello starter kit, con il nome della tua nuova applicazione (eg. $myapp).

Per comodità riportiamo i comandi da eseguire a terminale nelle piattaforme GNU/Linux e MacOS.

```bash
cd $myapp

# Principale file dell'ambiente
sed s/todo/$myapp/g docker-compose.yml > docker-compose.yml.new
sed s/todo/$myapp/g docker-coolify.yml > docker-coolify.yml.new
rm docker-compose.yml docker-coolify.yml
mv docker-compose.yml.new docker-compose.yml
mv docker-coolify.yml.new docker-coolify.yml

# Ambiente NextJS
sed s/todo/$myapp/g nextjs/package.json > nextjs/package.json.new
sed s/todo/$myapp/g nextjs/package-lock.json > nextjs/package-lock.json.new
rm nextjs/package.json nextjs/package-lock.json
mv nextjs/package.json.new nextjs/package.json
mv nextjs/package-lock.json.new nextjs/package-lock.json

```

# Avvio e configurazione dell'ambiente di sviluppo

Una volta personalizzato l'ambiente di sviluppo puoi avviare la nuova applicazione. L'inizializzazione e la configurazione dell'ambiente prevede le fasi a seguire.

## Primo avvio 

La fase iniziale di avvio prevede: 

- il download dei *container* Docker e dei pacchetti node; 
- il build dei *container* per i quali è stato specificato un Dockerfile;
- la creazione dell'utente e dello schema (vuoto) PostgresSQL.

Tutte queste operazioni vengono eseguite in automatico utilizzando il comando 

```bash
cd $myapp
docker compose up
```

Il primo avvio si può considerare concluso quando nel terminale compare questa sequenza di istruzioni

```bash
npm WARN deprecated glob@7.2.3: Glob versions prior to v9 are no longer supported
myapp-nextjs    |
added 617 packages, and audited 618 packages in 14s
myapp-nextjs    |
163 packages are looking for funding
  run `npm fund` for details
myapp-nextjs    |
9 vulnerabilities (6 moderate, 3 high)
myapp-nextjs    |
To address issues that do not require attention, run:
  npm audit fix
myapp-nextjs    |
To address all issues (including breaking changes), run:
  npm audit fix --force
myapp-nextjs    |
Run `npm audit` for details.
myapp-nextjs    |
 ✓ Ready in 15.7s
```

## Configurazione iniziale

Avviato l'ambiente l'applicativo è raggiungibile all'indirizzo http://localhost. Accedendo a questo indirizzo la prima volta, verrà presentata la maschera di configurazione dell'ambiente NextJS (questo quando manca il file *.env* all'interno della sottocartella nextjs). 

La pagina di configurazione del'ambiente NextJS è pensata per guidare lo sviluppatore nella configurazione iniziale dell'ambiente. Se hai seguito questa guida, il paramento **Descrittore della fonte dati** deve essere modificato in *postgresql://\$myapp:\$myapp@postgres/\$myapp*.

Una volta salvata la configurazione l'ambiente deve essere riavviato per caricare le variabili d'ambiente

```bash
cd $myapp
docker compose down # oppure ctrl-C
docker compose up
```

## Inizializzazione della banca dati

Il framework prevede il versionamento della banca dati. Nella documentazione del framework è riportata un guida di indicazioni sul ciclo di vita di un applicativo basato su SOUL, una parte della documentazione fà riferimento anche al versionamento della banca dati.

Una volta effettuata la configurazione iniziale il sistema è pronto per il versionamento della banca dati. La variabile d'ambiente DATABASE_URL presente nel file *nextjs/.env* viene utilizzata dal *container* prisma per inizializzare la banca dati dell'applicativo come riportato [nella guida dell'ORM Prisma](https://www.prisma.io/docs/orm/prisma-client/deployment/deploy-database-changes-with-prisma-migrate).

Per inizializzare lo schema della banca dati in PostgresSQL è sufficiente avviare nuovamente l'ambiente.

```bash
cd $myapp
docker compose up
```

Da questo momento in poi è possibile sviluppare nell'ambiente di lavoro SOUL.