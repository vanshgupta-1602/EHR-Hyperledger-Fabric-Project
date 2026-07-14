EHR on Hyperledger Fabric

A blockchain-based Electronic Health Record (EHR) system built on Hyperledger Fabric. Patients, doctors, and insurance providers are onboarded on-chain, medical records are written immutably to the ledger, and access to records is granted/controlled through chaincode-enforced permissions.

Why Blockchain for EHR?

Traditional EHR systems rely on centralized databases controlled by a single hospital or provider, making patient data siloed, hard to audit, and vulnerable to unauthorized tampering. This project uses Hyperledger Fabric — a permissioned blockchain — to give patients ownership over who can access their records, while keeping a tamper-evident, auditable history of every read/write on the ledger.

Architecture

┌─────────────────┐      ┌──────────────────┐      ┌─────────────────────┐
│   Client / API   │ ───► │  Node.js SDK      │ ───► │  Hyperledger Fabric  │
│  (Postman/Web)   │      │  (server-node-sdk)│      │  Test Network         │
└─────────────────┘      └──────────────────┘      │  - Org1 / Org2 Peers │
                                                      │  - Orderer           │
                                                      │  - CouchDB (state DB)│
                                                      │  - ehrChainCode      │
                                                      └─────────────────────┘


Chaincode (fabric-samples/asset-transfer-basic/chaincode-javascript, deployed as ehrChainCode): the smart contract enforcing onboarding, record writes, and access control.
server-node-sdk: Express.js REST API that wraps the Fabric Node SDK (fabric-network) to submit/evaluate transactions on behalf of clients.
Fabric test network: 2-org network (Org1, Org2) with CouchDB as the world-state database, enabling rich JSON queries and history lookups.


Tech Stack


Blockchain: Hyperledger Fabric 2.5, Fabric CA
Smart Contract: JavaScript (fabric-contract-api)
Backend: Node.js, Express 5, fabric-network SDK
State DB: CouchDB
Containerization: Docker & Docker Compose


Chaincode Functions

FunctionPurposeonboardPatientRegisters a new patient on the ledgeronboardDoctorRegisters a new doctoronboardInsuranceRegisters an insurance provideraddRecordWrites a new medical record for a patientgrantAccessGrants a doctor/provider access to a patient's recordsgetRecordByIdFetches a specific record by IDgetAllRecordsByPatientIdFetches all records belonging to a patientfetchLedgerReturns the full ledger statequeryHistoryOfAssetReturns the full transaction history of a given asset (audit trail)

API Endpoints

The server-node-sdk exposes the following REST endpoints (see EHR-APIs.postman_collection.json for ready-to-import requests):

MethodEndpointDescriptionGET/statusHealth checkPOST/registerPatientOnboard a new patientPOST/loginPatientAuthenticate a patientPOST/addRecordAdd a medical recordPOST/getRecordByIdGet a single recordPOST/getAllRecordsByPatientIdGet all records for a patientPOST/grantAccessGrant record access to a doctor/providerPOST/fetchLedgerFetch entire ledger statePOST/queryHistoryOfAssetGet transaction history for an asset

Getting Started

Prerequisites


Docker & Docker Compose
Node.js (v18+)
curl, jq


1. Bring up the Fabric network

bashcd fabric-samples/test-network
./network.sh up createChannel -ca -s couchdb

2. Deploy the chaincode

bash./network.sh deployCC -ccn ehrChainCode -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript

3. Start the backend

bashcd ../../server-node-sdk
npm install
npm run dev

4. Test the APIs

Import EHR-APIs.postman_collection.json into Postman and hit /status to confirm the server is up.

Known Issues


Docker Desktop for Linux users: chaincode installation can intermittently fail with docker build failed: write unix @->/run/host-services/docker.proxy.sock: write: broken pipe. This is a known Docker Desktop VM-proxy issue, not a chaincode/network bug. Workaround: switch to the native Docker Engine context (docker context use default) before running deployCC, or use the Chaincode-as-a-Service (CCaaS) builder under fabric-samples/builders/ccaas to bypass in-peer image builds entirely.


Project Structure

├── fabric-samples/          # Hyperledger Fabric test network + tooling
├── server-node-sdk/         # Express REST API + Fabric SDK integration
│   ├── app.js                # Route definitions
│   ├── helper.js              # Wallet/identity + chaincode invocation helpers
│   ├── invoke.js / query.js   # Submit/evaluate transaction wrappers
│   └── cert-script/           # Admin/user onboarding scripts (CA enrollment)
├── fabric-explorer/         # Hyperledger Explorer config for ledger visualization
├── backup/                  # Earlier iteration of the SDK layer
└── EHR-APIs.postman_collection.json

Roadmap


 Insurance claim creation & approval flow
 Researcher onboarding + consent-based data sharing
 Patient reward issuance for data-sharing consent
 Frontend dashboard for patients/doctors


License

See LICENSE.


