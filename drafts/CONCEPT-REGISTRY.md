# Cross-workshop concept registry

This registry is a **deduplication index for draft candidates**, not permanent learning material and not a record of what the learner has mastered. The permanent knowledge base remains `learning/`; workshop status remains in dated files under `workshop-progress/`.

## Audit seed

The initial seed covers seven dated workshop drafts and 56 numbered candidate objects. The source drafts remain unchanged. The registry maps those candidates to normalized concept IDs so future drafts can distinguish a new learning objective from a new release example.

Four high-confidence repeated placements share a canonical ID:

1. `RUNTIME-BINARY-COMPATIBILITY` — 2026-09-11 #1 and 2026-10-05 #8.
2. `DATA-OUTPUT-REGRESSION` — 2026-09-11 #2 and 2026-10-05 #5. The 2026-09-11 object is broader; the repeated subtopic is stale trailing output after a shorter replacement write.
3. `ORM-RELATIONSHIP-FILTERS` — 2026-10-05 #3 and 2026-11-02 #2.
4. `AI-DOCUMENT-EXTRACTION-EVALUATION` — 2026-10-12 #5 and 2026-10-26 #3.

These are four repeated placements, not four instructions to delete or rewrite historical drafts. Preserve unique details as examples or extensions under the canonical concept.

## Candidate index

`Source object` uses the numbered concept in the dated draft. `Relationship` is about cross-draft concept identity only; it is not a learning-progress status.

