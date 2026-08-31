---
title: Integrazione firma digitale in SOUL
description: Come integrare i servizi di firma digitale nelle applicazioni create con framework SOUL (e non solo), utilizzando il Middleware di Firma Digitale (MFD) dell'Università di Padova.
published: true
date: 2026-08-12:09:00.000+02:00
tags: [mfd, firma-digitale]
editor: markdown
dateCreated: 2026-08-31T11:00:00.000+02:00
---

# Integrazione della firma digitale in SOUL

Per utilizzare i servizi di firma digitale nelle applicazioni create con framework SOUL (e non solo), è possibile utilizzare il **Middleware di Firma Digitale (MFD)** dell'Università di Padova.

L'MFD fornisce le API REST per l'orchestrazione dei **flussi di firma digitale** attraverso due provider, esponendo
un'unica interfaccia indipendente dal provider alle applicazioni che devono far firmare
documenti.

Provider di firma supportati:

| Provider | Tipo di firma | Note |
|----------|----------------|-------|
| **U-Sign** (CINECA) | **FEQ** — Firma Elettronica Qualificata | Richiede un certificato di firma valido registrato su U-Sign. Un processo di firma per firmatario. |
| **Namirial eSignAnywhere v6** | **FEA** — Firma Elettronica Avanzata | OTP tramite SMS. Un'unica busta contenente tutti i firmatari. |



## Come usare l'API MFD UNIPD

Per usare l'API MFD sono necessari:

1. un `applicationId` (UUID) che identifica l'applicazione;
2. una chiave API per l'autenticazione tramite l'API Gateway.

Per ottenere queste credenziali, scrivi a
[ufficio.applicativi@unipd.it](mailto:ufficio.applicativi@unipd.it), mettendo in copia
[piattaforme.applicativi@unipd.it](mailto:piattaforme.applicativi@unipd.it).

L'API Gateway espone endpoint distinti per staging e produzione:

| Ambiente | URL base |
|-------------|----------|
| Staging | https://apigw-staging.ict.unipd.it/mfd/ |
| Produzione | https://apigw-production.ict.unipd.it/mfd/ |

Per autorizzare le richieste, includi l'header `apikey` con il valore `<API_KEY>` fornito
dall'ufficio ICT UNIPD (Ufficio Applicativi, Area Servizi Informatici e Telematici — ASIT).

Ad esempio, puoi verificare l'endpoint di stato del servizio con:

```bash
curl -H "apikey: <API_KEY>" https://apigw-production.ict.unipd.it/mfd/health
```

