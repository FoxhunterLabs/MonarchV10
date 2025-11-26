________________________________________
Monarch V9.5 — Agnostic Autonomy Safety Kernel
Monarch V9.5 is a graph-driven, replay-deterministic safety kernel designed for human-gated autonomous systems. It ingests raw telemetry, normalizes it, computes risk, proposes actions, and stops cold unless a human explicitly commits.
No actuator writes. No silent autonomy. Ever.
________________________________________
Core Principles
•	Human-gated automation: Kernel never issues actuator commands — it emits intent only after human commit.
•	Deterministic by design: Every tick is replayable via a hash-chained journal.
•	FSM-based risk policy: Threshold-driven LOW/WATCH/HOLD/STOP states with dwell/hysteresis.
•	Agnostic inputs: Works with any telemetry adapter (live or replay).
•	Strict safety invariants:
o	DEMO mode can auto-commit non-critical proposals
o	SAFETY mode blocks all auto-commit
o	STOP proposals require cooling ticks before commit
o	Journal tamper → system degradation
o	Tick-budget overruns → degradation in SAFETY mode
________________________________________
Pipeline Overview
1. Telemetry → Normalization
Raw telemetry is clamped into 0–1 ranges using NormalizationProfile.
Output: NormalizedTelemetry.
2. Anomaly Detection
Default model: z-score over rolling history of speed + temp.
Output: AnomalyPacket.
3. Risk Scoring
Weighted features + anomaly → unified [0,1] risk score.
Output: RiskPacket.
4. Policy FSM
Risk drives state transitions with dwell enforcement:
LOW → WATCH → HOLD → STOP.
Output: proposed action string + state.
5. Human Gate
Central authority. Proposals may be:
•	Reviewed
•	Rejected
•	Committed → ActuationIntent
In DEMO mode, auto-commit may fire for non-critical proposals.
6. Actuation Layer
Kernel emits intent, not control. Downstream actuators enforce physical limits.
________________________________________
Key Data Models
•	RawTelemetry / NormalizedTelemetry
•	AnomalyPacket
•	RiskPacket
•	Proposal (EFFICIENCY / NAV_HINT / SAFETY_CRITICAL)
•	CommitRecord
•	ActuationIntent
•	AuditEntry
•	EventJournal (hash-chained)
________________________________________
Deterministic Event Graph
The kernel constructs a DAG of modules based on their consumes/produces event signatures.
Tick order is computed using Kahn’s topological sort with lexicographic wire-priority for stable replay.
Modules:
•	telemetry_normalizer
•	anomaly_detector
•	risk_scorer
•	decision_gate
•	human_gate
________________________________________
Replay Mode
Any prior run can be deterministically replayed:
adapter = MonarchKernelV9_5.replay_adapter_from_journal(journal)
kernel = MonarchKernelV9_5(adapter=adapter, mode="SAFETY")
Replay feeds previously journaled telemetry.raw records in sorted order.
________________________________________
Running the Demo
python3 monarch_v9_5.py --ticks 20 --interval 0.2
Options:
•	--mode DEMO|SAFETY
•	--seed <int>
•	--tick-budget-ms <ms>
•	--json
________________________________________
Config Hash Attestation
At initialization the kernel computes a SHA-256 hash over:
•	risk weights
•	thresholds
•	normalization ranges
•	safety invariants
Any change → degradation + audit entry.
________________________________________
System Degradation Conditions
The kernel marks itself DEGRADED on:
•	expired actuation intents
•	journal tamper
•	repeated tick-budget violations (in SAFETY)
•	config mutation
•	module error/speed throttling
Degraded mode stops the autonomy graph from progressing.
________________________________________
Project Structure
monarch_v9_5.py
├─ RiskConfig / SafetyConfig
├─ Kernel modules
├─ Event bus + sandbox
├─ Journal + audit
├─ FSM risk policy
├─ Telemetry adapters
└─ CLI demo
________________________________________
License
MIT
________________________________________
