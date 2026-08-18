OLA Evidence

Evidence Layer for OLA di-OS

OLA Evidence is the evidence and provenance layer of OLA di-OS.

Its purpose is to transform operational observations, files, events and verification results into structured, traceable and independently verifiable evidence.

The core principle is:

«No important decision without traceable evidence.»

---

1. Purpose

OLA Evidence provides a common mechanism for:

- collecting evidence
- normalizing evidence
- calculating content hashes
- recording provenance
- timestamping observations
- linking evidence to decisions
- verifying evidence integrity
- exporting evidence for audits and reports

It is designed to separate:

FACT
  ↓
EVIDENCE
  ↓
VERIFICATION
  ↓
DECISION

from:

ASSUMPTION
  ↓
MODEL INTERPRETATION

This distinction is fundamental to OLA.

---

2. Evidence object

The minimum conceptual evidence object is:

{
  "id": "evidence-001",
  "sourceType": "system",
  "contentHash": "sha256:...",
  "capturedAt": "2026-08-18T09:00:00Z",
  "owner": "system",
  "verification": "PASS"
}

Additional metadata may include:

source
sourceUri
contentType
size
createdAt
capturedAt
collector
tenantId
missionId
decisionId
policyIds
signature
verificationStatus

The implementation should keep the core evidence schema stable while allowing controlled extension.

---

3. Evidence lifecycle

COLLECT
   ↓
NORMALIZE
   ↓
HASH
   ↓
STORE
   ↓
LINK
   ↓
VERIFY
   ↓
USE
   ↓
EXPORT

Collect

Acquire evidence from an authorized source.

Examples:

- uploaded files
- API responses
- system state
- logs
- configuration
- command results
- monitoring data
- agent execution output

Normalize

Convert the observation into a deterministic representation where practical.

Hash

Calculate a cryptographic content hash.

Example:

SHA-256(content)

The hash identifies the exact content that was evaluated.

Store

Persist the evidence and its metadata using an appropriate storage backend.

Link

Associate evidence with:

- missions
- operations
- policies
- decisions
- reports

Verify

Check that the evidence has not changed and that its provenance satisfies the applicable rules.

Export

Make evidence available for:

- audit
- investigation
- reporting
- customer review
- independent verification

---

4. Evidence states

OLA Evidence should distinguish between the existence of evidence and the conclusion derived from evidence.

Recommended verification states:

PASS
FAIL
UNKNOWN

PASS

Evidence satisfies the verification rule.

FAIL

Evidence exists but violates the verification rule.

UNKNOWN

The available evidence is insufficient to determine the result.

"UNKNOWN" is not equivalent to "PASS".

It must never silently become a successful result.

---

5. Integrity

Evidence integrity is based primarily on cryptographic hashing.

Example:

content
   │
   ▼
SHA-256
   │
   ▼
contentHash

If the content changes:

original content
      ≠
modified content

then:

hash(original)
      ≠
hash(modified)

This allows the system to detect content modification.

Hashing provides integrity evidence.

It does not, by itself, prove:

- who created the content
- that the source was trustworthy
- that the observation was truthful
- that the capture process was authorized

Those properties require additional provenance controls.

---

6. Provenance

Evidence should answer:

WHERE did it come from?
WHEN was it captured?
WHO or WHAT collected it?
WHAT exactly was collected?
HOW was it processed?
WHICH decision used it?

A provenance chain may therefore look like:

Source
  ↓
Collector
  ↓
Evidence
  ↓
Verification
  ↓
Decision
  ↓
Report

---

7. Evidence vs. conclusion

OLA must not confuse evidence with interpretation.

Example:

Evidence:
certificate_expiry = 2026-08-20

Possible interpretation:

The certificate will expire soon.

Decision:

RISK = HIGH

These are three different objects.

The evidence should remain independently inspectable.

The interpretation should be traceable to the evidence.

The decision should identify the rule or reasoning that produced it.

---

8. Verification model

A verification result can be represented conceptually as:

{
  "ruleId": "CERT-EXPIRY-001",
  "version": "1.0",
  "status": "FAIL",
  "reason": "Certificate expires within the configured threshold"
}

This makes the result reproducible and auditable.

Rules should be versioned.

A historical decision should retain the rule version used to produce it.

---

9. Evidence graph

Evidence should be linkable rather than existing as isolated files.

Example:

Mission
  │
  ├── Evidence A
  │      └── Policy P1
  │
  ├── Evidence B
  │      └── Policy P2
  │
  └── Evidence C
         └── Policy P3
                │
                ▼
             Decision
                │
                ▼
              Report

This allows an auditor or customer to move backwards:

REPORT
  ↓
DECISION
  ↓
POLICY
  ↓
EVIDENCE
  ↓
SOURCE

---

10. Immutability

Where evidence is used for audit or forensic purposes, the preferred model is append-oriented storage.

Instead of:

Evidence A
   ↓
EDIT
   ↓
Evidence A'

prefer:

Evidence A
   ↓
New observation / correction
   ↓
Evidence B

