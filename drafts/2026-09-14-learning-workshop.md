# Learning workshop — 2026-09-14

## Why these concepts matter this week

**Workshop title:** From Release Signals to Safe Data and AI Workflows

**Audience:** Experienced data professionals working with Python, R, Azure, APIs, document processing, analytics, data pipelines, or AI-enabled applications.

**Prerequisites:**

- Working knowledge of Python or R and basic software dependency management.
- Familiarity with HTTP APIs, structured data, databases, and batch or service-based data pipelines.
- Basic understanding of Azure resources, deployment environments, or cloud permissions.
- No assumption that the learner already understands LLM evaluation, agent tracing, or document-AI confidence scores.

**Important source and scope assumption:** The report does not establish which Python, R, Azure, Phoenix, or Foundry versions Kienbaum currently operates. All Kienbaum exposure statements remain unknown until lockfiles, runtime images, deployed interpreters, Azure configurations, and instrumentation are inspected.

### Learning objectives

By the end of the workshop, the learner should be able to:

1. Explain why runtime security updates matter when data pipelines process untrusted archives, XML, HTTP responses, and compressed content.
2. Distinguish a declared dependency version from the interpreter and runtime image actually executing a pipeline.
3. Design a document-extraction workflow that preserves source grounding, confidence, validation status, and human-review decisions.
4. Separate syntactic validity, schema validity, and semantic correctness in structured AI outputs.
5. Design a small but useful golden evaluation set for correctness, completeness, PII handling, latency, cost, and failed runs.
6. Use traces, spans, annotations, and version metadata to diagnose multi-step AI workflow failures.
7. Design asynchronous tool workflows with bounded concurrency, timeouts, retries, cancellation, and partial-failure handling.
8. Make an adoption decision that distinguishes GA APIs, beta tooling, public-preview features, and production-ready controls.

### Prioritized agenda

| Priority | Time | Segment | Expected output |
|---|---:|---|---|
| 1 | 15 min | Release-intelligence framing | A list of report facts, unknowns, and inferences |
| 2 | 30 min | Exercise E1: Runtime exposure and hostile-input threat model | Runtime inventory matrix and input-boundary threat model |
| 3 | 35 min | Exercise E2: Grounded document extraction | Field-level extraction contract with confidence, evidence, and review routing |
| 4 | 35 min | Exercise E3: Structured output and semantic validation | Validation design for an AI-generated invoice or contract record |
| 5 | 40 min | Exercise E4: Evaluation and trace diagnosis | Golden-set outline, evaluation metrics, and trace-based failure hypothesis |
| 6 | 20 min | Exercise E5: Asynchronous R workflow design | Bounded-concurrency workflow design with failure semantics |
| 7 | 15 min | Exercise E6: Adoption decision gate | Pilot/proceed/hold decision memo |
| 8 | 10 min | Review and discussion | Open questions and proposed review-card backlog |

### Source-linked release context

