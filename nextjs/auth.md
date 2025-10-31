---
title: Autenticazione e autorizzazione
description: Autenticazione con Identity Provider di Ateneo
published: true
date: 2025-09-22T07:29:27.676Z
tags: 
editor: markdown
dateCreated: 2025-06-30T07:20:00.495Z
---

# Autenticazione e autorizzazione

Il sistema di autenticazione di **SOUL** è progettato in conformità con le linee guida ufficiali fornite da **NextJS**, seguendo le best practice presenti nella documentazione alla voce [`Guide > Autentication`](https://nextjs.org/docs/pages/guides/authentication). In particolare, per garantire la sicurezza e l'interoperabilità dei sistemi, l'infrastruttura prevede l'integrazione con il **Single Sign-On (SSO) di Ateneo**, quale meccanismo centrale per la gestione delle identità e in parte delle sessioni utente. L'integrazione viene attivata al momento del rilascio delle nuove applicazioni in ambienti differenti da quello di sviluppo, assicurando così che tutte le nuove istanze dell'applicazione siano vincolate al sistema di autenticazione istituzionale. E' prevista una procedura formale di accreditamento del nuovo sistema presso il team che gestisce l'Identity Provider SAML (da ora IdP). La registrazione è necessaria per completare il processo di integrazione dei nuovi applicativi. Nel linguaggio SAML, le nuove applicazioni vengono chiamate  Service Provider (da ora SP). Per agevolare le attività di sviluppo e test, l'ambiente di sviluppo di SOUL mette a disposizione un **IdP di Mockup**, che consente di simulare il comportamento del sistema di autenticazione senza dover procedere immediatamente alla registrazione ufficiale nell'IdP. Questo approccio consente ai team di sviluppo di lavorare in modo autonomo e flessibile, garantendo al contempo la piena compatibilità e conformità in fase di rilascio.

## Identità dell'utente e gestione della sessione

Le applicazioni sviluppate con SOUL devono utilizzare l'IdP di Ateneo.  Il flusso di autenticazione e autorizzazione si  articola in più fasi, orchestrate tra applicazione lato client (Browser), applicazione lato server Next.js (SP) e server che contiene le identità degli utenti (IdP). Autenticazione e autorizzazione in SOUL si declinano rispetto alle [linee guida di autenticazione di NextJS](https://nextjs.org/docs/pages/guides/authentication#stateless-sessions) come segue:

* **Authentication**: la verifica dell'identità dell'utente è demandata all'IdP di Ateneo;
* **Session management**: è **stateless**. I dati della sessione e l'identità dell'utente sono salvati in un cookie. Mantenere la sessione stateless è conveniente in caso di dispiegamento negli ambienti cloud di Ateneo (ovvero con architettura PaaS e contenitori ephimeral);
* **Authorization**: l'autorizzazione è di tipo **secure**. In ogni pagina dell'applicazione vengono eseguiti dei controlli sulla base dei permessi memorizzati nella sessione utente. Il cookie di sessione contiene l'elenco dei permessi dell'utente autenticato. A partire da questi permessi, è possibile implementare controlli mirati nelle pagine che necessitano di autorizzazione (React Server Components) .

Per entrare nel dettaglio del **flusso di autenticazione**, quando un utente tenta di accedere ad una rotta protetta (es. `/secure/request`), viene reindirizzato all’IdP per effettuare il login. L’IdP verifica le credenziali dell’utente e restituisce un token SAML 2.0 all’applicazione. Questo token viene poi convertito in formato  JWT firmato. Il JWT aiuta a trasportare i dati di identità dell’utente in modo sicuro e compatto. Il JWT viene memorizzato in un cookie di sessione, con gli attributi `HttpOnly`, `Secure` e `SameSite` per mitigare rischi come XSS e CSRF.

A ogni richiesta verso una rotta protetta, il middleware lato server di Next.js intercetta la richiesta, estrae il JWT dal cookie e ne verifica la firma. Se la firma nel token è valida, il middleware consente l’accesso alla  risorsa. Quando la firma nel token è assente, scaduta o non valida, l’utente viene  reindirizzato ad una pagina di errore o al login. Lato client il coookie di sessione non può essere letto (`HttpOnly`) . Di conseguenza le informazioni utente possono essere recuperate unicamente lato server. Le informazioni conservate nel JWT consentono di personalizzare l'interfaccia utente utilizzando i permessi.

Il riconoscimento dell'utente è delegato all'IdP. L’uso di cookie di sessione è prerequisito per un’architettura stateless, scalabile e facilmente integrabile con altri servizi (sopratutto con infrastrutture basate su container di tipo effimero). L’intero  flusso è compatibile con lo standard SAML e può essere esteso per supportare OAuth2 o OpenID Connect. Questo modello è  particolarmente adatto per applicazioni moderne che richiedono  sicurezza, modularità e interoperabilità con sistemi di identità aziendali.

![Autenticazione con SAML e sessione con JWT](diagrammi/auth.svg)

### Tipi di identità restituite dall'IdP

Il processo di autenticazione SSO all’Università degli Studi di Padova si basa su un IdP conforme a SAML 2.0. A seguito dell'autenticazione l'IdP di Ateneo fornisce gli attributi dell'utente. Gli utenti possono essere interni  (studenti o dipendenti) oppure esterni (autenticati tramite: SPID; CIE; IDEM). 

Gli utenti interni si autenticano all’IdP locale e ricevono un insieme di attributi che includono: identificatori esterni e interni (`shib_extid`, `shib_id`); codice fiscale (`shib_codicefiscale`); nome e cognome (`shib_givenname`, `shib_sn`); l’indirizzo email (`shib_mail`). L'affiliazione all'Ateneo (`shib_edupersonscopedaffiliation`)  è opzionale e multivalore. L'affiliazione all'Ateneo può assumere uno o più valori dell'insieme `[ 'member@unipd.it', 'staff@unipd.it', 'alum@unipd.it' ]`.  

Gli attributi `shib_mail`e `shib_id` hanno come valore l'indirizzo email dell'utente autenticato. Per distinguere tra studenti e dipendenti, è necessario analizzare il  dominio dell’indirizzo email: gli studenti hanno email che terminano con `@studenti.unipd.it`, mentre i dipendenti usano `@unipd.it`. Questo permette di classificare correttamente l’utente e applicare le relative regole di accesso.

Un attributo utile a distinguere il tipo di di utente è `shib_authsource`, che ha valore `LOCAL` quando l'utente è registrato in Ateneo.  

Gli utenti esterni si autenticano tramite sistemi federati con l'Ateneo come: SPID; CIE; Idem. In questi casi, l’IdP del sistema federato trasmette il set di attributi dell'utente autenticato. 
Per fare un esempio, nel caso di federazione con SPID, l'IdP SPID include: il codice identificativo SPID (`spid_spidCode`); il codice fiscale (`spid_fiscalNumber`); un eventuale codice IVA (`spid_ivaCode`); l’indirizzo email (`spid_email`). L’attributo `shib_authsource` assume il valore `SPID`.  Quando l'utente è registrato sia SPID che in Ateneo, l'IdP di Ateneo **aggiunge** gli attributi personali di Unipd.

Dal punto di vista dello sviluppatore, è fondamentale implementare  controlli robusti sugli attributi ricevuti dall' IdP. In particolare, è necessario verifica il valore di `shib_authsource` per determinare la provenienza dell'utente e classificare l’utente come interno o esterno.  Questo approccio consente di gestire in modo sicuro e  coerente l’accesso ai servizi digitali dell’Ateneo, garantendo al  contempo flessibilità nell’integrazione con fonti di identità esterne.

Alcune regole per classificare le diverse tipologie di utente:

* Studenti che frequentano e personale: sono considerati affiliati all'Ateneo; hanno una casella di posta @unipd.it oppure @studenti.unipd.it;
* Quando gli **alumni** non sono più parte dell'organizzazione non hanno più diritto al **servizio di posta**, tuttavia conservano l'accesso con il loro account (nome.cognome@studenti.unipd.it);
* I collaboratori esterni hanno una casella mail che ha indirizzo nome.cognome@unipd.it ma non hanno affiliazione con l'organizzazione Ateneo.

![Tiplogie di identità di utenti restituite dal SSO](diagrammi/auth-identity-types.svg)

Segue il riferimento agli attributi che devono essere restituiti dall'IdP in linguaggio SAML per realizzare l'autenticazione. 

```xml
<!--
	Questo file è stato adattato a partire dal file attribute-map.xml.
	Per avere l'elenco completo degli attributi presenti nel file attribute-map.xml
 	e messi a disposizione dall'IdP, va fatta richiesta al team che gestisce l'IdP.
-->

<Attributes xmlns="urn:mace:shibboleth:2.0:attribute-map" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <!-- c'è sempre per utenze interne e IDEM, non per SPID/CIE -->
    <Attribute name="urn:oid:2.5.4.42" id="shib_givenname"/> <!-- SOUL: firstName -->
		<!-- .... -->
    <Attribute name="urn:oid:2.5.4.4" id="shib_sn"/> <!-- SOUL: familyName -->
    <!-- .... -->
    <!-- c'è sempre per utenze interne e IDEM, non per SPID/CIE -->
    <Attribute name="urn:oid:0.9.2342.19200300.100.1.3" id="shib_mail"/> <!-- SOUL: mail, SOUL: id -->
  	<!-- .... -->
    <!-- descrive il rapporto (affiliazione) di una persona con un’istituzione, “scopato” da un dominio (cioè legato a un dominio dell’organizzazione). 
		Esempio di valore ritornato: 'member@unipd.it', 'staff@unipd.it', 'alum@unipd.it'

		La parte prima della @ (es. member, alum, staff) indica il tipo di affiliazione.
		La parte dopo la @ (es. unipd.it) indica il dominio dell’organizzazione.
		-->
    <Attribute name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" id="shib_edupersonscopedaffiliation"/> 
    <!-- .... -->
  	<Attribute name="https://www.cca.unipd.it/sso/attributes/codicefiscale" id="shib_codicefiscale"/> <!-- SOUL: codiceFiscale -->
  	<!-- .... -->
  	<Attribute name="www.cca.unipd.it/sso/attributes/externalid" id="shib_extid"/>   <!-- SOUL: externalId -->
  	<!-- .... -->
  
 </Attributes>
```

Segue un esempio di pseudo codice per riconoscere e classificare le diverse tipologie di utente dopo il Single Sign-On

```
INIZIO

  FUNZIONE estrai_dominio(email = NON DEFINITO, id = NON DEFINITO)

      // Se entrambi email e id non sono definiti, ritorna NON DEFINITO
      SE email E' NON DEFINITO E id E' NON DEFINITO ALLORA
          RITORNA NON DEFINITO
      FINE SE

      // Se email non definito, usa id
      DICHIARA mail = SE email E' NON DEFINITO ALLORA id ALTRIMENTI email

      // Trova la posizione del simbolo '@'
      DICHIARA posizione_arroba = TROVA_POSIZIONE(mail, "@")

      // Se non c'è '@', ritorna NON DEFINITO
      SE posizione_arroba = NON TROVATO ALLORA
          RITORNA NON DEFINITO
      FINE SE

      // Estrai la parte dopo '@'
      DICHIARA dominio = SOTTOSTRINGA(mail, posizione_arroba + 1, FINE)

      RITORNA dominio

  FINE FUNZIONE


  FUNZIONE tipo_utente(mail = NON DEFINITO, id = NON DEFINITO, tipo_utente = NON DEFINITO, sorgente = "LOCAL")

      // Se l'utente non compare tra gli utenti dell'Ateneo può essere una federazione SPID o CIE
      SE sorgente E' UGUALE a "CIE" O sorgente E' UGUALE a "SPID" ALLORA
          RITORNA CITIZEN
      FINE SE

      // Altro tipo di federazione per il momento non gestita
      SE sorgente E' DIVERSO da "LOCAL" ALLORA
          RITORNA NON DEFINITO
      FINE SE

      // Se non conosco il dominio di provenienza dell'utente non posso dire nulla
      DICHIARA dominio = estrai_dominio(mail, id)
      SE dominio E' NON DEFINITO ALLORA
          RITORNA NON DEFINITO
      FINE SE

      // Se il dominio è studenti.unipd.it
      SE dominio E' UGUALE A 'studenti.unipd.it' E 'alum@unipd.it' E' IN tipo_utente ALLORA
          SE mail E' NON DEFINITO ALLORA
              RITORNA ALUMNI
          ALTRIMENTI
              RITORNA STUDENT
          FINE SE

      // Se il dominio è unipd.it
      ALTRIMENTI SE dominio E' UGUALE A 'unipd.it' ALLORA
          SE tipo_utente E' NON DEFINITO ALLORA
              RITORNA EXTERNAL
          ALTRIMENTI SE 'staff@unipd.it' E' IN tipo_utente ALLORA
              RITORNA STAFF
          FINE SE
      FINE SE

      // Se nessuna condizione è soddisfatta
      RITORNA NON DEFINITO

  FINE FUNZIONE

FINE

```

## Workflow di integrazione

Per integrare correttamente il sistema è necessario creare un nuovo file `.env`, che contiene le configurazioni essenziali. Le principali chiavi da configurare sono:

- **SSO_PROVIDER**, ovvero l'IdP al quale deve essere registrato l'SP. La registrazione può avvenire per diversi ambienti (`local` = Mockup, `unipd_test` = Unipd di Test,  `unipd` = Unipd di Produzione );
- **SSO_SP_ENTITY_ID**, che rappresenta l'identificativo con cui registrare l'SP presso l'IdP. Per convenzione, questo valore corrisponde al `BASE_URL`;
- **BASE_URL**, ossia l'indirizzo completo (incluso protocollo e dominio) tramite il quale l'applicativo sarà raggiungibile dal browser (eg. `https://booking-requests.ict.unipd.it`);
- **SP_PRIVATE_KEY** e **SP_CERT_KEY**, che sono le chiavi necessarie per garantire una comunicazione sicura tra Browser, SP e IdP.

All'interno dello [Starter Kit](https://github.com/Piattaforme-Applicativi/soul-starter-kit) è disponibile un'interfaccia utente che semplifica la generazione del file di configurazione. L'interfaccia utente è raggiungibile nell'applicazione al path `/configuration/new`.

Dopo aver creato correttamente il file `.env`, si può procedere con il dispiegamento del sistema sia in ambiente di staging che in produzione. Una volta che il sistema è operativo e raggiungibile tramite l'indirizzo configurato in `BASE_URL`, è possibile scaricare il file di metadata che identifica l'SP accedendo al path `/saml/metadata`.

A questo punto, con il file di metadata disponibile, è possibile completare la registrazione del nuovo SP presso l'IdP di Ateneo. Prima di accedere alla [pagina di richiesta di accreditamento Single Sign On di Ateneo](https://registrazionisp.ict.unipd.it/form/richiestesso), è necessario ricevere un nuovo codice di invito (**Invitation Code**) scrivendo alla [coda ticket di Ateneo](https://helpdesk.ammcentr.unipd.it/otrs/customer.pl?OTRSCustomerInterface=oBbwZal2mXfTD3xE9GABt7nCH0MNaaPD).

![Worflow di integrazione per un nuovo Service Provider in staging o produzione](diagrammi/auth-integration-workflow.svg)

Un possibile messaggio per chiedere l'Invitation Code nella coda ticket _Single Sign On potrebbe essere:

> Buongiorno,
>
> stò completando l'attività di integrazione dell'ambiente staging per il nuovo
> sistema "xxx" con l'Identity Provider SAML di Ateneo.
>
> Potreste gentilmente comunicarmi l'Invitation Code per completare la domanda nel form "Richieste accreditamento Single Sign-On di Ateneo - UniPD"

### Compiti dello sviluppatore

Segue l'elenco delle attività che lo sviluppatore deve portare a termine per integrare i nuovi applicativi con l'IdP di Ateneo.

| Codice | Nome del compito                                             | Descrizione del compito                                      |
| :----: | ------------------------------------------------------------ | ------------------------------------------------------------ |
|   S1   | Dispiegamento applicativo in ambiente di staging e produzione | Lo sviluppatore crea un nuovo file  `.env`  per gli ambienti di produzione o di staging. Lo sviluppatore dispiega il nuovo applicativo negli ambienti di staging o di produzione. |
|   A1   | Nuovo ticket registrazione SP                                | Il personale ASIT richiede un **Invitation Code** per completare la richiesta di accreditamento Single Sign On di Ateneo nella **coda ticket _Single Sign On** dell'Ateneo. |
|   S3   | Registrazione nuovo SP                                       | Lo sviluppatore scarica dal nuovo applicativo dispiegato in staging o produzione il file `/saml/metadata` che identifica l' applicativo come SP.  Lo sviluppatore in possesso di **Invitation Code** e del file XML che identifica il nuovo applicativo / SP, invia una nuova  richiesta di accreditamento. |

#### S3 - Registrazione di un nuovo SP

Al momento dell'invio della [richiesta di accreditamento Single Sign On di Ateneo](https://registrazionisp.ict.unipd.it/form/richiestesso) è necessario porre attenzione a queste informazioni richieste nella form:

* **Invitation Code**: ottenuto al passo S2;
* **Metadata SP**: è il file XML che può essere scaricato dal path `/saml/metadata`  dell'istanza dell'applicazione;
* **Attributi richiesti**: sono gli attributi necessari a creare il JWT nel cookie di sessione. Gli attributi che devono essere obbligatoriamente richiesti sono: `shib_codicefiscale`,`shib_extid` , `shib_sn`, `shib_givenname`, `shib_mail`, `shib_edupersonscopedaffiliation`;
* **Note eventuali**: deve essere utilizzato nel caso in cui è necessario abilitare l'autenticazione con il sistema nazionale di identità digitale italiano.

# Esempio di riconoscimento del tipo di utente

Il riconoscimento della tipologia di utente è utile per assegnare permessi a classi di utenti. Segue la stesura dello pseudo codice di classificazione delle diverse tipologie di utenza. 

```typescript
import { userType } from "@prisma/client";
import { AuthUser } from "@/types/auth-user";

// ---------------------------
// Helper function
// ---------------------------

function extractDomain(email?: string, id?: string): string | undefined {
  const mail = email ?? id;
  if (!mail) return undefined;

  const atPos = mail.indexOf("@");
  if (atPos === -1) return undefined;

  return mail.substring(atPos + 1);
}

// ---------------------------
// Interfaccia Strategy
// ---------------------------

interface UserTypeStrategy {
  isApplicable(user: AuthUser): boolean;
  resolveUserType(): UserType | undefined;
}

// ---------------------------
// Strategie concrete
// ---------------------------


const strategies: UserTypeStrategy[] = [
  // Utente federato SPID o CIE → CITIZEN
  {
    isApplicable: user => user.authSource === "CIE" || user.authSource === "SPID",
    resolveUserType: () => UserType.citizen,
  },

  // Federazione non gestita → NON DEFINITO
  {
    isApplicable: user =>
      user.authSource !== "LOCAL" &&
      user.authSource !== "CIE" &&
      user.authSource !== "SPID",
    resolveUserType: () => undefined,
  },

  // Studente alumni (senza mail) → ALUMNI
  {
    isApplicable: user => {
      const domain = extractDomain(user.mail, user.id);
      return domain === "studenti.unipd.it" &&
             user.affiliation?.includes("alum@unipd.it") &&
             !user.mail;
    },
    resolveUserType: () => UserType.alumni,
  },

  // Studente regolare (con mail) → STUDENT
  {
    isApplicable: user => {
      const domain = extractDomain(user.mail, user.id);
      return user.mail && domain === "studenti.unipd.it" &&
             user.affiliation?.includes("alum@unipd.it");
    },
    resolveUserType: () => UserType.student,
  },

  // Membro dello staff → STAFF
  {
    isApplicable: user => {
      const domain = extractDomain(user.mail, user.id);
      return domain === "unipd.it" &&
             user.affiliation?.includes("staff@unipd.it");
    },
    resolveUserType: () => UserType.staff,
  },

  // Utente esterno (nessuna affiliation) → EXTERNAL
  {
    isApplicable: user => {
      const domain = extractDomain(user.mail, user.id);
      return domain === "unipd.it" && !user.affiliation;
    },
    resolveUserType: () => UserType.external,
  },
];


// ---------------------------
// Funzione principale
// ---------------------------

export const getUserType = (user: AuthUser): UserType | undefined =>
  strategies.find(s => s.isApplicable(user))?.resolveUserType();
```

L'algoritmo permette di ritornare una tra le classi di utente: Alumno, Studente, Staff, Collaboratore esterno, Cittadino autenticato con SPID o CIE. Questa classificazione può essere utile per associare un ruolo ad tipo di utente evitando l'assegnazione dei singoli utenti e semplificando la gestione dei ruoli e dei permessi negli applicativi costruti a partire dallo Starter Kit.

# Esempio di utilizzo di identità e permessi 

L'identità dell'utente è utile in quegli scenari nei quali è necessario limitare l'acesso ai dati all'utente che stà utilizzando il sistema. I dati all'identità dell'utente sono custoditi in sessione. La funzione `payload()` ritorna i dati utente che vengono memorizzati nella rotta `/saml/post-response`, a seguito della verifica delle credenziali dell'utente nell'IdP. I dati dell'identità sono memorizzati nel cookie di sessione e possono essere recuperati dallo stesso solo lato server.

```jsx
"use server";
// ....
import { Session } from "@/types/session";
import { AuthUser } from "@/types/auth-user";
// ....

// ....
const user: AuthUser = await Session.payload();
// ....
```

I permessi vengono utilizzati per limitare l'accesso ai dati. I permessi vengono eriditati dal ruolo dell'utente. Ci sono due modi per verificare se l'utente è in possesso di un ruolo. Lato server la funzione `isAuthorized()` utilizza l'identità utente per verificare se un permesso è presente nel cookie di sessione.

```jsx
// nextjs/app/secure/request/page.tsx

"use server";

import { getSessionPayload } from "@/components/user/actions";
import { AuthUser } from "@/types/auth-user";
import { permissionType } from "@prisma/client";
import { isAuthorized } from "@/components/common/utils";
// ...

export default async function Page() {
  const user: AuthUser | null = await getSessionPayload();
  // ...
  return (
    <>
      <!-- <meta /> ... -->
      {!isAuthorized(user, permissionType.requestRead) ? (
          <UserNotAuthorized />
        ) : (
          <MyRequests requests={myRequests()} / >
        )}
    </>
  );
}
```

Lato client  il metodo `isAuthorized()` è messo a disposizione dal componente `<AuthProvider/>`. In questo caso **non è necessario specificare il parametro utente**.

```jsx
"use client";

// ...
import React, { useContext } from "react";
import AuthContext from "@/components/context/auth-context";
import { permissionType } from "@prisma/client";
// ...

export default function RequestListItem(props: { request: Request }) {
  const { isAuthorized } = useContext(AuthContext);
	// ...
  return (<>
            <!--

            -->
            {isAuthorized(permissionType.requestList) && (
                   <li className="nav-item dropdown">
            <!--

            -->
    </>);
}
```

