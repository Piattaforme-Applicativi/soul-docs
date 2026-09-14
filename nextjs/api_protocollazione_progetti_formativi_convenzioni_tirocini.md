---
title: Guida API di Protocollazione - Use Case: Progetti formativi e convenzioni di tirocinio.
description: 
published: true
date: 2026-09-14T16:24:53.601Z
tags: 
editor: markdown
dateCreated: 2026-09-14T16:13:19.502Z
---

# Guida API di Protocollazione

## Use Case: Progetti formativi e convenzioni di tirocinio.

## Autori:
- UNIPD (Università di Padova), Area Servizi Informatici e Telematici di Ateneo (ASIT), Ufficio Applicativi, Settore Tecnologie Emergenti;
- DonQ.

## Ultimo aggiornamento:
2026-09-14

---

# 1. Introduzione

Il servizio di **Protocollazione** espone delle API REST/JSON che fanno da ponte verso il
protocollo informatico (Titulus) di UNIPD.

Vengono fornite le API per le seguenti finalità:

## Anagrafica
Permette di:

  1. cercare persone (interne/esterne);
  2. creare persone esterne all'Ateneo, ovvero non inquadrate come personale interno dipendente dell'Ateneo (ad esempio studenti o collaboratori esterni);
  3. cercare strutture (interne/esterne);
  4. creare strutture esterne, ovvero enti/aziende esterni all'Ateneo.

## Protocollazione
Permette di protocollare in Titulus le convenzioni e i progetti formativi relativi ai tirocini.

> **Flusso tipico:** prima si recupera (o si crea) in anagrafica la struttura esterna (ente) e/o lo studente (persona esterna) per
ottenere il loro identificativo (`id`/`physdoc`); poi si chiama l'endpoint di
protocollazione passando gli stessi id.

# 2. Informazioni generali

## 2.1 Base URL e percorsi

Gli endpoint sono esposti sotto il prefisso `/career/api`:
```text
https://<host-del-servizio>/career/api/...
```

**NB**: in precedenza gli stessi endpoint (eventualmente con diverso nome) erano disponibili anche sotto il prefisso `/api` (es.
`/api/anagrafica/ente`), ma sono stati deprecati e rimossi come anticipato nella precedente versione della presente documentazione.

## 2.2 Autenticazione

Le chiamate richiedono l'header HTTP apikey (autenticazione via API key):
```text
apikey: <valore-fornito-separatamente>
```

## 2.3 Formato

Richieste e risposte sono in JSON (Content-Type: application/json).

### Risposta di successo:
```json
{
  "success": true,
  "message": "Data created successfully",
  "data": {
  }
}
```

**NB**: all'interno del campo `data` è presente il contenuto specifico restituito dall' endpoint.

**NB**: message è presente solo nelle operazioni di creazione/protocollazione.

### Risposta di errore:
```json
{
  "success": false,
  "error": "ValidationError",
  "code": "VALIDATION_ERROR",
  "message": "descrizione dell'errore",
  "details": []
}
```

## 2.4 Codici di stato HTTP

| Codice                    | Significato                                                                |
|---------------------------|----------------------------------------------------------------------------|
| 200 OK                    | Ricerca completata (anche se senza risultati).                             |
| 201 Created               | Risorsa creata / documento protocollato.                                   |
| 400 Bad Request           | Errore di validazione, parametri mancanti o entità non trovata in Titulus. |
| 500 Internal Server Error | Errore interno o errore restituito da Titulus.                             |

# 3. Glossario