| Report item | Directly supported release context | Learning signal for this workshop |
|---|---|---|
| Python 3.11.16 and 3.12.14 | Security releases address archive path traversal, XML crashes and denial-of-service behavior, HTTP header and cookie control-character injection, decompression memory-safety problems, unsafe archive extraction, and unbounded HTTP response metadata. [Python 3.12.14 release notes](https://www.python.org/downloads/release/python-31214/) · [Python 3.11.16 release notes](https://www.python.org/downloads/release/python-31116/) | Data pipelines must treat parsers, archive extractors, network responses, and runtime versions as security boundaries. |
| Azure Content Understanding Toolkit and `cu-cli` 0.1.0b2 | The preview CLI supports analyzer authoring, testing, batch extraction, confidence scores, source grounding, readiness checks, and human-review-oriented workflows. The service has a GA API version, while newer capabilities remain preview. [PyPI release](https://pypi.org/project/cu-cli/0.1.0b2/) · [Toolkit repository](https://github.com/Azure/content-understanding-toolkit) · [Azure Content Understanding updates](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/whats-new) | Document automation is not merely extraction. It requires provenance, confidence interpretation, validation, and human escalation. |
| Microsoft Foundry evaluations and Agent Monitoring Dashboard | Foundry documents cloud evaluations, Application Insights integration, recurring evaluations, live-traffic evaluation, alerts, token usage, latency, run success, evaluation scores, and red-team results. Several features are public preview. [Cloud evaluation documentation](https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/cloud-evaluation) · [Agent Monitoring Dashboard](https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/how-to-monitor-agents-dashboard) · [Foundry updates](https://learn.microsoft.com/en-us/azure/foundry/whats-new-foundry) | AI quality must be measured repeatedly after deployment, not judged only from a successful demo. |
| Arize Phoenix 20.10.0/20.11.0 and `arize-phoenix-evals` 3.8.0 | Recent releases add or expand completeness, retrieval relevance, PII, cost filtering, prompt-version metadata, annotations, error analysis, and agent-oriented trace capabilities. [Phoenix 20.11.0](https://github.com/Arize-ai/phoenix/releases/tag/arize-phoenix-v20.11.0) · [Phoenix 20.10.0](https://github.com/Arize-ai/phoenix/releases/tag/arize-phoenix-v20.10.0) · [`arize-phoenix-evals` 3.8.0](https://github.com/Arize-ai/phoenix/releases/tag/arize-phoenix-evals-v3.8.0) | Multi-step AI failures require traces and evaluator results connected to the same run, session, prompt, and tool versions. |
| R `ellmer` 0.5.0 | Current documentation describes streaming, asynchronous calls, tool/function calling, structured extraction, and multiple providers including Azure OpenAI. The report does not establish which capabilities were introduced specifically in 0.5.0. [CRAN package page](https://cran.r-project.org/package=ellmer) · [ellmer documentation](https://ellmer.tidyverse.org/) · [Streaming and async](https://ellmer.tidyverse.org/articles/streaming-async.html) · [Structured data](https://ellmer.tidyverse.org/articles/structured-data.html) | R-based AI workflows need the same contracts, validation, timeout, retry, observability, and security controls as Python services. |

### Direct report evidence versus teaching context

The release versions, documented capabilities, maturity labels, and recommended first tests above come directly from the release report. The broader explanations of trust boundaries, schema validation, evaluation design, tracing, bounded concurrency, and adoption gates are teaching context and reasonable engineering inference from those signals. They should not be interpreted as additional claims made by the release report.

### Adoption decision gates

The workshop should treat every release as a decision problem rather than an automatic upgrade or production deployment.

1. **Inventory gate:** Identify the deployed runtime, package versions, API versions, base images, Azure resources, instrumentation, and regions.
2. **Security gate:** Confirm that untrusted inputs, secrets, trace data, documents, and SAS URLs have an appropriate threat model and access policy.
3. **Controlled-pilot gate:** Use anonymized or low-risk data in an isolated environment. Pin versions and separate GA paths from preview paths.
4. **Correctness gate:** Compare results with manually labelled examples and explicit validation rules.
5. **Operational gate:** Test timeouts, retries, rate limits, partial failures, latency, cost, logging, and alerting.
6. **Human-control gate:** Define when low-confidence, unsafe, incomplete, or ambiguous results must be reviewed by a person.
7. **Promotion gate:** Promote only when evidence supports the intended use case and the residual risks are accepted by the appropriate owner.

## Learning map

```text
Runtime and input inventory
        ↓
Secure ingestion boundaries
        ↓
Document extraction with provenance and confidence
        ↓
Structured output and semantic validation
        ↓
Golden evaluation set and quality metrics
        ↓
Traces, spans, annotations, and operational diagnosis
        ↓
Bounded asynchronous execution
        ↓
Version pinning, preview isolation, and adoption gates
```

A parallel path applies to the Python security release:

```text
Declared Python version
        ↓
Actual interpreter and image inventory
        ↓
Runtime patching
        ↓
Archive/XML/HTTP/decompression regression tests
        ↓
Controlled deployment decision
```

## Concepts

The supplied relevant topic and review-card files are empty, so there is no finalized concept content to compare against in those files. The authoritative index does provide the taxonomy and filenames, but an indexed filename is not evidence that its contents have already been finalized. Accordingly, every selected item below is classified **NEW CONCEPT**. No existing entry is being rewritten or replaced.

The index contains suitable destinations for all selected concepts. No new permanent topic file is proposed.

### 1. Secure ingestion boundaries and resource-exhaustion attacks

**Classification:** **NEW CONCEPT**

**Reason:** The Python security releases expose a reusable engineering topic: data ingestion code is an attack surface. The concept is broader than any individual `tarfile`, XML, HTTP, or decompression defect.

**Release connection:** Python 3.11.16 and 3.12.14 address archive path traversal, unsafe extraction, XML denial-of-service behavior, HTTP control-character injection, decompression memory-safety issues, and unbounded HTTP response metadata. Kienbaum exposure is unknown until deployed runtimes and input paths are inventoried.

**Prerequisites:**

- Basic file, archive, XML, HTTP, and compressed-data handling.
- Basic understanding of trusted and untrusted inputs.
- Familiarity with Python services or data pipelines.

**Intuition:** A small or apparently ordinary input can cause a pipeline to perform unexpectedly large, unsafe, or unauthorized work. Security is not only about authentication; it also concerns what a parser or extractor is allowed to read, write, allocate, and process.

**Precise technical explanation:**

A secure ingestion boundary should make explicit:

- Which inputs are untrusted.
- Which formats and parsers are allowed.
- Where extracted files may be written.
- How much compressed and decompressed data may be processed.
- How many files, nesting levels, rows, or XML nodes are permitted.
- How long parsing and network reads may run.
- Which control characters, headers, cookies, URLs, and metadata are valid.
- Which errors result in rejection rather than partial processing.

Path traversal occurs when an archive member name or file path causes writes outside the intended extraction directory. Resource-exhaustion attacks occur when a small input causes excessive CPU, memory, disk, parser work, or network waiting. Runtime patching reduces exposure to known defects, but it does not replace input limits, least privilege, timeouts, and safe extraction logic.

**Realistic Kienbaum-style example:** An API accepts a ZIP file containing invoices and supporting XML files. A malicious or malformed archive contains:

- A member path that attempts to write outside the job workspace.
- Thousands of nested or duplicate files.
- Highly compressed data that expands far beyond the allowed document size.
- XML that causes excessive parser work.
- HTTP metadata that causes excessive processing or malformed downstream behavior.

A safe pipeline should extract only into an isolated workspace, enforce compressed and expanded-size limits, reject unsafe paths and links, restrict parser features, apply timeouts, and record a rejection reason without exposing sensitive content in logs.

**Why it matters:** Document and data-processing pipelines often sit at the boundary between client-controlled content and internal systems. A parser or archive defect can become a file-write, denial-of-service, data-leakage, or service-availability incident.

**Common misconception:** “The input is only a document, so it is data rather than executable risk.” Parsers and extractors execute complex logic. Untrusted data can exploit parser behavior or consume resources even when no application code is directly executed.

**Relationship to adjacent concepts:**

- **Runtime security baseline:** Patching removes known defects; boundary controls limit new or malformed inputs.
- **Validation:** Schema validation checks whether data has the expected structure; security validation also checks whether processing is safe.
- **Reliability:** Resource limits and timeouts prevent one input from exhausting shared service capacity.
- **Least privilege:** The extraction process should not have unnecessary filesystem, network, or Azure permissions.

**Recommended topic-file destination:**

- Primary: `learning/security.md`, category **secure input handling, parser safety, resource limits, and trust boundaries**.
- Supporting: `learning/data-processing.md`, category **safe ingestion and failure handling**.
- Supporting: `learning/deployment-and-infrastructure.md`, category **runtime security baseline**.

These files are supplied and empty, so the destination is confident at the taxonomy level but still requires human placement within the eventual section structure.

**Recommended review-card destination:**

- Primary: `review-cards/security.md`, category **secure ingestion and resource-exhaustion controls**.
- Supporting: `review-cards/data-processing.md`, category **defensive ingestion**.

**Compact diagram:**

```text
Untrusted file or response
        ↓
Format and size limits
        ↓
Safe parser/extractor in isolated workspace
        ↓
Validated, bounded intermediate data
        ↓
Downstream pipeline
```

**Exercise E1 contribution:** Build a threat model for a document-upload endpoint and identify at least five controls that are independent of merely upgrading Python.

---

### 2. Runtime inventory, patching, and deployment verification

**Classification:** **NEW CONCEPT**

**Reason:** The report emphasizes that upgrading application packages does not necessarily update the interpreter embedded in a container, CI runner, serverless runtime, or deployment image. This is a reusable deployment and security concept.

**Release connection:** The report recommends moving supported Python 3.11 and 3.12 deployments to 3.11.16 or 3.12.14, then testing representative APIs and document/data pipelines. Installed Kienbaum versions are unknown.

**Prerequisites:**

- Basic package manifests and lockfiles.
- Basic container or virtual-environment concepts.
- Familiarity with CI/CD or cloud deployment.

**Intuition:** The version written in a project file is not automatically the version executing production code. A pipeline can have several version authorities: `pyproject.toml`, a lockfile, a container base image, a CI runner, a serverless platform, and a deployment configuration.

**Precise technical explanation:**

A runtime security baseline requires both:

1. **Inventory:** Determine what is declared, built, deployed, and actually running.
2. **Verification:** Confirm the deployed artifact and interpreter version, then run regression tests against representative data and failure cases.

A useful inventory distinguishes:

- Application dependency versions.
- Python interpreter version.
- Base image tag and digest.
- Operating-system packages.
- CI runner image.
- Serverless runtime configuration.
- Deployment image actually used.
- Environment-specific overrides.
- Date and evidence source for each value.

The operational risk is version drift: a lockfile may be current while a production container still uses an older base image, or a deployment platform may provide an interpreter version independently of the application package environment.

**Realistic Kienbaum-style example:** A document-processing API declares Python 3.12 in its project metadata, but its container inherits an older Python base image. A CI runner uses another version, and a serverless function uses a platform-managed runtime. The team updates the package lockfile and assumes the security release is deployed. An inventory check reveals that production is still using an earlier interpreter.

The correct response is to rebuild the representative image, verify the interpreter from inside the deployed artifact, run archive/XML/HTTP/decompression regression tests, and record the evidence before promoting the image.

**Why it matters:** Security fixes only reduce production risk if the fixed runtime is actually present in the execution environment. This also improves incident response, reproducibility, and auditability.

**Common misconception:** “The lockfile is the runtime.” A lockfile can constrain packages while leaving the interpreter, base image, operating system, and platform runtime outside its control.

**Relationship to adjacent concepts:**

- **Secure ingestion:** The runtime patch is one layer of defense.
- **Deployment:** Image tags, digests, environment configuration, and release promotion determine what runs.
- **Observability:** Runtime version should be visible in service metadata or deployment evidence.
- **Version pinning:** Pinning makes an artifact reproducible; inventory verifies that the intended artifact was deployed.

**Recommended topic-file destination:**

- Primary: `learning/deployment-and-infrastructure.md`, category **runtime and image inventory, patching, and deployment verification**.
- Supporting: `learning/security.md`, category **security patch management**.

**Recommended review-card destination:**

- Primary: `review-cards/deployment-and-infrastructure.md`, category **runtime inventory and patch verification**.
- Supporting: `review-cards/security.md`, category **security release response**.

**Exercise E1 contribution:** Produce a runtime exposure matrix with columns for service, interpreter, dependency manifest, base image, CI runner, serverless configuration, evidence date, and test status.

---

### 3. Grounded document extraction, confidence, and human-in-the-loop routing

**Classification:** **NEW CONCEPT**

**Reason:** The Content Understanding release signal is not simply “use an AI extractor.” It introduces a reusable workflow pattern for turning unstructured documents into reviewable structured data while retaining evidence and uncertainty.

**Release connection:** The Content Understanding Toolkit supports structured fields, Markdown, tables, metadata, signatures, confidence scores, source grounding, custom analyzers, analyzer versions, and a Dynamic HITL component. The toolkit is beta and should be piloted with anonymized documents.

**Prerequisites:**

- Familiarity with document formats and structured records.
- Basic data-quality and validation concepts.
- Understanding that model outputs are uncertain estimates rather than authoritative facts.

**Intuition:** A useful extraction result is not just a value. It is a value plus evidence, confidence, analyzer version, validation status, and an explicit decision about whether a person must review it.

**Precise technical explanation:**

A grounded extraction record should preserve at least:

- Field name.
- Extracted value.
- Source location or evidence span.
- Confidence signal.
- Analyzer or model version.
- Validation status.
- Human-review status.
- Timestamp and document identifier.
- Reason for rejection or escalation, where applicable.

Source grounding connects an extracted field back to the document region, page, table, or text span from which it was derived. It enables a reviewer to verify the result and helps diagnose whether an error came from document layout, OCR, classification, extraction, or downstream transformation.

A confidence score is a service or model signal. It is not automatically a calibrated probability that the field is correct. Thresholds should therefore be established against labelled examples and business risk. High-value or legally sensitive fields may require review even when confidence is high.

**Realistic Kienbaum-style example:** A contract-intake workflow extracts:

- Client name.
- Contract start and end dates.
- Notice period.
- Fee amount and currency.
- Signatory names.
- Governing law.
- Missing-signature status.

For each field, the workflow stores the source page and text region, confidence, analyzer version, validation result, and review status. A low-confidence notice period or a date inconsistent with the contract header is sent to a reviewer rather than passed directly into a reporting database.

**Why it matters:** Document automation can reduce manual effort while preserving accountability. Without grounding and review routing, an apparently structured result can hide an extraction error that is difficult to detect downstream.

**Common misconception:** “A confidence score tells us whether the document is correct.” Confidence usually describes the extraction system’s signal for a particular field, not the legal, business, or semantic truth of the document.

**Relationship to adjacent concepts:**

- **Structured output validation:** Confidence and provenance complement, but do not replace, schema and business-rule checks.
- **Human-in-the-loop:** Human review should be triggered by defined conditions rather than used as an unstructured fallback.
- **Evaluation:** A labelled document set is needed to calibrate thresholds.
- **Data lineage:** Source grounding is fine-grained lineage from output back to document evidence.

**Recommended topic-file destination:**

- Primary: `learning/ai-and-llm-systems.md`, category **grounded document extraction and human-in-the-loop workflows**.
- Supporting: `learning/validation-and-data-quality.md`, category **confidence, provenance, and review status**.
- Supporting: `learning/data-processing.md`, category **document ingestion and transformation pipelines**.

**Recommended review-card destination:**

- Primary: `review-cards/ai-and-llm-systems.md`, category **grounded extraction and confidence-aware review**.
- Supporting: `review-cards/validation-and-data-quality.md`, category **field-level validation and provenance**.

**Compact diagram:**

```text
Document
   ↓
Analyzer
   ↓
Value + source evidence + confidence
   ↓
Schema and business-rule checks
   ├── accepted
   ├── corrected
   └── human review
```

**Exercise E2:** Use anonymized invoice or contract examples to define five extracted fields, label the correct values, identify source evidence, and specify which conditions trigger human review.

---

### 4. Structured output contracts and semantic validation for AI workflows

**Classification:** **NEW CONCEPT**

**Reason:** The report highlights structured extraction and tool calling in both Azure document workflows and R `ellmer`. The generalizable concept is that a machine-readable response still needs multiple layers of validation.

**Release connection:** `ellmer` documentation describes structured data extraction and tool/function calling. The report specifically warns that structured output does not guarantee semantic correctness and recommends explicit R checks.

**Prerequisites:**

- JSON or tabular data structures.
- Basic schema validation.
- Familiarity with API request and response contracts.
- Basic understanding of R or Python error handling.

**Intuition:** “The response is valid JSON” is the beginning of validation, not the end. A result can be syntactically valid, schema-valid, and still wrong for the business.

**Precise technical explanation:**

Validation should be layered:

1. **Transport validation:** Was a response received within the permitted time?
2. **Syntax validation:** Can the response be parsed?
3. **Structural validation:** Does it match the expected object, fields, types, and required values?
4. **Semantic validation:** Are dates ordered correctly, amounts plausible, currencies recognized, and identifiers consistent?
5. **Cross-record validation:** Does the result agree with related records or source systems?
6. **Policy validation:** Is the proposed tool call allowed, authorized, read-only where required, and within scope?
7. **Human validation:** Should an expert review the result before it affects a client-facing or operational system?

Tool calling adds an additional boundary: a model can propose an action, but the application must authorize and execute it. A read-only tool should be technically prevented from mutating records, not merely described as read-only in a prompt.

**Realistic Kienbaum-style example:** An R workflow asks an approved Azure OpenAI deployment to extract invoice data and call a read-only currency-reference tool. The response is checked for:

- Required fields.
- Correct types.
- ISO-like currency codes.
- Nonnegative amounts.
- Due date not earlier than invoice date.
- Currency exchange rate retrieved from an allowed source.
- No unexpected tool name or arguments.
- No write operation.

A schema-valid record with a due date before the invoice date is rejected or routed for review.

**Why it matters:** Downstream databases, reports, and client workflows often assume that structured records are reliable. Explicit validation prevents malformed or semantically wrong model responses from silently becoming authoritative data.

**Common misconception:** “Function calling guarantees safe execution.” Function calling standardizes the proposed call shape; it does not replace application authorization, argument validation, idempotency, or side-effect controls.

**Relationship to adjacent concepts:**

- **Grounded extraction:** Grounding explains where a value came from; semantic validation checks whether it makes sense.
- **Data contracts:** The schema is an interface between the AI component and downstream consumers.
- **Security:** Tool permissions and argument checks are authorization controls.
- **Observability:** Validation failures should be visible as structured events, not only free-text logs.

**Recommended topic-file destination:**

- Primary: `learning/validation-and-data-quality.md`, category **layered validation and AI data contracts**.
- Supporting: `learning/ai-and-llm-systems.md`, category **structured outputs and tool contracts**.
- Supporting: `learning/security.md`, category **tool authorization and side-effect control**.

**Recommended review-card destination:**

- Primary: `review-cards/validation-and-data-quality.md`, category **syntax, schema, and semantic validation**.
- Supporting: `review-cards/ai-and-llm-systems.md`, category **structured output and safe tool use**.

**Exercise E3:** Design a validation contract for a structured invoice or contract record. Include at least two syntax/structure checks, three semantic checks, and one tool-authorization check.

---

### 5. Golden datasets and recurring evaluation of AI systems

**Classification:** **NEW CONCEPT**

**Reason:** The Foundry and Phoenix updates point to a reusable evaluation discipline: define representative cases, measure several dimensions, calibrate against human judgement, and repeat the checks over time.

**Release connection:** Foundry documents evaluations over datasets, interactions, conversations, models, and agents, with Application Insights integration, recurring evaluations, alerts, and red-team monitoring. The report recommends a 50–100-case golden dataset. Phoenix adds retrieval relevance, completeness, PII, cost, and error-analysis capabilities.

**Prerequisites:**

- Basic testing and sampling concepts.
- Understanding of labelled examples.
- Familiarity with service-level metrics such as latency and failure rate.
- Basic awareness that evaluation metrics can disagree.

**Intuition:** A model or agent is not “good” in the abstract. It is good or bad on a defined task, dataset, population, metric, and risk threshold.

**Precise technical explanation:**

A golden dataset is a curated set of representative examples with expected outcomes or expert judgements. It should include ordinary cases and deliberately difficult cases, such as:

- Different document layouts.
- Missing or ambiguous fields.
- Multilingual or unusual terminology.
- Refusal and safety cases.
- Retrieval distractors.
- PII-containing examples.
- Tool failures and malformed responses.
- Cases where incomplete answers are more dangerous than incorrect verbosity.

Evaluation should separate dimensions that answer different questions:

- **Correctness:** Is the answer or extracted value right?
- **Completeness:** Did the workflow include required information?
- **Grounding or retrieval relevance:** Was the answer supported by the right source?
- **PII handling:** Did the workflow expose, retain, or transform sensitive data incorrectly?
- **Failed-run rate:** Did the workflow complete?
- **Latency:** How long did it take, including tail latency?
- **Token and operating cost:** What resources did it consume?
- **Safety or red-team results:** Did it fail under adversarial or prohibited inputs?

Automated evaluators should be compared with human review. A threshold suggested in vendor documentation is a starting point, not a Kienbaum-approved service level.

Recurring evaluation detects regression after changes to prompts, models, analyzers, retrieval indexes, tools, dependencies, or deployment environments.

**Realistic Kienbaum-style example:** A reporting assistant answers questions over internal project documents. The team creates 75 cases:

- 40 ordinary questions.
- 10 questions requiring refusal or clarification.
- 10 questions with retrieval distractors.
- 5 PII-sensitive cases.
- 5 incomplete-document cases.
- 5 tool-failure cases.

Each case receives a human reference answer and expected evidence. The team measures correctness, completeness, grounding, PII handling, latency, cost, and failed runs, then schedules a recurring evaluation after prompt or model changes.

**Why it matters:** A successful demonstration can conceal regressions, subgroup failures, unsupported answers, excessive cost, or safety failures. A small, well-designed evaluation set is more useful than an unexamined large test corpus.

**Common misconception:** “A single aggregate score tells us whether the agent is ready.” Aggregate scores can hide severe failures in a small but high-risk slice. Evaluation must preserve slices and failure examples.

**Relationship to adjacent concepts:**

- **Statistics and sampling:** The golden set should represent important use cases and risk slices, while recognizing that it may not estimate every production population precisely.
- **Validation:** Per-record validation checks individual outputs; evaluation measures system behavior across cases.
- **Observability:** Evaluation results explain quality; traces help explain how a failure occurred.
- **Red teaming:** Red-team cases deliberately probe safety and misuse rather than ordinary correctness.

**Recommended topic-file destination:**

- Primary: `learning/statistics-and-evaluation.md`, category **golden datasets, evaluation metrics, sampling, and human calibration**.
- Supporting: `learning/ai-and-llm-systems.md`, category **LLM and agent evaluation**.
- Supporting: `learning/validation-and-data-quality.md`, category **regression and acceptance testing**.

The statistics-and-evaluation topic file is listed in the authoritative index but was not supplied, so exact existing-section comparison is incomplete.

**Recommended review-card destination:**

- Primary: `review-cards/statistics-and-evaluation.md`, category **AI evaluation design and golden datasets**.
- Supporting: `review-cards/ai-and-llm-systems.md`, category **agent quality evaluation**.

The corresponding statistics review-card file was not supplied; this is a routing recommendation rather than a confirmed section mapping.

**Compact diagram:**

```text
Representative cases
        ↓
Human references and risk labels
        ↓
Automated evaluators + human calibration
        ↓
Quality, safety, latency, cost, and failure metrics
        ↓
Recurring evaluation and release decision
```

**Exercise E4:** Create a 50–100-case evaluation plan for an internal document or reporting agent. Define the slices, expected outcomes, metrics, review method, and one proposed alert.

---

### 6. Trace-based observability for multi-step AI workflows

**Classification:** **NEW CONCEPT**

**Reason:** The release report connects Foundry monitoring and Phoenix tracing to a common engineering problem: the final answer is often insufficient to diagnose where a multi-step workflow failed.

**Release connection:** Foundry monitoring provides operational metrics and integration with Application Insights. Phoenix adds trace/session annotations, prompt-version metadata, span-level cost filtering, completeness analysis, PII detection, and error-analysis capabilities.

**Prerequisites:**

- Basic logs, metrics, and monitoring concepts.
- Familiarity with request IDs or correlation IDs.
- Understanding of multi-step pipelines or agent workflows.

**Intuition:** A final response is the end of a causal chain. A trace shows the chain: retrieval, prompt assembly, model call, tool call, validation, retries, and final rendering.

**Precise technical explanation:**

A trace represents one end-to-end workflow execution. Spans represent meaningful steps within that execution. Useful metadata may include:

- Run, session, and parent-child identifiers.
- Prompt or configuration version.
- Model and deployment identifier.
- Tool name and outcome.
- Retrieval query and document identifiers, subject to privacy controls.
- Input and output token counts.
- Latency.
- Retry count.
- Validation result.
- Evaluator annotations.
- Error type and status.

Logs are useful for individual events, metrics summarize behavior over time, and traces preserve the causal path of one request. A production trace design must avoid casually storing raw prompts, client documents, personal data, secrets, or unrestricted tool arguments.

**Realistic Kienbaum-style example:** An agent produces an incomplete summary of a client report. The final answer alone does not explain the problem. The trace shows:

1. Retrieval returned only two of five relevant sections.
2. The prompt version omitted a completeness instruction.
3. The model completed successfully.
4. The output validator accepted the response because it checked syntax but not required sections.

The team can now distinguish a retrieval issue, prompt regression, and validation gap rather than simply recording “bad answer.”

**Why it matters:** Tracing shortens diagnosis time, supports cost and latency analysis, enables evaluator annotations, and makes release regressions explainable.

**Common misconception:** “More logs equal better observability.” Unstructured logs can increase volume without preserving relationships between steps. Observability depends on useful identifiers, dimensions, and disciplined data retention.

**Relationship to adjacent concepts:**

- **Evaluation:** Evaluators tell us that a result failed; traces help explain why.
- **Security and privacy:** Trace data may contain the same sensitive content as the original request.
- **Reliability:** Spans expose where timeouts, retries, and partial failures occur.
- **Versioning:** Prompt, model, analyzer, and tool versions must be attached to runs to make comparisons meaningful.

**Recommended topic-file destination:**

- Primary: `learning/observability-and-reliability.md`, category **traces, spans, annotations, and AI workflow diagnosis**.
- Supporting: `learning/ai-and-llm-systems.md`, category **agent observability and evaluation traces**.
- Supporting: `learning/security.md`, category **sensitive telemetry and retention**.

**Recommended review-card destination:**

- Primary: `review-cards/observability-and-reliability.md`, category **tracing multi-step workflows**.
- Supporting: `review-cards/ai-and-llm-systems.md`, category **agent traces and evaluator annotations**.

**Exercise E4 contribution:** Given a failed agent run, hypothesize whether the failure arose from retrieval, prompting, tool execution, validation, or rendering, and identify the minimum trace attributes needed to distinguish those causes.

---

### 7. Asynchronous tool workflows with bounded concurrency and explicit failure semantics

**Classification:** **NEW CONCEPT**

**Reason:** The report identifies asynchronous execution and tool calling as important parts of the R `ellmer` path. The reusable concept is not merely “use async”; it is how to control concurrency, rate limits, retries, timeouts, and partial results.

**Release connection:** `ellmer` documentation describes asynchronous calls, streaming, and tool/function calling, with integration into R’s async ecosystem through packages such as `promises`, `later`, and `coro`. Phoenix 3.8.0 also changes asynchronous rate limiting to non-blocking sleep. The report does not establish which capabilities were introduced specifically in `ellmer` 0.5.0.

**Prerequisites:**

- Basic sequential API calls.
- Familiarity with timeouts, retries, and rate limits.
- Basic understanding of asynchronous execution and futures/promises.
- Awareness that concurrent calls can exceed provider or downstream capacity.

**Intuition:** Asynchronous execution can reduce idle waiting, but unrestricted concurrency turns one slow or large workload into a rate-limit, cost, memory, or reliability problem.

**Precise technical explanation:**

A bounded asynchronous workflow specifies:

- Maximum concurrent operations.
- Per-operation and overall deadlines.
- Queue behavior when capacity is full.
- Cancellation behavior.
- Which failures are retryable.
- Retry count and backoff.
- Whether operations are idempotent.
- How partial results are recorded.
- How rate limits and provider quotas are respected.
- How completed, failed, rejected, and cancelled items are distinguished.

Concurrency is not the same as parallel CPU execution. For network-bound LLM or document calls, asynchronous concurrency can allow other work to proceed while one request waits, but the provider and downstream systems still impose capacity constraints.

**Realistic Kienbaum-style example:** An R workflow processes 100 anonymized reports through an approved Azure OpenAI deployment. It allows at most five in-flight requests, applies a 60-second per-request timeout, retries transient 429 or 5xx responses with bounded backoff, does not retry schema-validation failures automatically, and writes a durable result ledger for completed, failed, and review-required documents.

A read-only tool call may be retried if it is safe and idempotent. A mutation such as creating or updating a client record should require stronger authorization and idempotency controls.

**Why it matters:** Async workflows affect latency, provider cost, quota consumption, memory use, and failure recovery. Without explicit bounds, a proof of concept may behave unpredictably under realistic batch size.

**Common misconception:** “Async makes the workflow faster without trade-offs.” Async often improves throughput or responsiveness, but it can increase concurrency pressure, reorder completion, complicate cancellation, and make partial failure more difficult to reason about.

**Relationship to adjacent concepts:**

- **Distributed systems:** Retries, idempotency, backoff, and partial failure apply even when the workflow is implemented in one application.
- **Observability:** Each concurrent operation needs a correlation identifier and terminal status.
- **Validation:** A successful network response may still fail schema or semantic validation.
- **Cost control:** Concurrency limits help control token and provider costs but do not replace budget monitoring.

**Recommended topic-file destination:**

- Primary: `learning/concurrency-and-parallelism.md`, category **asynchronous I/O, bounded concurrency, and cancellation**.
- Supporting: `learning/distributed-systems.md`, category **retries, idempotency, and partial failure**.
- Supporting: `learning/ai-and-llm-systems.md`, category **async model and tool workflows**.

The concurrency and distributed-systems topic contents were not supplied, so comparison and exact section mapping are incomplete.

**Recommended review-card destination:**

- Primary: `review-cards/concurrency-and-parallelism.md`, category **bounded async workflows**.
- Supporting: `review-cards/distributed-systems.md`, category **retry and idempotency semantics**.
- Supporting: `review-cards/ai-and-llm-systems.md`, category **async tool calling**.

The corresponding files were not supplied, so these are routing recommendations.

**Exercise E5:** Design an R workflow for 100 documents with a concurrency limit of five. Specify timeout, retry, rate-limit, cancellation, validation, and partial-result behavior.

---

### 8. Version pinning, GA/preview separation, and maturity-aware adoption

**Classification:** **NEW CONCEPT**

**Reason:** The report repeatedly distinguishes security releases, GA APIs, beta tooling, public-preview features, and rapidly changing observability packages. The reusable concept is controlled adoption based on maturity and evidence.

**Release connection:**

- `cu-cli` 0.1.0b2 is explicitly beta.
- Content Understanding has a GA API version, while newer capabilities are preview.
- Foundry recurring evaluation, continuous evaluation, red-team, and monitoring features include public-preview elements.
- Phoenix has rapid consecutive releases and a deprecated evaluator migration path.
- The report recommends pinning tested versions and isolating preview API use.

**Prerequisites:**

- Basic deployment environments and version control.
- Familiarity with API versioning and dependency pinning.
- Understanding of pilot, staging, and production environments.

**Intuition:** A feature can be useful without being stable enough for production. “Available” does not mean “safe to standardize.”

**Precise technical explanation:**

Maturity-aware adoption separates:

- Package version.
- API version.
- Feature maturity.
- Support and service-level expectations.
- Data sensitivity.
- Reversibility of the change.
- Observability and rollback capability.
- Migration cost if the preview contract changes.

A controlled pilot should pin package and API versions, use a separate environment or resource where appropriate, isolate preview features from the GA path, document assumptions, and define a rollback or migration test. A production decision should be based on workload-specific evidence rather than release momentum alone.

**Realistic Kienbaum-style example:** A document team tests `cu-cli` beta features against anonymized contracts while production uses the GA Content Understanding API. The pilot records analyzer version, API version, confidence behavior, grounding quality, cost, and human-review rate. A promotion decision is deferred if a preview-only feature is essential but lacks adequate regression coverage or rollback options.

Similarly, Phoenix versions are pinned for a sandbox comparison with Foundry. The team records the deprecated document-relevance evaluator and creates a migration test for retrieval relevance before upgrading.

**Why it matters:** Preview dependencies can change behavior, schemas, permissions, pricing, regional availability, or support status. Isolation protects production workflows while allowing learning.

**Common misconception:** “Pinning makes a preview feature production-ready.” Pinning improves reproducibility; it does not create a production SLA, eliminate privacy risk, or guarantee future support.

**Relationship to adjacent concepts:**

- **Deployment:** Environment separation and rollback are deployment controls.
- **Evaluation:** A promotion gate requires evidence from representative data.
- **Security:** Preview telemetry, SAS URLs, prompts, and traces may contain sensitive material.
- **Architecture:** A replaceable adapter reduces the cost of moving away from a preview dependency.

**Recommended topic-file destination:**

- Primary: `learning/deployment-and-infrastructure.md`, category **version pinning, API versioning, environment isolation, and release gates**.
- Supporting: `learning/software-architecture.md`, category **replaceable integrations and dependency boundaries**.
- Supporting: `learning/ai-and-llm-systems.md`, category **AI service maturity and preview management**.

The software-architecture topic contents were not supplied, so exact comparison is incomplete.

**Recommended review-card destination:**

- Primary: `review-cards/deployment-and-infrastructure.md`, category **GA/preview separation and adoption gates**.
- Supporting: `review-cards/software-architecture.md`, category **replaceable provider integrations**.
- Supporting: `review-cards/ai-and-llm-systems.md`, category **AI feature maturity**.

**Exercise E6:** Write a one-page decision memo recommending pilot, hold, or proceed for one of the release items. The memo must state evidence, unknowns, data sensitivity, version/API pinning, rollback plan, and promotion criteria.

## Proposed permanent-resource changes

The following are proposals only. No permanent resource has been modified.

### Proposed index changes

Add cross-references to the authoritative learning-resource index:

- `security.md`
  - Secure ingestion boundaries.
  - Parser and archive resource limits.
  - Runtime security patching.
  - AI tool authorization and sensitive telemetry.
- `deployment-and-infrastructure.md`
  - Runtime and image inventory.
  - Interpreter verification.
  - Version and API pinning.
  - GA/preview isolation and adoption gates.
- `ai-and-llm-systems.md`
  - Grounded document extraction.
  - Confidence-aware human review.
  - Structured outputs and tool contracts.
  - Golden datasets and recurring AI evaluation.
  - Agent tracing and operational monitoring.
  - Asynchronous AI workflows.
- `validation-and-data-quality.md`
  - Provenance and confidence.
  - Layered syntax, schema, semantic, and cross-record validation.
  - AI regression testing.
- `observability-and-reliability.md`
  - Traces, spans, annotations, evaluator linkage, and cost/latency diagnosis.
- `data-processing.md`
  - Defensive document ingestion and bounded transformations.
- `statistics-and-evaluation.md`
  - Golden-set design, risk slices, human calibration, and metric interpretation.
- `concurrency-and-parallelism.md`
  - Async I/O, bounded concurrency, cancellation, and backpressure.
- `distributed-systems.md`
  - Retry classification, idempotency, rate limits, and partial failure.
- `software-architecture.md`
  - Replaceable provider adapters and dependency boundaries.

No new topic file is recommended because every selected concept fits an existing indexed category and the index explicitly states that no suitable topic file is missing.

### Proposed topic-file additions

Eventually add concise, independently reviewable entries rather than one long release-specific article:

1. **Security**
   - Secure ingestion as a trust boundary.
   - Archive traversal and resource-exhaustion controls.
2. **Deployment and infrastructure**
   - Declared versus deployed runtime inventory.
   - Patch verification and regression evidence.
   - GA/preview separation and reproducible adoption.
3. **AI and LLM systems**
   - Grounded extraction and human-in-the-loop routing.
   - Structured model outputs and tool contracts.
   - Golden evaluations and trace-based diagnosis.
4. **Validation and data quality**
   - Provenance-aware field validation.
   - Syntax, schema, semantic, and policy validation.
5. **Observability and reliability**
   - Trace/span design for multi-step AI workflows.
   - Linking traces with evaluator results and versions.
6. **Statistics and evaluation**
   - Representative golden sets, risk slices, and human calibration.
7. **Concurrency and distributed systems**
   - Bounded async execution, retries, idempotency, and partial results.

All additions should be written as new entries. Existing finalized entries, if discovered later, should remain stable; only genuinely additive material should be appended after human review.

### Proposed review-card changes

Propose cards in the corresponding taxonomy files for:

- Secure ingestion and resource exhaustion.
- Runtime inventory and security patch verification.
- Grounded extraction and confidence-aware review.
- Structured outputs and semantic validation.
- Golden datasets and recurring evaluation.
- Trace-based AI observability.
- Bounded asynchronous tool workflows.
- GA/preview separation and adoption gates.

The supplied review-card files for security, deployment, observability, validation, data processing, and AI are empty. The review-card files for statistics and evaluation, concurrency, distributed systems, and software architecture are indexed but were not supplied. Therefore, exact comparison with prior cards is incomplete for those destinations.

## Proposed review cards

All cards below are proposals for human review. Unless later materials establish a different learning date, each proposed card uses **First learned date: 2026-09-14**.

### Card 1 — Secure ingestion boundaries

- **Recall:** What makes an archive, XML document, compressed payload, or HTTP response an untrusted input?
- **Mental hook:** “Bound the work before trusting the data.”
- **Why I care:** Data pipelines can suffer path traversal, denial of service, memory exhaustion, or malformed-header failures before business validation begins.
- **Contrast:** Runtime patching addresses known implementation defects; input limits and isolation constrain what any input is allowed to make the system do.
- **Tiny example:** Reject an archive if a member resolves outside the extraction root or if decompressed size exceeds the pipeline limit.
- **Test yourself:** List three controls that still matter after upgrading Python.
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/security.md`, secure ingestion and resource-exhaustion section; supporting destination `review-cards/data-processing.md`.

### Card 2 — Runtime inventory and patch verification

- **Recall:** Why can a current lockfile coexist with an outdated production interpreter?
- **Mental hook:** “Declared is not deployed.”
- **Why I care:** A security release only reduces risk when the fixed interpreter and image are actually running.
- **Contrast:** Dependency management constrains packages; deployment verification confirms the executing runtime.
- **Tiny example:** Inspect the Python version from inside the deployed container rather than inferring it from `pyproject.toml`.
- **Test yourself:** Name five places where a Python version may be controlled or recorded.
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md`, runtime inventory and patch verification; supporting destination `review-cards/security.md`.

### Card 3 — Grounded extraction and confidence-aware review

- **Recall:** What should accompany an extracted document field besides its value?
- **Mental hook:** “A value without evidence is an unreviewable guess.”
- **Why I care:** Client documents require traceability, correction, and human escalation when extraction is uncertain.
- **Contrast:** Confidence is a system signal; source grounding is evidence location; neither independently proves business correctness.
- **Tiny example:** Store `notice_period = 3 months` with page, text span, confidence, analyzer version, and review status.
- **Test yourself:** Which fields in a contract would you always review manually, regardless of confidence?
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/ai-and-llm-systems.md`, grounded extraction; supporting destination `review-cards/validation-and-data-quality.md`.

### Card 4 — Structured output and semantic validation

- **Recall:** What are the differences between syntax, schema, semantic, and policy validation?
- **Mental hook:** “Parseable is not plausible, and plausible is not authorized.”
- **Why I care:** AI-generated records can silently contaminate databases and reports even when their JSON structure is valid.
- **Contrast:** A schema checks shape; semantic validation checks meaning; authorization checks whether a proposed tool action is permitted.
- **Tiny example:** Reject an invoice where the due date precedes the invoice date, even if every field has the correct type.
- **Test yourself:** Write one semantic rule and one tool-authorization rule for a read-only reporting assistant.
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/validation-and-data-quality.md`, layered validation; supporting destination `review-cards/ai-and-llM-systems.md`.

### Card 5 — Golden datasets and recurring AI evaluation

- **Recall:** What makes a golden dataset useful?
- **Mental hook:** “Measure the cases you would regret getting wrong.”
- **Why I care:** A demo can pass while high-risk document layouts, refusal cases, PII cases, or retrieval failures regress.
- **Contrast:** A per-record validation rule checks one result; an evaluation set measures system behavior across representative cases.
- **Tiny example:** A 75-case set includes ordinary questions, retrieval distractors, PII cases, incomplete documents, and tool failures.
- **Test yourself:** Which metric would reveal incomplete answers that exact-match accuracy might miss?
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/statistics-and-evaluation.md`, AI evaluation design; supporting destination `review-cards/ai-and-llM-systems.md`. Exact existing-card comparison is incomplete because the statistics review-card file was not supplied.

### Card 6 — Trace-based AI observability

- **Recall:** What does a trace add beyond a final response and a log line?
- **Mental hook:** “Follow the causal path, not just the outcome.”
- **Why I care:** Retrieval, prompt assembly, tool use, retries, validation, cost, and latency can all contribute to one bad answer.
- **Contrast:** Metrics show population behavior; logs show events; traces connect the steps of one execution.
- **Tiny example:** A trace links a missing report section to a retrieval result, prompt version, model call, and validator outcome.
- **Test yourself:** Name five attributes you would attach to an agent trace without storing unrestricted raw client content.
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/observability-and-reliability.md`, AI traces and diagnosis; supporting destination `review-cards/ai-and-llM-systems.md`.

### Card 7 — Bounded asynchronous tool workflows

- **Recall:** What controls are needed when processing many documents asynchronously?
- **Mental hook:** “Async removes waiting; bounds prevent overload.”
- **Why I care:** Unbounded concurrency can cause rate limits, runaway cost, memory pressure, and difficult partial failures.
- **Contrast:** Concurrency controls how much work is in flight; retries determine what happens after failure; idempotency determines whether repeating work is safe.
- **Tiny example:** Process 100 reports with five concurrent requests, 60-second timeouts, bounded retries for 429/5xx responses, and a durable result ledger.
- **Test yourself:** Which failures should not be retried automatically?
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/concurrency-and-parallelism.md`, bounded async workflows; supporting destinations `review-cards/distributed-systems.md` and `review-cards/ai-and-llM-systems.md`. Exact existing-card comparison is incomplete because those files were not supplied.

### Card 8 — GA/preview separation and adoption gates

- **Recall:** Why does version pinning not make a preview feature production-ready?
- **Mental hook:** “Reproducible risk is still risk.”
- **Why I care:** Beta and preview features may change behavior, contracts, permissions, pricing, or support status.
- **Contrast:** Pinning controls reproducibility; maturity assessment controls whether the capability is appropriate for the environment.
- **Tiny example:** Test `cu-cli` beta capabilities with anonymized documents while production remains on the GA API version.
- **Test yourself:** What evidence would you require before promoting a preview-only feature?
- **First learned date:** 2026-09-14
- **Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md`, maturity-aware adoption; supporting destinations `review-cards/ai-and-llM-systems.md` and `review-cards/software-architecture.md`. Exact comparison is incomplete where the supplied card files are absent.

### Exercise and review-card routing summary

| Item | Exercise | Intended permanent topic-file destination |
|---|---|---|
| Secure ingestion | E1 | `security.md`; supporting `data-processing.md` |
| Runtime inventory | E1 | `deployment-and-infrastructure.md`; supporting `security.md` |
| Grounded extraction | E2 | `ai-and-llM-systems.md`; supporting `validation-and-data-quality.md` |
| Structured validation | E3 | `validation-and-data-quality.md`; supporting `ai-and-llM-systems.md` |
| Golden evaluation | E4 | `statistics-and-evaluation.md`; supporting `ai-and-llM-systems.md` |
| Trace diagnosis | E4 | `observability-and-reliability.md`; supporting `ai-and-llM-systems.md` |
| Async workflow | E5 | `concurrency-and-parallelism.md`; supporting `distributed-systems.md` |
| Adoption gate | E6 | `deployment-and-infrastructure.md`; supporting `software-architecture.md` |
| Review cards 1–2 | Cards 1–2 | `security.md` and `deployment-and-infrastructure.md` |
| Review cards 3–5 | Cards 3–5 | `ai-and-llM-systems.md`, `validation-and-data-quality.md`, and `statistics-and-evaluation.md` |
| Review cards 6–8 | Cards 6–8 | `observability-and-reliability.md`, `concurrency-and-parallelism.md`, `distributed-systems.md`, and `deployment-and-infrastructure.md` |

## Questions worth discussing

1. Which Kienbaum document or data workflow would provide the best first pilot while keeping client and personal data exposure low?
2. Which document fields should always require human review, even when the extraction confidence is high?
3. How should correctness, completeness, grounding, PII handling, latency, and cost be weighted for a client-facing assistant?
4. What minimum trace attributes would help diagnose failures without storing raw sensitive prompts and documents?
5. Which AI tool calls can safely be retried, and which require idempotency keys or explicit human authorization?
6. When should a preview capability be isolated for learning, and when should it be rejected because the migration or rollback cost is too high?
7. How should Foundry and Phoenix findings be compared when their evaluators, trace schemas, retention policies, and operating costs differ?

## Safety and permanence

This is a **DRAFT** for discussion with a human Learning Coach.

No permanent learning-resource index, topic file, or review-card file has been modified. The proposed classifications, destinations, topic additions, and review cards require human review before they are added to the Data Engineering Learning Resource.

The supplied files contain no finalized concept entries in the provided AI, security, deployment, observability, validation, data-processing, or review-card destinations. Several indexed destinations, including statistics and evaluation, concurrency and parallelism, distributed systems, and software architecture, were not supplied; comparison with their existing contents is therefore incomplete. No claim is made that those files are empty or that any proposed concept is absent from them.