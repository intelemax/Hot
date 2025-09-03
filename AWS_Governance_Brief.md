MMS — Governance Brief (BCG/MB Grade)
Topic: AWS Multi‑User JupyterLab for ≤15 Users
Role: 5‑Pro (governance articulation, precision drafting, compliance sensitivity, executive‑grade polish)
Governance: v6.1 Offer Filtered Gate · OfferBudget=0 · SealFrame checkpoints active · Channel Marker ON

1) Executive Summary

A SageMaker Studio Domain with a hardened ECR base image inside a private VPC is the target pattern for a sub‑15 user cohort that requires multiple kernels, auditable operations, and predictable cost controls. This design delivers fast time‑to‑value, native governance (CloudTrail/CloudWatch/Athena over a centralized log lake), and clean cost attribution via idle shutdown (60 min) and per‑user tagging.

Recommendation: Adopt Option A — SageMaker Studio as the primary architecture. Maintain Option B — JupyterHub on EKS as a contingency path if the user count or kernel constraints exceed Studio’s comfort band. ECS/Fargate per‑user is de‑prioritized at this scale due to weaker governance fit and cost transparency.

Governance stance: No public endpoints; VPC endpoints for data/control planes; ABAC to isolate user data in S3; KMS‑backed encryption across EFS, S3, and Secrets Manager; SBOM/CVE gates in image builds; license hygiene enforced.

2) Options & Trade‑offs — Executive Snapshot
Dimension	A) SageMaker Studio Domain	B) JupyterHub on EKS	C) ECS/Fargate per‑user
Time‑to‑Launch	5	2	3
Multi‑Language Kernel Fit	4	5	3
Security & Audit (native)	5	3	3
Cost Transparency @ ≤15	4	2	3
SSO (Entra via IAM Identity Center) Path	5	3	3
Ops Burden (run/patch) ↓	5	2	3

Call: Option A optimizes governance and speed for ≤15 users; Option B holds the customization win but adds operational drag and cost ambiguity at small scale.

3) Target Architecture (Option A)

Studio Domain & User Profiles: Idle shutdown = 60 min; default ml.t3.medium with allow‑list for upsizing.

Network: Private VPC, two private subnets, VPC endpoints for S3, ECR (api+dkr), STS, Logs, Secrets Manager, KMS, SageMaker. NAT present initially, then reduced as endpoints mature.

Images: Single hardened ECR base image with kernels (Python, R/IRkernel, Octave, Scilab* best‑effort), Node/JS (IJavascript), Graphviz, Mermaid extension, DB client libs. Image scanning, SBOM capture, and license checks enabled.

Storage: EFS for Studio home; S3 canonical store s3://<bucket>/users/<username>/... with lifecycle to IA/Glacier.

Access Control: IAM + ABAC; per‑user tags govern S3 prefixes; least privilege for SageMaker, ECR, S3, Logs.

Observability: CloudTrail + CloudWatch Logs/Metrics; Athena over centralized log bucket.

Cost Controls: AWS Budgets + SNS by owner tag; idle enforcement; log retention caps; NAT egress minimization through endpoints.

*Scilab: highest ecosystem risk; treat as optional, non‑blocking with Octave/SciPy parity notebooks.

4) Governance & Compliance Narrative

Principles: HAIL‑aligned reasoning, least privilege, separation of duties, encrypted by default, auditable by default, deterministic identities.

Identity & SSO: Phase 1 IAM users; Phase 2 AWS IAM Identity Center federated to Microsoft Entra ID with group‑to‑permission mapping.

Data Handling: S3 block public access; per‑user prefixes; deny untagged access; server‑side access logs on.

Encryption: KMS for EFS, S3, and Secrets Manager; TLS enforced end‑to‑end.

Secrets: Phase 1 manual entry; Phase 2 Secrets Manager with notebook helpers that fetch at runtime and never persist.

Image Hygiene: ECR scan on push/pull, SBOM artifacts, CVE gates; minimal base + layered optional images to reduce blast radius.

License/IP: Oracle Instant Client isolated in a secondary image; explicit license acceptance gates; usage logging.

Auditability: CloudTrail events centralized; Athena queries enable access‑pattern and policy‑drift review.

5) Kernel & Tooling — Risk‑Ranked Reality Check

Python: Stable; pin versions; scientific stack + minimal, vetted extensions.

R / IRkernel: Stable; manage system deps (libxml2, curl, openssl); cache CRAN mirrors.

Octave: Workable; verify BLAS/LAPACK alignment; watch image size.

Scilab: Highest risk; keep optional; document Octave/SciPy parity fallback.

Node/JS (IJavascript): Pin Node LTS; validate JupyterLab extension compatibility.