| Source object | Normalized candidate concept | Canonical ID | Relationship |
|---|---|---|---|
| [2026-09-11 #1](2026-09-11-learning-workshop.md) | Dependency, runtime, and binary compatibility | `RUNTIME-BINARY-COMPATIBILITY` | Shared canonical concept |
| [2026-09-11 #2](2026-09-11-learning-workshop.md) | Regression testing for data correctness and destructive output writes | `DATA-OUTPUT-REGRESSION` | Broader object; includes the stale-output subtopic |
| [2026-09-11 #3](2026-09-11-learning-workshop.md) | Observability for parallel efficiency | `PARALLEL-EFFICIENCY-OBSERVABILITY` | Distinct candidate |
| [2026-09-11 #4](2026-09-11-learning-workshop.md) | Contract-first document-to-structured-data pipelines | `DOCUMENT-TO-STRUCTURED-DATA` | Distinct candidate |
| [2026-09-11 #5](2026-09-11-learning-workshop.md) | Reproducible, configuration-driven analyzer workflows | `REPRODUCIBLE-ANALYZER-WORKFLOWS` | Distinct candidate |
| [2026-09-11 #6](2026-09-11-learning-workshop.md) | Evaluation as a quality gate for AI systems | `AI-EVALUATION-GATES` | Related to later evaluation candidates; not automatically merged |
| [2026-09-11 #7](2026-09-11-learning-workshop.md) | Observability for AI and agent operations | `AI-AGENT-OBSERVABILITY` | Related to later telemetry candidates; compare objective before merging |
| [2026-09-11 #8](2026-09-11-learning-workshop.md) | Durable workflows and human-in-the-loop control | `DURABLE-WORKFLOWS-APPROVALS` | Distinct candidate |
| [2026-09-11 #9](2026-09-11-learning-workshop.md) | Data minimization and PII controls in AI pipelines | `AI-DATA-MINIMIZATION-PII` | Related security family; distinct control focus |
| [2026-09-14 #1](2026-09-14-learning-workshop.md) | Secure ingestion boundaries and resource-exhaustion controls | `SECURE-INGESTION-RESOURCE-BOUNDS` | Distinct candidate |
| [2026-09-14 #2](2026-09-14-learning-workshop.md) | Runtime inventory, patching, and deployment verification | `RUNTIME-INVENTORY-PATCH-VERIFICATION` | Related to compatibility/adoption; distinct inventory objective |
| [2026-09-14 #3](2026-09-14-learning-workshop.md) | Grounded document extraction, confidence, and human review | `GROUNDED-DOCUMENT-EXTRACTION` | Related to extraction evaluation; distinct provenance/review objective |
| [2026-09-14 #4](2026-09-14-learning-workshop.md) | Structured-output contracts and semantic validation | `STRUCTURED-OUTPUT-SEMANTIC-VALIDATION` | Related to typed outputs; compare validation objective before merging |
| [2026-09-14 #5](2026-09-14-learning-workshop.md) | Golden datasets and recurring AI evaluation | `GOLDEN-SET-AI-EVALUATION` | Related evaluation candidate; do not assume duplicate |
| [2026-09-14 #6](2026-09-14-learning-workshop.md) | Trace-based diagnosis of multi-step AI workflows | `AI-TRACE-DIAGNOSIS` | Related to AI observability; distinct trace-diagnosis objective |
| [2026-09-14 #7](2026-09-14-learning-workshop.md) | Bounded asynchronous tool workflows and failure handling | `BOUNDED-ASYNC-TOOL-WORKFLOWS` | Related to async I/O and batch jobs; distinct application |
| [2026-09-14 #8](2026-09-14-learning-workshop.md) | Version pinning, preview isolation, and maturity-aware adoption | `PREVIEW-MATURITY-ADOPTION-GATES` | Related to upgrade controls; distinct maturity objective |
| [2026-10-05 #1](2026-10-05-learning-workshop.md) | Parameterized SQL and SQL-injection prevention | `PARAMETERIZED-SQL` | Distinct security mechanism |
| [2026-10-05 #2](2026-10-05-learning-workshop.md) | TLS negotiation and fail-closed security configuration | `TLS-FAIL-CLOSED-CONFIGURATION` | Distinct security mechanism |
| [2026-10-05 #3](2026-10-05-learning-workshop.md) | ORM relationship filters across loading strategies | `ORM-RELATIONSHIP-FILTERS` | Shared canonical concept |
| [2026-10-05 #4](2026-10-05-learning-workshop.md) | DataFrame semantic checks at parsing, dtype, and memory boundaries | `DATAFRAME-SEMANTIC-CHECKS` | Distinct candidate |
| [2026-10-05 #5](2026-10-05-learning-workshop.md) | Worksheet replacement and stale trailing rows | `DATA-OUTPUT-REGRESSION` | Shared canonical concept; stale-output subtopic |
| [2026-10-05 #6](2026-10-05-learning-workshop.md) | Async I/O signaling and pooled-resource lifecycle | `ASYNC-IO-POOL-LIFECYCLE` | Related to bounded async work; distinct liveness mechanism |
| [2026-10-05 #7](2026-10-05-learning-workshop.md) | Parallel-worker runtime and environment assumptions | `PARALLEL-WORKER-ENVIRONMENT` | Related to parallel efficiency; distinct execution-context objective |
| [2026-10-05 #8](2026-10-05-learning-workshop.md) | Native package and runtime compatibility | `RUNTIME-BINARY-COMPATIBILITY` | Shared canonical concept |
| [2026-10-05 #9](2026-10-05-learning-workshop.md) | Optional telemetry failure and graceful startup | `TELEMETRY-DEGRADED-STARTUP` | Related observability family; distinct startup-policy objective |
| [2026-10-12 #1](2026-10-12-learning-workshop.md) | Meaning-preserving coercion and validation order | `MEANING-PRESERVING-COERCION` | Distinct candidate |
| [2026-10-12 #2](2026-10-12-learning-workshop.md) | Egress policy as a data-boundary control | `EGRESS-POLICY-BOUNDARY` | Related to data handling; distinct network control |
| [2026-10-12 #3](2026-10-12-learning-workshop.md) | AI tool discovery versus authorization | `AI-TOOL-ACCESS-AUTHORIZATION` | Related to agent permissions; distinct access-layer objective |
| [2026-10-12 #4](2026-10-12-learning-workshop.md) | Reproducibility across independently versioned layers | `LAYERED-EXPERIMENT-REPRODUCIBILITY` | Related to analyzer reproducibility; broader version tuple |
| [2026-10-12 #5](2026-10-12-learning-workshop.md) | Ground-truth evaluation of AI document extraction | `AI-DOCUMENT-EXTRACTION-EVALUATION` | Shared canonical concept |
| [2026-10-12 #6](2026-10-12-learning-workshop.md) | Provenance, claim correctness, and request telemetry | `AI-EVIDENCE-AND-REQUEST-TELEMETRY` | Related to AI observability; distinct evidence/correctness objective |
| [2026-10-26 #1](2026-10-26-learning-workshop.md) | Keyed reconciliation with tolerances and CI gating | `KEYED-DATA-RECONCILIATION` | Distinct data-comparison mechanism |
| [2026-10-26 #2](2026-10-26-learning-workshop.md) | Asynchronous batch jobs and service limits | `ASYNC-BATCH-JOBS-SERVICE-LIMITS` | Related to async workflows; distinct job lifecycle/limits objective |
| [2026-10-26 #3](2026-10-26-learning-workshop.md) | Ground-truth evaluation of AI extraction | `AI-DOCUMENT-EXTRACTION-EVALUATION` | Shared canonical concept |
| [2026-10-26 #4](2026-10-26-learning-workshop.md) | Controlled upgrades, support lifecycles, and shadow migration | `CONTROLLED-UPGRADES-SHADOW-MIGRATION` | Related to upgrade safety; distinct migration/rollback objective |
| [2026-10-26 #5](2026-10-26-learning-workshop.md) | Instrumenting LLM calls for tokens, latency, and cost | `LLM-CALL-COST-OBSERVABILITY` | Related to AI telemetry; distinct per-call cost objective |
| [2026-10-26 #6](2026-10-26-learning-workshop.md) | Data-handling boundaries for third-party AI services | `THIRD-PARTY-AI-DATA-HANDLING` | Related security family; distinct governance objective |
| [2026-11-02 #1](2026-11-02-learning-workshop.md) | Bound parameters and expanding `IN` lists | `EXPANDING-IN-PARAMETERS` | Related to parameterized SQL; distinct library mechanism |
| [2026-11-02 #2](2026-11-02-learning-workshop.md) | Relationship loading strategies and join conditions | `ORM-RELATIONSHIP-FILTERS` | Shared canonical concept |
| [2026-11-02 #3](2026-11-02-learning-workshop.md) | ORM session state versus database state | `ORM-SESSION-DATABASE-STATE` | Distinct database concept |
| [2026-11-02 #4](2026-11-02-learning-workshop.md) | Presence semantics in request inputs | `REQUEST-PRESENCE-SEMANTICS` | Distinct API contract concept |
| [2026-11-02 #5](2026-11-02-learning-workshop.md) | Validation timing for defaults | `DEFAULT-VALIDATION-TIMING` | Distinct validation mechanism |
| [2026-11-02 #6](2026-11-02-learning-workshop.md) | Open versus closed schemas and unknown keys | `SCHEMA-EXTRA-KEY-POLICY` | Distinct validation/security mechanism |
| [2026-11-02 #7](2026-11-02-learning-workshop.md) | Numeric and value domains, precision, range, and parsing | `NUMERIC-VALUE-DOMAINS` | Distinct validation concept |
| [2026-11-02 #8](2026-11-02-learning-workshop.md) | Security-advisory triage by range, code path, and mode | `SECURITY-ADVISORY-TRIAGE` | Distinct security process |
| [2026-11-02 #9](2026-11-02-learning-workshop.md) | Join cardinality and duplicate keys | `JOIN-CARDINALITY` | Distinct data-correctness mechanism |
| [2026-11-02 #10](2026-11-02-learning-workshop.md) | Dependency pinning, version ranges, and upgrade gates | `DEPENDENCY-UPGRADE-GATES` | Related to upgrade safety; distinct dependency-control objective |
| [2026-11-09 #1](2026-11-09-learning-workshop.md) | Least-privilege credentials, expiry, and rotation | `CREDENTIAL-LEAST-PRIVILEGE-ROTATION` | Distinct security mechanism |
| [2026-11-09 #2](2026-11-09-learning-workshop.md) | Supply-chain integrity, provenance, lockfiles, and pinning | `SOFTWARE-SUPPLY-CHAIN-INTEGRITY` | Related to dependency management; distinct provenance objective |
| [2026-11-09 #3](2026-11-09-learning-workshop.md) | Breaking changes, compatibility contracts, and regression tests | `BREAKING-CHANGE-REGRESSION-TESTING` | Related to upgrade and data-correctness families; distinct compatibility objective |
| [2026-11-09 #4](2026-11-09-learning-workshop.md) | Text encoding and the bytes/text boundary | `TEXT-ENCODING-BOUNDARY` | Distinct data-ingestion concept |
| [2026-11-09 #5](2026-11-09-learning-workshop.md) | Out-of-core execution and memory budgets | `OUT-OF-CORE-MEMORY-BUDGETS` | Distinct data-processing concept |
| [2026-11-09 #6](2026-11-09-learning-workshop.md) | Typed structured outputs from models | `TYPED-STRUCTURED-MODEL-OUTPUTS` | Related to structured-output validation; distinct output-typing objective |
| [2026-11-09 #7](2026-11-09-learning-workshop.md) | Threshold selection and labeled-set evaluation | `THRESHOLD-LABELED-SET-EVALUATION` | Related AI-evaluation family; distinct statistical decision objective |
| [2026-11-09 #8](2026-11-09-learning-workshop.md) | Untrusted content and approval gates for tool-using agents | `AGENT-UNTRUSTED-CONTENT-APPROVAL` | Related to tool authorization; distinct prompt-injection control objective |

## Recurring families to compare carefully

The following families recur across drafts. Recurrence is a prompt to compare, **not** an instruction to merge all members:

| Family | Seen in | Deduplication caution |
|---|---|---|
| Release/upgrade safety | All seven drafts | Separate runtime inventory, binary compatibility, API maturity, dependency provenance, shadow migration, and compatibility testing where the mechanism differs. |
| Data correctness and validation | All seven drafts | Separate coercion, ORM state, row identity, join cardinality, encoding, schemas, and output replacement; reuse the shared validation/regression mental model. |
| Security and trust boundaries | All seven drafts | Do not collapse SQL injection, TLS, credentials, parser safety, egress, third-party data governance, and prompt injection. |
| AI evaluation | 2026-09-11, 2026-09-14, 2026-10-12, 2026-10-26, 2026-11-09 | Reuse golden-set foundations; keep extraction metrics, confidence calibration, and threshold/cost decisions distinct when they add a new method. |
| Observability and telemetry | 2026-09-11, 2026-09-14, 2026-10-05, 2026-10-12, 2026-10-26 | Distinguish parallel-efficiency measurement, trace diagnosis, startup degradation, evidence/correctness, and per-call cost. |
| Async, concurrency, and workflows | 2026-09-11, 2026-09-14, 2026-10-05, 2026-10-26 | Distinguish process workers, async I/O signaling, bounded tool concurrency, and service-side batch-job state. |

## Maintenance rules

- Add a row for every learning-object candidate in each new workshop draft, with a stable canonical ID.
- Reuse an ID only when the learning objective and mechanism are substantially the same. For meaningful additions, create a separate ID and cross-reference the concept it extends.
- Add new release notes and examples to the existing candidate rather than duplicating the entire lesson when they only illustrate the same concept.
- Keep historical drafts intact. This registry is for comparison and future consolidation, not retroactive status changes.
- Do not store learning progress here. Use dated workshop-progress files for that.