Historical evidence should not silently disappear.

If correction is necessary, the system should preserve the relationship between the original and the corrected record.

---

11. Cryptographic signatures

Hashing and signing solve different problems.

Hash

Provides evidence that content has changed.

content → hash

Signature

Can provide evidence that a particular signing key authorized the record.

record + private key
        ↓
     signature

OLA may support signed evidence records using an asymmetric signature scheme such as Ed25519.

Signature verification should be separate from ordinary content hashing.

---

12. Security requirements

OLA Evidence should follow:

- least privilege
- explicit source authorization
- tenant isolation
- secure secret management
- input validation
- controlled serialization
- integrity verification
- audit logging
- restricted evidence access
- no plaintext secrets in evidence unless explicitly required and protected

Evidence itself may contain sensitive information.

Therefore:

«Evidence integrity does not remove the need for evidence confidentiality.»

---

13. API concept

A minimal implementation may expose operations equivalent to:

POST   /evidence
GET    /evidence/{id}
POST   /evidence/{id}/verify
GET    /missions/{id}/evidence
GET    /decisions/{id}/evidence
GET    /reports/{id}/evidence

The exact API should follow the current runtime architecture rather than being duplicated unnecessarily.

---

14. Example workflow

Input

A customer uploads a configuration file.

customer-config.yaml

Collection

OLA creates an evidence record.

evidence_id = ev_001

Hash

SHA-256(customer-config.yaml)

Analysis

A policy evaluates the configuration.

RULE-CONFIG-001

Verification

status = FAIL

Decision

risk = HIGH

Report

The final report contains a reference to:

decision
  ↓
rule
  ↓
evidence
  ↓
contentHash

The customer can therefore inspect the basis of the decision.

---

15. Minimal implementation

A minimal Evidence implementation should initially provide:

Evidence
├── schema
├── creation
├── hashing
├── storage
├── retrieval
├── verification
├── provenance
└── tests

Do not introduce distributed infrastructure until the product actually requires it.

A reliable local implementation with deterministic tests is preferable to a complex evidence platform that cannot be verified end-to-end.

---

16. Testing

Evidence testing should cover at least:

Schema

- required fields
- valid types
- invalid records

Hashing

- deterministic hashes
- modified content detection
- empty content handling

Provenance

- source association
- timestamp handling
- mission association
- decision association

Verification

- PASS
- FAIL
- UNKNOWN
- rule versioning

Integrity

- original content verifies
- modified content fails verification

Security

- unauthorized access denied
- cross-tenant access denied
- secrets not exposed

End-to-end

Create evidence
      ↓
Hash
      ↓
Store
      ↓
Retrieve
      ↓
Verify
      ↓
Use in decision
      ↓
Generate report

---

17. Definition of Done

OLA Evidence is not considered production-ready merely because an evidence table or class exists.

Minimum Definition of Done:

[ ] Evidence schema implemented
[ ] Deterministic hashing implemented
[ ] Evidence persistence implemented
[ ] Retrieval implemented
[ ] Verification implemented
[ ] Provenance links implemented
[ ] Integrity tests passing
[ ] Security tests passing
[ ] E2E workflow passing
[ ] Documentation matches actual implementation

For production claims, there must also be deployment evidence.

---

18. Non-goals

OLA Evidence is not intended to:

- automatically declare every source trustworthy
- turn model output into facts
- replace enterprise SIEM/SOAR systems
- provide legal certification by itself
- guarantee truth merely because data is hashed
- store unrestricted sensitive information
- become an unnecessarily complex distributed storage system

---

19. Design principle

The Evidence layer should remain boring, deterministic and inspectable.

The AI layer can be probabilistic.

The evidence layer should be substantially less so.

AI:
"What do I think happened?"

OLA Evidence:
"What can we actually demonstrate?"

OLA Verification:
"Does the evidence satisfy the rule?"

OLA Decision:
"What should we do based on that verified state?"

---

20. Relationship to OLA di-OS

OLA Evidence is a foundational subsystem of OLA di-OS.

OLA di-OS
│
├── Runtime
├── Evidence
│   ├── Collection
│   ├── Provenance
│   ├── Integrity
│   └── Verification
├── Policy
├── Decision
├── Reporting
└── Product / Dashboard

Evidence should remain reusable across the rest of the platform.

---

21. Current-state rule

This README describes the intended contract and behavior of OLA Evidence.

It must not be interpreted as proof that every capability described here is currently implemented.

Implementation status must be established from:

source code
+
tests
+
CI results
+
runtime evidence
+
deployment evidence

If implementation and documentation disagree, the actual verified implementation wins and the README must be corrected.

---

22. Final principle

OLA Evidence exists to establish a simple chain:

SOURCE
  ↓
EVIDENCE
  ↓
HASH / PROVENANCE
  ↓
VERIFICATION
  ↓
DECISION
  ↓
REPORT

The objective is not to make AI appear trustworthy.

The objective is to make important AI-assisted operations more inspectable, reproducible and accountable.

Evidence first. Claims second.# ola-evidence-gateway