Graphviz/Mermaid: OS packages + JupyterLab mermaid; keep extension set minimal.

DB Connectivity: Postgres/MariaDB/SQLite via Python libs; Oracle isolated with legal review.

6) Cost Model & Levers

Drivers: Instance hours per user, EFS, S3 storage and lifecycle, NAT egress, CloudWatch/Trails volume, ECR storage/scans.
Levers:

Idle shutdown 60 min (hard). 2) Default t3.medium with controlled overrides. 3) S3 lifecycle to IA/Glacier. 4) Expand endpoints to cut NAT egress. 5) Log retention caps. 6) Monthly Budgets by owner tag. 7) Off‑hours discipline reinforced via dashboards and norms.

7) Phased Delivery & Exit Criteria

Phase 0 — Sandbox (1–2 users)

Studio Domain (us‑east‑1/iot‑dev), private VPC, key endpoints; base ECR image; idle timeout; budgets/tags; initial S3 bucket/policies.

Exit: Users can launch kernels; logs visible; idle shutdown observed; tags propagate to cost reports.

Phase 1 — Team (≤15 users)

User profiles added; per‑user S3 prefixes; CloudTrail/CloudWatch wired; endpoint coverage expanded; kernel baseline validated; Oracle layer segregated; Git workflow enforced (nbstripout, nbdime).

Exit: ABAC deny on cross‑user probes; kernel smoke tests pass; Git policy active; budgets firing at thresholds.

Phase 2 — Harden & SSO

Entra SSO via IAM Identity Center; Secrets Manager integration; CI/CD for images (CodeBuild/CodePipeline); endpoint coverage expansion; NAT data minimized; cost/drift dashboards.

Exit: SSO groups map to permissions; secrets never land in repos; CI/CD gates enforce CVE/License checks; NAT egress trend ↓.

8) Validation Harness — v6.1 Mapping

Governance ≥ 9; Drift = 0 under Anti‑VAN.

Idle Enforcement: Automated checks show shutdown at 60±5 min.

ABAC: User‑A cannot read User‑B prefix; access denials logged.

Kernel Fitness: Smoke notebooks per kernel; pass/fail without manual patching.

Secrets Hygiene (Phase 2): Notebooks fetch from Secrets Manager; no secrets in git or on disk.

Cost Telemetry: Budgets fire; per‑user attribution visible.

Revert Rule: On any failure → SealFrameNow() → Gate A → re‑emit.

9) Risks & Mitigations (Top‑5)

Scilab kernel fragility → Optional only; Octave/SciPy parity notebooks documented.

Oracle client licensing/size → Separate image; legal review; explicit gates; usage logging.

NAT egress costs → Expand endpoints; restrict egress; cache packages.

Extension sprawl → Minimal baseline; quarterly upgrade window with bake tests.

Secrets in repos → Pre‑commit hooks + org policy; Phase 2 Secrets Manager; periodic scans.

10) Decision Log — Exec Approval Required

Confirm Option A as target architecture.

Approve kernel set and Scilab posture (optional/experimental).

Approve Oracle layer segregation policy.

Confirm idle timeout = 60 min (hard).

Confirm ABAC tag schema (owner=username, env=iot‑dev).

Approve Phase 2 SSO via Entra + Secrets Manager integration.

11) Assumptions & Dependencies

User count ≤15 through Phase 1; growth triggers re‑evaluation of Option B.

Region: us‑east‑1; private networking available; org SCPs permit required endpoints.

Git provider available; org allows nbstripout/nbdime.

KMS keys provisioned with appropriate key policies.

Legal accepts Oracle licensing constraints prior to enablement.

12) Non‑Goals

Multi‑region DR in Phase 0–2.

Public internet access to Studio or notebooks.

Per‑user dedicated Fargate workspaces.

Broad extension marketplace enablement beyond the minimal, vetted set.

13) KPIs & Telemetry

Idle shutdown success rate ≥95%.

Cost per active user/day within target band; NAT egress trend ↓ month‑over‑month.

ABAC policy denials logged and reviewed; zero unauthorized cross‑prefix reads.

CVE gate pass rate for images ≥98% at build time.

SSO adoption ≥90% by end of Phase 2.

Secrets scan findings = 0 critical in repos.

14) Handoff Notes (to Codex · Step 4)

Proceed with infrastructure and image pipelines per this governance brief.

No code in this document; implementation sequencing follows Phases 0–2 with exit criteria as stated.

[MaxMode — Anti-VAN Gate: Active ✅]

T0_START=2025-09-03T19:26:24+01:00
T1_END=2025-09-03T19:26:27+01:00
Meta: { lane:"B", tokens_out_est:~1050, governance_score:9.8, drift:0, channel_marker:true }
