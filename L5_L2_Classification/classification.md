# L5 Narrow / L2 General Classification — api-oss-queue
**Platform:** Anticloud | **PAX:** 27B | **IP:** USPTO pending 2026, Anticloud FZ LLE
**Domain:** Sovereign job queue: async task scheduling for long-running Anticloud operations

## L5 Narrow
api-oss-queue specializes in sovereign job queue: async task scheduling for long-running anticloud operations within the Anticloud sovereign deployment boundary. All operations stay local — no cloud services, no external APIs, no data exfiltration. The narrow scope ensures deterministic, auditable behavior that PAX 27B can reason about precisely.

## L2 General
L2 General means api-oss-queue is available to all 9 Anticloud tiers without per-tier configuration. The same API serves hospital, defense, robotics, and research deployments.

## PAX Integration
PAX 27B is used for intelligent job prioritization: given queue depth and job descriptions, PAX recommends which jobs to process first based on urgency and resource requirements.

## AIOSS Audit Relevance
Every queue event (job ID + job type + enqueue hash + start hash + complete hash) is AIOSS-chained: H_n = SHA3-256(H_{n-1} || entry_hash_n || timestamp_n).
Tamper-evident, offline-verifiable, zero cloud dependency.

## Regulatory / Compliance
ISO/IEC 25010 (reliability), NIST SP 800-53 CP-2 (contingency planning)
