# Ingestion API example

This folder contains a minimal, public-safe example for sending records into Salesforce Data Cloud through the Ingestion API.

## What this example assumes

Before running `send-data.py`, complete the connect/prepare setup steps:

1. create an Ingestion API connector
2. upload the schema with `sf data360 connection schema-upsert`
3. create the corresponding data stream in the UI if your org requires that step

Related connector definitions live in:
- [../../../sf-datacloud-connect/examples/connections/ingest-api-connection.json](https://github.com/Jaganpro/sf-skills/blob/53c9956c4279f8e9abda51164a3e9444811f0e8e/skills/sf-datacloud-connect/examples/connections/ingest-api-connection.json)
- [../../../sf-datacloud-connect/examples/connections/ingest-api-schema.json](https://github.com/Jaganpro/sf-skills/blob/53c9956c4279f8e9abda51164a3e9444811f0e8e/skills/sf-datacloud-connect/examples/connections/ingest-api-schema.json)

## Prerequisites

```bash
pip install PyJWT cryptography requests
```

## Setup

```bash
cd skills/sf-datacloud-prepare/examples/ingestion-api
cp .env.example .env
# edit .env with your values
python3 send-data.py
```

## Environment variables

- `CONSUMER_KEY` — external client app consumer key
- `CONSUMER_SECRET` — external client app consumer secret if your auth flow needs it
- `SF_USERNAME` — Salesforce username used for JWT auth
- `SF_LOGIN_URL` — login host such as `https://login.salesforce.com`
- `TENANT_URL` — Data Cloud tenant URL such as `https://<tenant>.c360a.salesforce.com`
- `PRIVATE_KEY_FILE` — path to the JWT private key
- `CONNECTOR_NAME` — Ingestion API connector name
- `OBJECT_NAME` — uploaded schema object name

## Notes

- auth is a staged flow: JWT → Salesforce token → Data Cloud token
- the ingestion endpoint uses the Data Cloud tenant URL, not the Salesforce instance URL
- `202` means the payload was accepted for processing
- validation failures often appear in the Problem Records DLO family

<!-- Modified by Automation Skills on 2026-11-24: relative links to files that were not copied now point to the original repository. -->