La documentazione API completa è disponibile in formato OpenAPI (OAS) tramite Swagger UI all'indirizzo:
[https://apigw-production.ict.unipd.it/mfd/docs/](https://apigw-production.ict.unipd.it/mfd/docs/).

## Caricare i documenti e creare un flusso di firma

### 1. Caricare i file

Invia uno o più file come `multipart/form-data`:

```bash
curl -X POST 'https://apigw-production.ict.unipd.it/mfd/documents' \
  -H 'apikey: <API_KEY>' \
  -F 'applicationId=74ab0186-f0e3-4f15-8796-a70636e845e6' \
  -F 'files=@document_1.pdf' \
  -F 'files=@document_2.pdf'
```

Esempio di risposta:

```json
{
  "documents": [
    {
      "documentId": "a30385be-1ecb-46ba-9a04-e272b7feb675",
      "filename": "document_1.pdf",
      "mimeType": "application/pdf"
    },
    {
      "documentId": "4b39c696-bc3e-49d3-a174-9e39cb96974c",
      "filename": "document_2.pdf",
      "mimeType": "application/pdf"
    }
  ]
}
```

### 2. Creare il flusso di firma

Fai riferimento ai documenti caricati usando i valori `documentId` restituiti:

```bash
curl -X POST 'https://apigw-production.ict.unipd.it/mfd/signflows' \
  -H 'Content-Type: application/json' \
  -H 'apikey: <API_KEY>' \
  -d '{
    "applicationId": "74ab0186-f0e3-4f15-8796-a70636e845e6",
    "description": "Firma digitale per convenzione e progetto formativo",
    "documentIds": [
      "a30385be-1ecb-46ba-9a04-e272b7feb675",
      "4b39c696-bc3e-49d3-a174-9e39cb96974c"
    ],
    "signersSequential": [
      {
        "signerId": "mario.rossi@unipd.it",
        "requiredSignature": "FEQ",
        "requiredSignatureType": "PADES_GRAFICO",
        "placeholderSignature": "signer1"
      },
      {
        "signerId": "andrea.bianchi@org.it",
        "firstName": "Andrea",
        "lastName": "Bianchi",
        "phone": "+390123456789",
        "requiredSignature": "FEA",
        "signaturePositions": [
          {
            "documentNumber": 1,
            "pageNumber": 1,
            "x": 350,
            "y": 120,
            "width": 180,
            "height": 70
          }
        ]
      }
    ]
  }'
```

I documenti possono anche essere forniti **inline** tramite l'array `documents`
(`{ filename, mimeType, base64Content }`) al posto di `documentIds`. È necessario fornire
almeno uno dei due. I documenti caricati vengono **consumati** (rimossi dallo storage
temporaneo) alla creazione del flusso.


#### Posizione della firma
La posizione della firma è gestita in due modi:
- **Segnaposto** `placeholderSignature`: il firmatario può firmare in un'area predefinita del documento PDF (placeholder di firma), indicata da un segnaposto identificato da una chiave univoca, ad esempio: `signer1`. Il placeholder di firma nel documento da firmare può essere inserito con programmi di editing PDF come Adobe Acrobat, LibreOffice Draw o simili. Per l'inserimento programmatico del placeholder in ambiente Node.js, la strada più semplice è `pdf-lib` + `@signpdf/placeholder-pdf-lib`.
- **Posizione esplicita** `signaturePositions`: il firmatario può firmare in una posizione specifica del documento PDF, indicata dalla pagina del documento dove si intende apporre la firma (ad esempio la numero 1) e dalle coordinate (x, y) e dimensioni (width, height) dell'area di firma.

> **Note**
> - Per i file di grandi dimensioni, preferisci `documentIds` perché il contenuto base64 inline aumenta la dimensione della richiesta.
> - `phone`, `firstName` e `lastName` sono obbligatori per i firmatari FEA che usano OTP-SMS su Namirial.
> - Un firmatario non può comparire più di una volta in `signersSequential`.

Esempio di risposta:

```json
{
  "flowId": "b1f4f9a1-31d5-4df3-ab2e-eb840a283e07",
  "status": "pending",
  "dtCreated": "2026-07-24T10:43:47.230Z"
}
```

Dopo la creazione del flusso di firma, il primo firmatario riceve un'email. Ogni firmatario
successivo viene avvisato soltanto dopo che il precedente ha completato la propria firma.

## Monitorare un flusso di firma

Sostituisci `<FLOW_ID>` con l'ID restituito alla creazione del flusso:

```bash
curl -X GET 'https://apigw-production.ict.unipd.it/mfd/signflows/<FLOW_ID>/status' \
  -H 'apikey: <API_KEY>' \
  -H 'Accept: application/json'
```

Esempio di risposta:

```json
{
  "flowId": "3f77d566-d46a-41e7-a397-1f6cf754bd40",
  "status": "pending",
  "applicationId": "74ab0186-f0e3-4f15-8796-a70636e845e6",
  "dtCreated": "2026-04-01T10:00:00Z",
  "dtUpdated": "2026-04-13T10:00:00Z",
  "signaturesListStatus": [
    {
      "signerId": "mario.rossi@unipd.it",
      "firstName": "Mario",
      "lastName": "Rossi",
      "requiredSignature": "FEQ",
      "signatureStatus": {
        "signatureStatus": "signed",
        "dtSigned": "2026-04-02T10:00:00Z"
      }
    },
    {
      "signerId": "andrea.bianchi@org.it",
      "firstName": "Andrea",
      "lastName": "Bianchi",
      "requiredSignature": "FEA",
      "signatureStatus": {
        "signatureStatus": "pending"
      }
    }
  ]
}
```

## Scaricare i documenti firmati

Quando lo stato del flusso è `signed`, scarica i documenti finali con:

```bash
curl -X GET 'https://apigw-production.ict.unipd.it/mfd/signflows/<FLOW_ID>/download' \
  -H 'apikey: <API_KEY>' \
  -H 'Accept: application/octet-stream' \
  -o 'signed-documents.zip'
```

Per le istruzioni sullo sviluppo locale e la documentazione completa del progetto, consulta
[`README.md`](https://github.com/Piattaforme-Applicativi/mfd-api/blob/main/README.md).