| Termine    | Descrizione                                                                                                  |
|------------|--------------------------------------------------------------------------------------------------------------|
| physdoc    | Identificativo univoco e permanente di un record in Titulus (è l' id restituito dalle anagrafiche).          |
| protocollo | Numero di protocollo assegnato al documento al momento della registrazione.                                  |
| repertorio | Numero di repertorio assegnato al documento (registro dei contratti/convenzioni).                            |
| fascicolo  | Cartella documentale in cui il documento protocollato viene inserito (gestito automaticamente dal servizio). |

# 4. Endpoint Anagrafica

## 4.1 GET /career/api/anagrafica/persona-esterna

### Cosa fa
Cerca una persona esterna all'Ateneo (ad esempio uno studente) in Titulus tramite codice fiscale.

|   Parametro       |   Obbligatorio  |   Descrizione                   |
|-------------------|-----------------|---------------------------------|
|   codice_fiscale  |   Sì            |   Codice fiscale della persona  	|

### Controlli
Se `codice_fiscale` è assente, la richiesta fallisce con 400. 

### Risposta 200
`data` è un array con 0 o 1 elemento (mai 404 se non trovata: viene
restituito un array vuoto).

```json
{
  "success": true,
  "data": [
    {
      "id": "RSSMRA90A15H501Z",
      "codice_fiscale": "RSSMRA90A15H501Z",
      "nome": "Mario",
      "cognome": "Rossi",
      "recapito": {
        "indirizzo": {
          "value": "Via Roma 123",
          "cap": "35100",
          "comune": "Padova",
          "prov": "PD",
          "nazione": "Italia"
        },
        "telefono": [
          {
            "num": "+39 049 1234567",
            "tipo": "fisso"
          }
        ],
        "email": "mario.rossi@example.com",
        "email_certificata": "mario.rossi@pec.it"
      }
    }
  ]
}
```

> Il campo `recapito` consolida automaticamente i dati di recapito presenti in Titulus;
`telefono` è sempre un array.

## 4.2 POST /career/api/anagrafica/persona-esterna

### Cosa fa:

Crea una nuova persona esterna all'Ateneo (ad esempio uno studente) in Titulus. L' id (`physdoc`) restituito va usato come `studente.id` in fase di protocollazione.

### Corpo della richiesta:

|   Campo                       |   Obbligatorio              |   Formato / note                                            |
|-------------------------------|-----------------------------|-------------------------------------------------------------|
|   nome                        |   Sì                        |   Stringa                                                   |
|   cognome                     |   Sì                        |   Stringa                                                   |
|   luogo_nascita               |   Sì                        |   Stringa                                                   |
|   nazione_nascita             |   Sì                        |   Stringa                                                   |
|   data_nascita                |   No                        |   Formato GG/MM/AAAA (es.31/01/1990)                        |
|   sesso                       |   No                        |   M oppure F                                                |
|   codice_fiscale              |   No                        |   Codice fiscale italiano valido (16caratteri)              |
|   partita_iva                 |   No                        |   Stringa                                                   |
|   recapito                    |   No                        |   Oggetto (vedi sotto)                                      |
|   recapito.indirizzo          |   Sì (se recapitopresente)  |   { nazione, prov, comune,cap, value } — tutti obbligatori  |
|   recapito.telefono           |   Sì (se recapitopresente)  |   Array di { num, tipo }                                    |
|   recapito.email              |   Sì (se recapitopresente)  |   Email valida                                              |
|   recapito.email_certificata  |   No                        |   Email valida                                              |
|   appartenenza                |   No                        |   { cod_uff, qualifica } —entrambi obbligatori se presente  |
|   competenze                  |   No                        |   Stringa                                                   |




### Formati dei sotto-campi:
- `cap`: 1–10 caratteri (cifre/spazi/trattini);
- `telefono.num`: numero telefonico (prefisso internazionale opzionale, es. +39 049 1234567);
- `telefono.tipo`: 'fisso' o 'mobile'.

### Esempio richiesta:

```json
{
  "nome": "Mario",
  "cognome": "Rossi",
  "data_nascita": "31/01/1990",
  "luogo_nascita": "Padova",
  "nazione_nascita": "Italia",
  "sesso": "M",
  "codice_fiscale": "RSSMRA90A15H501Z",
  "recapito": {
    "indirizzo": {
      "nazione": "Italia",
      "prov": "PD",
      "comune": "Padova",
      "cap": "35100",
      "value": "Via Roma 123"
    },
    "telefono": [
      {
        "num": "+39 049 1234567",
        "tipo": "fisso"
      }
    ],
    "email": "mario.rossi@example.com",
    "email_certificata": "mario.rossi@pec.it"
  }
}
```

### Risposta 201
`data` contiene l'oggetto persona creato (stesso formato della ricerca), incluso `id`.

## 4.3 GET /career/api/anagrafica/struttura-esterna

### Cosa fa:
Ricerca una struttura esterna (azienda/ente esterno all'Ateneo) per codice fiscale o partita IVA.

### Parametri query:

| Parametro | Obbligatorio | Descrizione |
|---|---|---|
| `codice_fiscale` | Almeno uno tra i due | Codice fiscale dell'ente |
| `partita_iva` | Almeno uno tra i due | Partita IVA dell'ente |
| `nome` | No | Filtro aggiuntivo per ragione sociale |

### Controlli e comportamento:

- Deve essere fornito **almeno uno** tra `codice_fiscale` e `partita_iva`. Se mancano
entrambi → 400.
- Se vengono forniti entrambi, **la partita IVA ha la precedenza**.
- I valori `"null"`, `"undefined"` e stringa vuota vengono ignorati (trattati come assenti).
- `nome` è un **filtro applicato lato servizio** dopo la risposta di Titulus (non è una ricerca
nativa): match parziale e **case-sensitive**. Utile quando Titulus restituisce più record con lo
stesso CF/partita IVA.

### Risposta 200

`data` è un array di strutture esterne (enti/aziende esterne all'Ateneo). Vuoto se nessun risultato.

```json
{
  "success": true,
  "data": [
    {
      "id": "12345678901",
      "nome": "Azienda Example S.r.l.",
      "codice_fiscale": "12345678901",
      "partita_iva": "IT12345678901",
      "indirizzo": {
        "value": "Via Industria 45",
        "cap": "35100",
        "comune": "Padova",
        "prov": "PD",
        "nazione": "Italia"
      },
      "telefono": [
        {
          "num": "+39 049 9876543",
          "tipo": "fisso"
        }
      ],
      "email": "info@example.com",
      "email_certificata": "azienda@pec.it",
      "note": "Azienda partner per tirocini"
    }
  ]
}
```

## 4.4 POST /career/api/anagrafica/struttura-esterna

### Cosa fa: 
Crea una nuova struttura esterna (ente/azienda esterna all'Ateneo) in Titulus. L' id (`physdoc`) restituito va usato
come `ente.id` in fase di protocollazione.

### Corpo della richiesta:
| Campo | Obbligatorio | Formato / note |
|---|---|---|
| `codice_fiscale` | Sì | Stringa |
| `partita_iva` | Sì | Stringa |
| `nome` | Sì | Ragione sociale |
| `indirizzo` | Sì | `{ nazione, prov, comune, cap, value }` — tutti obbligatori |
| `telefono` | Sì | Array di `{ num, tipo }` (può essere vuoto) |
| `email` | Sì | Email valida |
| `tipologia` | No | Default `"ENTE"` |
| `email_certificata` | No | Stringa |
| `sito_web` | No | Array di `{ url }` |
| `note` | No | Stringa |


### Esempio richiesta:
```json
{
  "codice_fiscale": "12345678901",
  "partita_iva": "IT12345678901",
  "nome": "Azienda Example S.r.l.",
  "indirizzo": {
    "nazione": "Italia",
    "prov": "PD",
    "comune": "Padova",
    "cap": "35100",
    "value": "Via Industria 45"
  },
  "telefono": [
    {
      "num": "+39 049 9876543",
      "tipo": "fisso"
    }
  ],
  "email": "info@example.com",
  "email_certificata": "azienda@pec.it",
  "sito_web": [
    {
      "url": "https://www.example.com"
    }
  ],
  "note": "Azienda partner per tirocini"
}
```

### Risposta 201:
`data` contiene l'`id` della struttura esterna creata.

```json
{
  "success": true,
  "message": "Data created successfully",
  "data": {
    "id": "12345678901"
  }
}
```

# 5. Endpoint Protocollazione

Entrambi gli endpoint:
1. recuperano i dati della struttura esterna (ente) e della persona esterna (ad es. studente) da Titulus tramite `id`;
2. creano e protocollano il documento;
3. lo inseriscono nel fascicolo dell'anno corrente;
4. (se richiesto) avviano la notifica PEC ai destinatari;
5. restituiscono numero di protocollo, repertorio ed esito della notifica.

> **Prerequisito:** struttura esterna (ente) e persona esterna (studente) devono **esistere già** in Titulus. È necessario quindi recuperare il loro `id` con
gli endpoint di anagrafica prima di protocollare.

## 5.1 POST /career/api/protocolla/convenzione-tirocinio


### Cosa fa:
Protocolla una convenzione di tirocinio.

### Corpo della richiesta:

| Campo | Obbligatorio | Note |
|---|---|---|
| `oggetto` | Sì | Oggetto della convenzione |
| `autore` | Sì | Nome e cognome dell'autore |
| `ente` | Sì | Vedi sotto |
| `ente.id` | Sì | `physdoc` dell'ente (da `GET /api/anagrafica/ente`) |
| `ente.firmatario` | No | Nome e cognome del firmatario dell'ente |
| `ente.notifica` | No | `true` per inviare la notifica PEC all'ente |
| `unipd.firmatario` | No | Nome e cognome del firmatario UNIPD |
| `note` | No | Note libere |
| `allegati` | No | Array di allegati (vedi §5.3) |

### Esempio richiesta:

```json
{
  "oggetto": "Convenzione di tirocinio di formazione e di orientamento per tirocini curriculari",
  "autore": "Rossi Mario",
  "unipd": {
    "firmatario": "Mario Rossi"
  },
  "ente": {
    "id": "12345678901",
    "firmatario": "Mario Rossi",
    "notifica": true
  },
  "note": "Documento generato"
}
```

## 5.2 POST /career/api/protocolla/progetto-formativo-tirocinio

### Cosa fa:
Protocolla un progetto formativo.

### Corpo della richiesta:
Come sopra, con in più lo **studente**.

| Campo | Obbligatorio | Note |
|---|---|---|
| `oggetto` | Sì | Oggetto del progetto formativo |
| `autore` | Sì | Nome e cognome dell'autore |
| `ente.id` | Sì | `physdoc` dell'ente |
| `ente.firmatario` | No | Firmatario dell'ente |
| `ente.notifica` | No | `true` per notifica PEC all'ente |
| `studente.id` | Sì | `physdoc` dello studente (da `GET /api/anagrafica/persona`) |
| `studente.firmatario` | No | Se assente, si usa nome+cognome da anagrafica |
| `studente.notifica` | No | `true` per notifica PEC allo studente |
| `unipd.firmatario` | No | Firmatario UNIPD |
| `note` | No | Note libere |
| `allegati` | No | Array di allegati (vedi §5.3) |

### Esempio richiesta:

```json
{
  "oggetto": "Progetto formativo di tirocinio curriculare",
  "autore": "Rossi Mario",
  "unipd": {
    "firmatario": "Mario Rossi"
  },
  "ente": {
    "id": "12345678901",
    "firmatario": "Mario Rossi",
    "notifica": false
  },
  "studente": {
    "id": "RSSMRA90A15H501Z",
    "firmatario": "Mario Rossi",
    "notifica": true
  },
  "note": "Documento generato"
}
```

## 5.3 Allegati (entrambi gli endpoint)

`allegati` è un array opzionale. Se presente, **ogni allegato è obbligatorio in tutti i suoi
campi**:

| Campo | Note |
|---|---|
| `nome` | Nome file con estensione (es. `documento.pdf`) |
| `descrizione` | Descrizione visibile in Titulus |
| `contenuto` | Contenuto del file in Base64. Sono accettati solo PDF (il tipo è fisso a `application/pdf`) |

```json
{
  "allegati": [
    {
      "nome": "progetto.pdf",
      "descrizione": "Progetto formativo firmato",
      "contenuto": "<PDF in Base64>"
    }
  ]
}
```

## 5.4 Valori impostati automaticamente

I seguenti dati sono impostati dal servizio di Protocollazione e **non** è possibile settarli nella
richiesta: repertorio, classificazione, ufficio responsabile (RPA), riferimento esterno UNIPD,
fascicolo di destinazione.

# 6. Recupero del Numero di protocollo e del Repertorio (risposta della protocollazione)

Entrambi gli endpoint di protocollazione, in caso di successo, rispondono con `201` e nel
campo `data` riportano protocollo, repertorio ed esito della notifica.

### Esempio risposta:
```json
{
  "success": true,
  "message": "Data created successfully",
  "data": {
    "protocollo": {
      "cod": "2026-UNPD0Z9-0000162",
      "number": "162",
      "date": "18/05/2026"
    },
    "repertorio": {
      "number": "39/2026",
      "value": "Contratti - Convenzioni"
    },
    "notifica": {
      "inviata": true,
      "destinatari": {
        "ente": true,
        "studente": true
      }
    }
  }
}
```

### Dettaglio dei campi:
| Campo | Descrizione |
|---|---|
| `protocollo.cod` | Codice completo del protocollo assegnato da Titulus (es. `2026-UNPD0Z9-0000162`) |
| `protocollo.number` | Numero progressivo del protocollo (es. `162`) |
| `protocollo.date` | Data di protocollazione, formato `GG/MM/AAAA` |
| `repertorio.number` | Numero di repertorio, formato `progressivo/anno` (es. `39/2026`) |
| `repertorio.value` | Descrizione del repertorio (es. `Contratti - Convenzioni`) |
| `notifica.inviata` | `true` se il workflow di notifica PEC è stato avviato |
| `notifica.destinatari` | Presente quando `inviata = true`. Indica chi è stato notificato: `{ "ente": boolean, "studente": boolean }`. Il campo `studente` è presente solo per il progetto formativo. |
| `notifica.errore` | Presente solo se l'avvio della notifica è fallito (vedi §6.1) |


## 6.1 Notifica PEC ai destinatari

Impostando `notifica: true su ente e/o `studente nella richiesta, dopo la
protocollazione il servizio invia una **PEC** al soggetto (o l'email ordinaria se la PEC non è
disponibile in anagrafica).

### Possibili valori dell'oggetto `notifica` nella risposta:

| Caso | Valore |
|---|---|
| Nessuna notifica richiesta | `{ "inviata": false }` |
| Notifica avviata | `{ "inviata": true, "destinatari": { "ente": true, "studente": false } }` |
| Avvio notifica fallito | `{ "inviata": false, "errore": "Notifica PEC non avviata: il documento è stato protocollato correttamente, la notifica va gestita manualmente." }` |

> **Importante:** la protocollazione è **irreversibile**. Se il documento viene protocollato
correttamente ma l'avvio della notifica fallisce, la risposta resta `201` con
`notifica.inviata = false` e un messaggio in `notifica.errore`: **il numero di
protocollo è valido**, va solo gestita manualmente la notifica.

# 7. Controlli di validazione che bloccano creazione anagrafica e protocollazione

Tutti i controlli seguenti, se non superati, **interrompono l'operazione** e restituiscono un
errore. In particolare, per la protocollazione, i controlli di esistenza e di email avvengono
**prima** della creazione del documento: se falliscono, **nessun protocollo viene consumato**.

## 7.1 POST /api/carrer/anagrafica/persona-esterna

### Cosa fa:
Creazione anagrafica persona esterna (studente) in Titulus.

### Controlli:

| Controllo | Esito se non superato |
|---|---|
| `nome`, `cognome`, `luogo_nascita`, `nazione_nascita` presenti | 400 |
| `data_nascita`, se presente, nel formato `GG/MM/AAAA` | 400 |
| `sesso`, se presente, uguale a `M` o `F` | 400 |
| `codice_fiscale`, se presente, formato CF italiano valido (16 caratteri) | 400 |
| Se `recapito` è presente: indirizzo completo (`nazione`, `prov`, `comune`, `cap`, `value`), telefono valido, email valida | 400 |
| `telefono[].tipo` uguale a `fisso` o `mobile`; `telefono[].num` in formato valido | 400 |


## 7.2 POST /career/api/anagrafica/struttura-esterna

### Cosa fa:
Creazione anagrafica struttura esterna in Titulus.

### Controlli:

| Controllo | Esito se non superato |
|---|---|
| `codice_fiscale`, `partita_iva`, `nome` presenti | 400 |
| Indirizzo completo (`nazione`, `prov`, `comune`, `cap`, `value`) | 400 |
| Email valida | 400 |
| `telefono[]` valido (`num` + `tipo` ∈ { `fisso`, `mobile` }) | 400 |

## 7.3 Protocollazione — entrambi gli endpoint

### Controlli

| Controllo | Esito se non superato | Messaggio |
|---|---|---|
| `oggetto` e `autore` presenti | 400 | dettaglio di validazione |
| `ente.id` presente | 400 | dettaglio di validazione |
| `studente.id` presente (solo progetto formativo) | 400 | dettaglio di validazione |
| L'ente identificato da `ente.id` esiste in Titulus | 400 | `Ente {id} not found` |
| Lo studente identificato da `studente.id` esiste in Titulus (solo progetto formativo) | 400 | `Studente {id} not found` |
| Se `ente.notifica = true`: l'ente ha almeno una email in anagrafica | 400 | `Ente {id} non ha una email per la notifica PEC` |
| Se `studente.notifica = true`: lo studente ha almeno una email in anagrafica | 400 | `Studente {id} non ha una email per la notifica PEC` |
| Se `allegati` presente: ogni allegato ha `nome`, `descrizione`, `contenuto` | 400 | dettaglio di validazione |

> **Nota sulla notifica:** la mail deve essere sempre presente per un soggetto da notificare; la PEC è facoltativa (se assente, la notifica usa l'email). Se un soggetto ha `notifica: true` ma non ha alcuna email in anagrafica, la protocollazione non viene effettuata.

## 7.4 Forma degli errori di validazione

Gli errori di validazione del corpo/parametri (`400`) restituiscono l'envelope di errore con
error: "ValidationError" e, in `message`, il dettaglio dei campi non validi:

```json
{
  "success": false,
  "error": "ValidationError",
  "code": "VALIDATION_ERROR",
  "message": "...dettaglio dei campi non validi...",
  "details": []
}
```

Gli errori provenienti da Titulus mantengono lo stato e il messaggio restituiti dal protocollo informatico.

# 8. Endpoint di servizio (facoltativi)

| Metodo | Path | Descrizione |
|---|---|---|
| GET | `/health/live` | Liveness probe — risponde `200 { "message": "OK" }` |
| GET | `/health/ready` | Readiness probe — risponde `200 { "message": "OK" }` |

La documentazione interattiva (Swagger UI) è disponibile su https://apigw-staging.ict.unipd.it/protocollazione/docs/.
