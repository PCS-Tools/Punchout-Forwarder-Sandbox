# Punchout-Forwarder-Sandbox

Tiny Express relay between a customer's Coupa **test** instance and the NetSuite
**SB2** punchout Suitelet (`customscript_punchout_order_suitelet`, script 1279).
Coupa POSTs cXML (PunchOutSetupRequest / OrderRequest) here; the relay forwards
the body unchanged to the Suitelet's external URL and returns the Suitelet's cXML
response. If NetSuite exceeds the timeout, the relay answers Coupa with a
synthetic cXML 200 so the order is not retried (the Suitelet still finishes).

## Hosting

Runs on **Railway** (workspace PCS Projects, project `punchout-forwarder-sandbox`),
deployed from the `main` branch of this repo. Previously Heroku app `punchout-forward-sb`.

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `PORT` | `3000` | Listen port (Railway injects this). |
| `NETSUITE_SUITELET_URL` | SB2 external URL of script 1279 | Where cXML is forwarded. Set this to re-point the relay without a code change. |
| `NETSUITE_TIMEOUT_MS` | `28000` | Upstream timeout before the relay answers Coupa with a synthetic cXML 200. |

`GET /health` returns `ok` (Railway health check). `GET /` returns `Home Page`.

## Coupa side

Supplier Punchout URL and Order URL = this service's public URL (`https://<railway-domain>/`).
Credentials are validated by the Suitelet, not here: Sender Identity = the parent
customer's NetSuite internal id, SharedSecret = `custrecord_po_shared_secret` on the
matching `customrecord_punchout_config` record. The setup response redirects the
buyer to the SB2 storefront `dev-pcs.theplsstore.com`.

## Local run

```
npm install
npm start
curl -X POST -H "Content-Type: text/xml" --data @setup.xml http://localhost:3000/
```
