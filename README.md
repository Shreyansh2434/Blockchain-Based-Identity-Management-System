# Blockchain-Based-Identity-Management-System


Compact, privacy-preserving demo of signed digital credentials (issue → present → verify → revoke). Minimal on-chain footprint (hashes/flags only), simple Node.js backend, lightweight frontend and optional smart-contract layer.

Table of contents

Project overview

Key features

Architecture & tech stack

Quick start (dev)

API reference (core endpoints)

Credential format & data model

Revocation strategy

Security & key management

Testing & demo

Deployment notes

Team & deliverables

Contributing / License / Contact

Project overview

This repository contains a proof-of-concept identity system that issues signed Course Completion Certificates to holders (students), allows holders to present credentials, and enables verifiers to validate authenticity and check revocation — all while keeping PII off-chain and minimizing data exposure. Implementation focuses on clarity, reproducibility, and a short demo workflow (issue → present → verify → revoke).

Key features

Deterministic JSON credential signing (RSA / Ed25519-compatible).

Backend APIs: /issue, /verify, /revoke, /issuer/public-key.

MySQL metadata store for credential hashes + revocation status.

Optional blockchain layer: store keccak256/sha256 credential hash in a minimal smart contract (issue/revoke flags only).

Postman collection + Swagger UI for easy testing.

Demo scripts to reproduce full flow locally.

Architecture & tech stack

Backend: Node.js + Express.js (REST)

Blockchain (optional): Ethers.js / Web3.js + minimal smart contract (mapping bytes32 hash => bool revoked) — Hardhat for local testing

Database: MySQL (local or Docker)

Auth: JWT (RS256) for admin/issuer operations

Docs / Tests: Swagger UI (/api-docs), Postman collection, unit & integration tests

Frontend: Minimal static pages (issue / present / verify) — hostable on GitHub Pages / Netlify

Quick start (dev)
prerequisites

Node.js v18+

MySQL (local or Docker)

Git, npm/yarn

(optional) Hardhat / Ganache for smart contract tests

example .env
PORT=3000
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASS=password
DB_NAME=credentials_db
JWT_PRIVATE_KEY_PATH=./keys/issuer_private.pem
JWT_PUBLIC_KEY_PATH=./keys/issuer_public.pem
BLOCKCHAIN_ENABLED=false
CONTRACT_ADDRESS=
WEB3_PROVIDER_URL=http://localhost:8545

install & run
git clone <repo-url>
cd <repo-directory>
npm install
# initialize DB (run migrations)
npm run migrate
# start backend
npm run dev
# open http://localhost:3000 and/or http://localhost:3000/api-docs

API reference (core endpoints)

All endpoints accept/return JSON. Issuer/revoke require JWT auth.

POST /issue

Issue a signed credential.
Request (example):

{
  "subject": { "id": "student-123", "name": "Alice" },
  "claims": { "course": "Intro to Blockchain", "grade": "A" },
  "expiry": "2026-09-01T00:00:00Z"
}


Response:

{
  "credential": { /* canonical credential JSON + signature */ },
  "credential_hash": "0x..."
}

POST /verify

Verify presented credential (signature / time / revocation).
Request:

{ "credential": { /* full signed credential JSON */ } }


Response:

{ "status": "PASS" | "FAIL", "reasons": [ "signature_invalid", "revoked", "expired" ] }

POST /revoke

Revoke a credential by hash or id. Requires issuer auth.
Request:

{ "credential_hash": "0x..." }

GET /issuer/public-key

Return issuer public key for client-side verification.

Credential format & data model

Canonical credential JSON example:

{
  "id": "urn:uuid:...",
  "issuer": { "id": "urn:org:uni", "name": "Dept. of Informatics" },
  "subject": { "id": "student-123", "name": "Alice" },
  "claims": { "course": "Intro to Blockchain", "grade": "A" },
  "issued": "2025-09-01T00:00:00Z",
  "expiry": "2026-09-01T00:00:00Z",
  "signature": "<base64-or-hex-signature>"
}


Sign canonicalized JSON with deterministic key ordering.

Compute and store sha256/keccak256 hash of canonical JSON for DB and optional on-chain writes.

Revocation strategy

Primary (off-chain): MySQL credentials table with credential_hash and revoked boolean for fast checks.

Optional (on-chain): Minimal contract mapping bytes32 => bool with Issued/Revoked events. Backend optionally calls contract methods and syncs via event listeners. On-chain storage contains no PII.

Security & key management

Demo: keys stored in /keys (not production-safe).

Production recommendations:

Use HSM or cloud KMS for private keys.

Serve APIs over HTTPS, enforce validation and rate limiting.

Rotate issuer keys; publish key metadata and key IDs.

Restrict CI exposure of deployer keys for on-chain interactions.

Testing & demo

Unit tests: signature verification, canonicalization, hash computation.

Integration tests: issue → verify (PASS), issue → revoke → verify (FAIL).

Smart contract tests (if enabled): deploy, issue event, revoke event.

Postman collection included to run the full flow automatically.

Include a short demo script and optional 2–4 minute demo video.

Deployment notes

Demo deployment: backend + MySQL on Render / Railway / Heroku free tiers; frontend on GitHub Pages or Netlify.

Blockchain-enabled deployment: use Hardhat + Infura/Alchemy for testnet (check current testnet availability) and minimize on-chain writes to reduce gas.

Team & deliverables

Team (authors): Shreyansh Rathaur, Rudraksh Rohilla, Akash Yadav, Aakarshan Tyagi, Shreya Sengar.

Deliverables

Source code with README & run instructions.

Backend with /issue, /verify, /revoke, /issuer/public-key.

MySQL schema + sample data; smart contract source & deployment scripts (optional).

Postman collection, test logs, demo video, and a short synopsis/report.

Contributing

Fork the repo and create a feature branch.

Run tests locally: npm test.

Submit a PR with clear description and demo steps.

For contract changes, include updated Hardhat tests.
