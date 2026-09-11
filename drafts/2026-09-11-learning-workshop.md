# Learning workshop — 2026-09-11

**Status: DRAFT for discussion with a Learning Coach.** No permanent learning-resource, topic, index, or review-card files have been created or modified.

## Why these concepts matter this week

This week’s report combines two kinds of engineering risk:

- **Silent or catastrophic correctness failures:** a pandas wheel regression could cause process crashes, while openxlsx defects could produce incorrect dates or stale cells in Excel deliverables.
- **Compatibility and release-management risk:** Pydantic’s Python 3.15 support is available in a beta release, making isolated compatibility testing more appropriate than broad production deployment.
- **Performance uncertainty:** `future` journaling can reveal whether parallel jobs are actually using workers effectively or merely adding coordination overhead.
- **A maturing document-to-AI pipeline:** Azure Content Understanding, `cu-cli`, Microsoft Foundry evaluations, agent monitoring, and long-running hosted agents introduce concepts spanning extraction, contracts, reproducibility, quality gates, observability, security, and durable workflows.

The learning goal is therefore not to memorize package-specific release notes. It is to understand the reusable engineering concepts behind them:

1. dependency and runtime compatibility;
2. regression testing for data correctness;
3. observability of parallel work;
4. contract-first document-to-structured-data pipelines;
5. reproducible, configuration-driven analyzer workflows;
6. evaluation as an AI quality gate;
7. observability for AI and agent operations;
8. durable workflows with human approval;
9. data minimization and PII controls for AI pipelines.

### Comparison with existing learning materials

All supplied relevant topic files and review-card files are currently empty. Consequently:

- every selected concept is classified as **NEW CONCEPT**;
- no selected concept is classified as **EXTEND EXISTING CONCEPT**;
- no selected concept is classified as **NO UPDATE NEEDED**;
- there are no finalized entries to rewrite or replace.

The authoritative index lists `statistics-and-evaluation.md` and `distributed-systems.md`, but those corresponding topic and review-card files were not among the supplied files. Exact comparison against those two destinations is therefore incomplete. The supplied taxonomy still provides suitable primary destinations for the selected concepts, so no new topic file is required.

## Learning map

```text
Dependency graph, runtime, and binary compatibility
                    ↓
        Compatibility CI and release gates
                    ↓
      Reproducible deployment and configuration
                    ↓
     Regression tests for data and API contracts
                    ↓
       Reliable document-to-data pipelines
                    ↓
AI evaluation gates ───────────────→ AI observability
        ↓                                  ↓
Release or rollback decision       Operational feedback
                    ↓
       Durable workflows and approvals
                    ↓
         Secure handling of documents,
            prompts, outputs, and PII
```

A parallel-processing track runs alongside the main path:

```text
Parallel execution model
        ↓
Workers, chunks, serialization, and overhead
        ↓
Parallel-efficiency diagnostics
        ↓
Tuning and evidence-based scaling decisions
```

## Concepts

### 1. Dependency, runtime, and binary compatibility

#### Classification and reason

**NEW CONCEPT**

The supplied deployment, architecture, and validation files are empty, so there is no existing finalized concept to extend. This is a broadly reusable idea behind both the pandas regression and the Pydantic Python 3.15 compatibility release.

#### Prerequisites

- Python environments and package managers.
- Direct versus transitive dependencies.
- Lockfiles and dependency constraints.
- Basic CI concepts.
- The distinction between a language runtime, a package, and a compiled binary dependency.

#### Release signal — directly supported by the report

- pandas 3.0.5 fixes a regression in pandas 3.0.4 wheels that could cause segmentation faults on Python 3.14 and in datetime-related code paths.
- The report attributes the stated cause to an incompatible NumPy build dependency.
- Pydantic 2.14.0b2 adds support for Python 3.15-specific features, but it is a beta release.
- The recommended actions are to inspect lockfiles, rebuild images, test compatibility, and isolate Python 3.15 testing rather than broadly deploying the beta.

#### Intuition

A data application is not just “Python plus pandas.” It is a stack:

```text
Operating system and CPU
          ↓
Python interpreter
          ↓
Compiled numerical libraries
          ↓
Python packages
          ↓
Application code
          ↓
Data and deployment configuration
```

A package can be logically compatible with an application while still being incompatible with a particular interpreter, operating system, CPU architecture, or compiled dependency.

#### Precise technical explanation

Compatibility is a relationship among several dimensions:

- interpreter version, such as Python 3.14 or 3.15;
- package version;
- transitive dependency versions;
- platform and architecture;
- binary or ABI compatibility;
- build tools and wheel availability;
- application-level assumptions about APIs and data types.

A lockfile constrains the dependency graph, but it does not by itself prove that every combination is safe. A package may resolve successfully and still fail during import, datetime processing, serialization, or another runtime path.

A responsible compatibility gate usually includes:

1. an explicit environment matrix;
2. controlled dependency resolution;
3. clean image or environment rebuilds;
4. import and smoke tests;
5. representative data-path tests;
6. a clear policy for stable, beta, and preview dependencies.

For example, a Python 3.15 compatibility lane can be useful without making Python 3.15 the production runtime.

#### Realistic data-engineering example

A consulting ETL image contains:

```text
Python 3.14
pandas
NumPy
SQLAlchemy
timezone-aware datetime transformations
Parquet serialization
```

The team should:

1. inventory the image and lockfile;
2. upgrade pandas to 3.0.5 or a later approved version;
3. rebuild the image rather than modifying a running container;
4. test import, datetime parsing, timezone conversion, resampling, and serialization;
5. run a representative ETL job;
6. record the environment identifier alongside the test result.

Separately, the team can add a non-production CI job using Python 3.15 and `pydantic==2.14.0b2` to test schemas, JSON serialization, validation errors, and API contracts.

#### Why it matters

Dependency failures can occur before business logic runs, and they can be difficult to reproduce if environments are rebuilt informally. Compatibility testing also prevents a common release mistake: treating support announced in a beta or preview release as equivalent to production readiness.

#### Common misconception

> “If the resolver installed the packages successfully, the environment is compatible.”

Resolution proves that a dependency graph can be constructed. It does not prove that imports, compiled code, datetime paths, serialization, or representative workloads behave correctly.

#### Relationship to adjacent concepts

- **Deployment and infrastructure:** environment images, CI matrices, promotion gates.
- **Software architecture:** dependency boundaries and upgrade isolation.
- **Validation and data quality:** compatibility smoke tests and regression suites.
- **Observability and reliability:** detecting failures after deployment and measuring rollback impact.

#### Compact diagram

```text
Package release
      ↓
Dependency resolver
      ↓
Environment build
      ↓
Compatibility tests
      ↓
Promotion decision
```

#### Recommended topic-file destination

- **Primary:** `learning/deployment-and-infrastructure.md`
  - Proposed category: **Runtime compatibility, dependency locking, and release gates**.
- **Supporting:** `learning/software-architecture.md`
  - Proposed category: **Dependency boundaries and upgrade impact**.
- **Rationale:** the operational decision is about environments and release gates, while dependency relationships are also an architectural concern.

#### Recommended review-card destination

- **Primary:** `review-cards/deployment-and-infrastructure.md`
  - Proposed category: **Runtime and dependency compatibility**.
- **Uncertainty:** the file is supplied but empty, so the category is a recommended destination rather than an existing section.

---

### 2. Regression testing for data correctness and destructive write boundaries

#### Classification and reason

**NEW CONCEPT**

The report shows two different failure modes that are easy to miss with ordinary “did the job finish?” testing: incorrect date interpretation and stale cells remaining after a smaller output replaces a larger one.

#### Prerequisites

- Assertions and test fixtures.
- Expected versus actual outputs.
- Data types, schemas, and boundary conditions.
- Basic file or table comparison techniques.
- The difference between a process failure and a data-correctness failure.

#### Release signal — directly supported by the report

- openxlsx 4.2.9 fixes date detection.
- It adds overwrite support to `writeData()` so stale cells can be cleared when rewriting with a smaller data range.
- The report recommends workbook comparison tests.
- pandas 3.0.5 requires tests for import, datetime, resampling, timezone, serialization, and representative ETL paths.

#### Intuition

A pipeline can complete with exit code zero and still produce a wrong report.

There are at least three distinct test questions:

```text
Did the process run?
        ↓
Did it produce an output?
        ↓
Is the output correct, complete, and free of stale data?
```

The third question is the one most likely to be missed.

#### Precise technical explanation

Regression testing protects against a previously observed or reasonably foreseeable failure returning after a change.

For data-producing systems, useful regression assertions can cover:

- exact values for high-risk fields;
- date interpretation and timezone behavior;
- column names and types;
- row counts and key uniqueness;
- expected blanks;
- absence of obsolete rows or cells;
- output shape after a dataset shrinks;
- serialization and round-trip behavior.

A shrinking-output test is especially important for report generation. Suppose a previous run writes 100 rows and the next run should write only 80. A correct overwrite must ensure that rows 81–100 do not remain visible in the workbook.

Golden-file comparisons can help, but they should distinguish meaningful data changes from irrelevant metadata such as workbook calculation settings or generated timestamps.

#### Realistic data-engineering example

A monthly report pipeline writes an Excel sheet containing invoice data.

Test fixtures include:

- a normal dataset with dates in the expected format;
- a dataset with an empty optional field;
- a smaller second dataset;
- a timezone-sensitive timestamp;
- an expected workbook or normalized sheet representation.

The tests verify that:

1. dates are interpreted as dates rather than arbitrary numbers or strings;
2. the expected number of rows is present;
3. cells outside the shortened range are blank;
4. formulas and intentionally preserved formatting behave as designed;
5. a pandas transformation does not change timezone or resampling semantics unexpectedly.

#### Why it matters

Excel reports are often consumed manually and may not have a downstream schema validator. A stale cell or misinterpreted date can therefore become a business decision error rather than a visible system failure.

#### Common misconception

> “A snapshot test of the whole workbook is always the best answer.”

A raw binary snapshot may be noisy and brittle. The useful test target is the business-relevant representation: values, types, dimensions, required formatting, and absence of obsolete content.

#### Relationship to adjacent concepts

- **Validation and data quality:** assertions, contracts, and correctness criteria.
- **Data processing:** transformations, serialization, and output boundaries.
- **Deployment:** regression gates before a package upgrade is promoted.
- **Observability:** detecting unexpected output changes after release.

#### Compact diagram

```text
Input fixture
     ↓
Transformation or workbook write
     ↓
Normalized output
     ↓
Values + types + shape + absence checks
     ↓
Pass/fail release decision
```

#### Recommended topic-file destination

- **Primary:** `learning/validation-and-data-quality.md`
  - Proposed category: **Regression testing for data-producing systems**.
- **Supporting:** `learning/data-processing.md`
  - Proposed category: **Output boundaries, serialization, and stale-data prevention**.
- **Rationale:** the central idea is correctness testing, with data-processing examples showing why output boundaries matter.

#### Recommended review-card destination

- **Primary:** `review-cards/validation-and-data-quality.md`
  - Proposed category: **Regression tests for values, types, shape, and absence**.
- **Uncertainty:** the supplied file has no existing section structure, so this is a proposed category only.

---

### 3. Observability for parallel efficiency

#### Classification and reason

**NEW CONCEPT**

The `future` release provides a concrete signal for a general performance concept: parallel execution must be measured, not assumed to be beneficial.

#### Prerequisites

- Processes, workers, tasks, and futures.
- Sequential versus parallel execution.
- Wall-clock time and throughput.
- Serialization and coordination overhead.
- Basic metrics interpretation.

#### Release signal — directly supported by the report

With `options(future.journal = TRUE)`, `future` 1.75.0 reports parallelization-efficiency information. The report identifies possible explanations such as round-trip overhead, poor worker utilization, or workloads that do not benefit from parallel execution. It recommends comparing worker utilization and elapsed time while tuning chunks, backends, and worker counts.

#### Intuition

Parallelism is a trade:

```text
Potential compute speedup
        -
Coordination, startup, data-transfer, and serialization cost
        =
Actual benefit
```

Adding workers increases capacity, but it can also increase contention and overhead.

#### Precise technical explanation

A parallel job should be evaluated using both outcome and execution evidence.

Useful measurements include:

- total elapsed time;
- task execution time;
- worker utilization;
- time spent waiting;
- data-transfer or serialization cost;
- memory pressure;
- task size distribution;
- throughput per worker;
- failure and retry behavior.

A job is a poor candidate for parallelization when tasks are too small, data movement dominates computation, or the backend introduces more coordination than useful work.

A simple conceptual model is:

$$
T_{\text{parallel}} \approx T_{\text{compute}}/N + T_{\text{coordination}} + T_{\text{transfer}} + T_{\text{startup}}
$$

The idealized division by $$N$$ is not a promise. The other terms may dominate.

#### Realistic data-engineering example

An R pipeline calculates customer-level features for a large portfolio. The team tests:

- sequential execution;
- four workers;
- eight workers;
- different chunk sizes;
- different future backends.

For each run, it records:

- elapsed time;
- memory use;
- worker utilization;
- number of tasks;
- serialization time if available;
- output equivalence.

The team might discover that eight workers are slower than four because each task repeatedly transfers a large lookup table.

#### Why it matters

Without efficiency telemetry, teams can mistake higher worker counts for better engineering. This can increase cloud cost and operational complexity while making the workload slower.

#### Common misconception

> “If CPU usage is high, the parallel design is efficient.”

High CPU usage can coexist with excessive serialization, uneven task sizes, memory contention, or poor end-to-end throughput. Efficiency is measured against useful completed work and elapsed time.

#### Relationship to adjacent concepts

- **Concurrency and parallelism:** execution models and worker coordination.
- **Observability and reliability:** metrics that explain execution behavior.
- **Data processing:** partitioning, chunking, and movement of intermediate data.
- **Deployment:** resource sizing and cost control.

#### Compact diagram

```text
Workload
  ↓
Partition into tasks
  ↓
Schedule to workers
  ↓
Compute + transfer + synchronize
  ↓
Measure elapsed time and utilization
  ↓
Tune chunking, backend, and worker count
```

#### Recommended topic-file destination

- **Primary:** `learning/concurrency-and-parallelism.md`
  - Proposed category: **Parallel-efficiency diagnostics and overhead**.
- **Supporting:** `learning/observability-and-reliability.md`
  - Proposed category: **Performance telemetry for concurrent jobs**.
- **Rationale:** the execution model belongs in concurrency, while the measurement discipline belongs in observability.

#### Recommended review-card destination

- **Primary:** `review-cards/concurrency-and-parallelism.md`
  - Proposed category: **Measure parallel benefit instead of assuming it**.
- **Supporting option:** `review-cards/observability-and-reliability.md` for a separate performance-observability card.
- **Uncertainty:** both supplied files are empty, so there is no existing section to preserve.

---

### 4. Contract-first document-to-structured-data pipelines

#### Classification and reason

**NEW CONCEPT**

Azure Content Understanding’s reported capabilities provide a useful entry point into a reusable pipeline pattern: documents must be converted into structured, validated data through explicit contracts rather than treated as undifferentiated text.

#### Prerequisites

- APIs and request/response contracts.
- Schemas and structured data.
- Basic document-processing concepts.
- Classification, extraction, and validation.
- Awareness of synchronous versus asynchronous service calls.

#### Release signal — directly supported by the report

Azure Content Understanding 2.0 preview adds or improves:

- synchronous Read and Layout APIs;
- contextualization;
- semantic chunking;
- prebuilt analyzers;
- classification;
- structured outputs;
- agentic mode.

The report distinguishes a GA API version from the preview version and recommends a pilot using anonymized documents, field-level precision and recall, manual review time, and downstream validation failures.

#### Teaching context and inference

The report does not claim that semantic chunking or structured outputs automatically produce correct business data. The general engineering lesson is that extraction quality must be measured against an explicit schema and downstream acceptance criteria.

#### Intuition

A document-to-data pipeline has several contracts:

```text
Document contract
      ↓
Extraction contract
      ↓
Structured-output schema
      ↓
Validation contract
      ↓
Downstream database or report contract
```

A visually plausible extraction can still be unusable if a required field is missing, a value has the wrong type, or evidence from two unrelated sections has been combined.

#### Precise technical explanation

A contract-first pipeline defines:

- accepted input types and size limits;
- service API version and preview status;
- expected output fields;
- field types and allowed values;
- confidence or review thresholds;
- handling for missing, ambiguous, or conflicting evidence;
- error and retry behavior;
- persistence and lineage requirements.

Semantic chunking groups content according to meaning rather than only character count or page boundaries. It can help preserve coherent context for retrieval or extraction, but it does not eliminate the need to validate relationships across chunks.

Synchronous APIs can simplify small request-response operations. Larger documents, long-running analysis, or human review may require asynchronous or stateful workflow patterns.

#### Realistic data-engineering example

A consulting team processes anonymized supplier contracts.

The pipeline:

1. submits a document to a version-pinned analysis API;
2. extracts layout and semantically coherent sections;
3. classifies the document type;
4. extracts fields such as supplier, renewal date, notice period, and governing law;
5. validates types and required fields;
6. sends low-confidence or contradictory results to manual review;
7. stores the result with document hash, analyzer version, API version, and validation outcome.

Field-level precision and recall are calculated against a reviewed sample.

#### Why it matters

Document extraction is often the first step in a larger data pipeline. Errors propagate into databases, reports, search indexes, and decisions. Contract-first design makes those errors visible and testable.

#### Common misconception

> “Structured JSON means the data is reliable.”

A structured response can still contain a wrong value, unsupported inference, missing evidence, or an incorrectly classified document. Structure is not correctness.

#### Relationship to adjacent concepts

- **APIs and networking:** versioned endpoints, request/response semantics, errors, and sync/async behavior.
- **AI and LLM systems:** contextualization, chunking, classification, and extraction.
- **Validation and data quality:** schema checks, field-level metrics, and downstream validation.
- **Security:** document handling and sensitive content controls.

#### Compact diagram

```text
Document
  ↓
Read and layout analysis
  ↓
Semantic sections or chunks
  ↓
Classification and extraction
  ↓
Schema validation
  ↓
Accepted record or human review
```

#### Recommended topic-file destination

- **Primary:** `learning/ai-and-llm-systems.md`
  - Proposed category: **Document extraction, semantic chunking, and structured outputs**.
- **Supporting:** `learning/apis-and-networking.md`
  - Proposed category: **Versioned service contracts and synchronous/asynchronous APIs**.
- **Supporting:** `learning/validation-and-data-quality.md`
  - Proposed category: **Validation of extracted business fields**.
- **Rationale:** the main concept concerns AI-enabled document processing, but it only becomes reliable when API and data contracts are explicit.

#### Recommended review-card destination

- **Primary:** `review-cards/ai-and-llm-systems.md`
  - Proposed category: **From documents to validated structured data**.
- **Supporting option:** `review-cards/validation-and-data-quality.md` for field-level extraction metrics.
- **Uncertainty:** the supplied files are empty, so the recommended categories are not existing sections.

---

### 5. Reproducible, configuration-driven analyzer workflows

#### Classification and reason

**NEW CONCEPT**

The `cu-cli` release signals a broader practice: analysis resources, analyzer definitions, configuration, expected outputs, and test commands should be reproducible and reviewable in source control.

#### Prerequisites

- Version control.
- CI/CD pipelines.
- Environment configuration.
- Separation of configuration and secrets.
- Basic cloud-resource provisioning.
- Test fixtures and expected outputs.

#### Release signal — directly supported by the report

The preview `cu-cli` can:

- provision the required Microsoft Foundry resource;
- optionally deploy supported models;
- configure Content Understanding defaults;
- analyze local files;
- manage analyzers and local configuration.

The report recommends evaluating it with sanitized documents, storing analyzer definitions and CLI configuration in Azure Repos, testing analyzers in an Azure DevOps pipeline, and comparing extracted fields with checked-in expected output.

#### Teaching context and inference

The reusable concept is not the particular CLI. It is **configuration as code plus executable validation**:

```text
Versioned definitions
        +
Pinned environment
        +
Repeatable commands
        +
Expected outputs
        =
Reproducible workflow
```

#### Precise technical explanation

A reproducible analyzer workflow should make the following explicit:

- analyzer definition;
- model and API versions;
- service configuration;
- input corpus or sanitized fixture;
- expected output;
- validation rules;
- resource and environment identifiers;
- deployment and teardown steps.

Secrets should not be committed with configuration. Instead, pipelines should inject them through an approved secret-management mechanism.

A workflow becomes more reproducible when a clean environment can be created from versioned inputs and a known command sequence, rather than relying on manual portal configuration.

#### Realistic data-engineering example

A team creates a repository with:

```text
analyzers/
  contract-fields.yaml
config/
  preview-environment.yaml
tests/
  expected-contract-fields.json
fixtures/
  sanitized-contract-001.pdf
pipeline/
  analyzer-tests.yml
```

The CI pipeline:

1. creates or selects a disposable resource group;
2. applies the analyzer configuration;
3. runs analysis on sanitized fixtures;
4. compares extracted fields to expected results;
5. records precision, recall, and validation failures;
6. destroys or resets temporary resources.

#### Why it matters

Manual analyzer configuration creates hidden state. A reproducible workflow makes changes reviewable, supports rollback, and allows teams to distinguish a model change from a configuration change or service-version change.

#### Common misconception

> “Putting a configuration file in Git automatically makes the workflow reproducible.”

Reproducibility also requires versioned dependencies, stable inputs, known service versions, controlled secrets, repeatable commands, and explicit expected outputs.

#### Relationship to adjacent concepts

- **Deployment and infrastructure:** environment provisioning and CI/CD.
- **Software architecture:** declarative configuration and separation of concerns.
- **Validation and data quality:** expected-output tests.
- **AI systems:** analyzer and model versioning.

#### Compact diagram

```text
Repository
  ├── analyzer definitions
  ├── configuration
  ├── sanitized fixtures
  └── expected outputs
          ↓
       CI pipeline
          ↓
   Disposable environment
          ↓
    Analysis and comparison
          ↓
   Reviewable pass/fail result
```

#### Recommended topic-file destination

- **Primary:** `learning/deployment-and-infrastructure.md`
  - Proposed category: **Configuration as code and reproducible analysis workflows**.
- **Supporting:** `learning/software-architecture.md`
  - Proposed category: **Declarative configuration and executable interfaces**.
- **Supporting:** `learning/validation-and-data-quality.md`
  - Proposed category: **Expected-output tests for analyzers**.
- **Rationale:** the workflow is primarily a deployment and CI/CD practice, with architectural and validation dependencies.

#### Recommended review-card destination

- **Primary:** `review-cards/deployment-and-infrastructure.md`
  - Proposed category: **Reproducible configuration-driven workflows**.
- **Uncertainty:** no existing review-card section is available to confirm a more specific placement.

---

### 6. Evaluation as a quality gate for AI systems

#### Classification and reason

**NEW CONCEPT**

Microsoft Foundry cloud evaluations introduce a reusable distinction between building an AI application and measuring whether a particular version is acceptable for release or continued operation.

#### Prerequisites

- Test cases and expected behavior.
- Sampling and representative datasets.
- Precision, recall, and other evaluation metrics.
- Versioning of application and deployment artifacts.
- Basic understanding of false positives and false negatives.

#### Release signal — directly supported by the report

The report describes cloud evaluations for repeatable pre-deployment and production workflows. It recommends a 30–50 case golden set for one summarization or classification workflow and evaluating:

- factuality;
- completeness;
- format compliance;
- refusal behavior;
- PII handling.

Scores should be stored with the application version and deployment identifier.

#### Teaching context and inference

A golden set is not automatically representative. It must be curated, reviewed, maintained, and periodically expanded to reflect real failure modes and data drift.

#### Intuition

An AI application needs a versioned quality loop:

```text
Prompt or input
      ↓
Model and application version
      ↓
Output
      ↓
Evaluation criteria
      ↓
Score and failure analysis
      ↓
Release, investigate, or rollback
```

#### Precise technical explanation

An evaluation suite should define:

- the input cases;
- the task and expected properties;
- the scoring method;
- acceptance thresholds;
- segmentation dimensions;
- the application, model, prompt, and deployment versions;
- treatment of ambiguous or human-reviewed cases.

Different tasks require different metrics. A classification workflow may use precision, recall, and confusion matrices. A summarization workflow may require factuality, completeness, structure, and human review. A refusal or safety workflow may need explicit adversarial and boundary cases.

An evaluation is a quality gate when the result can affect deployment or promotion. It becomes more useful when failures are stored with enough metadata to reproduce and investigate them.

#### Realistic data-engineering example

A team maintains a 40-case golden set for generating structured summaries of project documents.

For every prompt or model change, the pipeline records:

```text
application version
model identifier
prompt/template version
deployment identifier
case identifier
output
evaluation scores
human review status
```

Promotion is blocked if factuality falls below the agreed threshold or if format compliance fails for required fields.

#### Why it matters

A successful demo is not evidence of stable quality. Versioned evaluations make quality changes visible before or after deployment and reduce dependence on anecdotal examples.

#### Common misconception

> “A single average score tells us whether the AI system is safe to release.”

An average can conceal severe failures in a small but important subgroup, such as documents containing PII, rare contract clauses, or multilingual content. Segment-level analysis and failure review are necessary.

#### Relationship to adjacent concepts

- **Validation and data quality:** acceptance criteria and regression testing.
- **Statistics and evaluation:** sampling, uncertainty, metric interpretation, and subgroup analysis.
- **AI systems:** model, prompt, retrieval, and agent behavior.
- **Observability:** production monitoring after the offline gate.

#### Compact diagram

```text
Curated golden set
        ↓
Versioned AI application
        ↓
Repeatable evaluation
        ↓
Metric and failure analysis
        ↓
Deployment decision
```

#### Recommended topic-file destination

- **Primary:** `learning/ai-and-llm-systems.md`
  - Proposed category: **Evaluation gates for AI applications**.
- **Supporting:** `learning/validation-and-data-quality.md`
  - Proposed category: **Golden datasets and acceptance thresholds**.
- **Optional indexed destination:** `statistics-and-evaluation.md`.
  - This file is listed in the authoritative index but was not supplied, so its existing contents and exact category cannot be compared.
- **Rationale:** use the supplied AI and validation files as the primary destinations rather than assuming content in the absent statistics file.

#### Recommended review-card destination

- **Primary:** `review-cards/ai-and-llm-systems.md`
  - Proposed category: **Golden-set evaluation and release gates**.
- **Supporting option:** `review-cards/validation-and-data-quality.md`.
- **Uncertainty:** the indexed statistics review-card file was not supplied; no destination within it is assumed.

---

### 7. Observability for AI and agent operations

#### Classification and reason

**NEW CONCEPT**

The Agent Monitoring Dashboard provides a direct signal for applying standard observability principles to AI workloads, where cost, latency, success, and evaluation quality must be considered together.

#### Prerequisites

- Metrics, logs, and traces.
- Percentiles, especially p95 latency.
- Correlation identifiers and deployment metadata.
- Basic cost accounting.
- Difference between operational health and business quality.

#### Release signal — directly supported by the report

The Microsoft Foundry Agent Monitoring Dashboard tracks:

- token usage;
- latency;
- success rates;
- evaluation outcomes.

The report recommends instrumenting a non-critical internal agent, defining thresholds for failed runs, p95 latency, token cost per task, and evaluation score, reviewing traces weekly, and verifying sensitive prompt/output handling.

#### Teaching context and inference

AI observability should connect technical telemetry to a unit of business work, such as “one document classified” or “one report drafted,” rather than reporting only aggregate token counts.

#### Intuition

An AI system can be operationally available but still poor:

```text
Request succeeds
      ≠
Output is correct, affordable, timely, and safe
```

Useful monitoring combines:

```text
Reliability + latency + cost + quality + security signals
```

#### Precise technical explanation

AI observability should answer:

- Did the request complete?
- How long did it take?
- How many tokens or other resources did it consume?
- Which application, model, prompt, and deployment produced the result?
- Did the output satisfy evaluation criteria?
- Did the workflow require retries or human intervention?
- Was sensitive content handled according to policy?

Percentiles are often more useful than averages for latency because a small number of slow requests can materially affect users. Traces help connect an agent run to tool calls, retrieval steps, model calls, retries, and approval events.

#### Realistic data-engineering example

An internal agent prepares a draft summary from project documents.

The team monitors:

- success rate;
- p50 and p95 latency;
- token cost per completed task;
- number of tool calls;
- evaluation score;
- human rejection rate;
- prompt and output handling policy.

Each run includes an application version, model identifier, deployment identifier, and trace ID. A weekly review looks for changes after prompt, model, or tool updates.

#### Why it matters

Without operational telemetry, teams may detect quality degradation only through user complaints. Cost and latency can also grow silently as prompts, context windows, or tool chains become more complex.

#### Common misconception

> “A successful HTTP response means the agent is healthy.”

HTTP success says that a request completed at the transport or service layer. It does not establish that the answer was useful, compliant, affordable, or safe.

#### Relationship to adjacent concepts

- **Observability and reliability:** metrics, traces, thresholds, and incident response.
- **AI evaluation:** offline and production quality measurements.
- **APIs:** request correlation and service-level behavior.
- **Security:** sensitive prompt and output handling.
- **Deployment:** linking telemetry to versions and releases.

#### Compact diagram

```text
Agent request
  ↓
Trace: retrieval → model → tools → response
  ↓
Metrics: latency, tokens, success, cost
  ↓
Quality signal: evaluation or human review
  ↓
Operational decision
```

#### Recommended topic-file destination

- **Primary:** `learning/observability-and-reliability.md`
  - Proposed category: **AI and agent observability**.
- **Supporting:** `learning/ai-and-llm-systems.md`
  - Proposed category: **Operational telemetry for AI applications**.
- **Supporting:** `learning/deployment-and-infrastructure.md`
  - Proposed category: **Version-linked monitoring and promotion feedback**.
- **Rationale:** the main concept is observability, with AI-specific dimensions and deployment metadata.

#### Recommended review-card destination

- **Primary:** `review-cards/observability-and-reliability.md`
  - Proposed category: **AI operational health versus output quality**.
- **Supporting option:** `review-cards/ai-and-llm-systems.md`.
- **Uncertainty:** the supplied observability card file is empty, so the category is a draft recommendation.

---

### 8. Durable workflows and human-in-the-loop control

#### Classification and reason

**NEW CONCEPT**

The report’s long-running hosted-agent signal points to a general workflow-engineering problem: a multi-step process must preserve state, survive interruption, avoid duplicate side effects, and stop for an explicit approval decision when required.

#### Prerequisites

- State machines and workflow stages.
- Timeouts and retries.
- Idempotency.
- Persistent state.
- Basic failure handling.
- Authentication and authorization for approval actions.

#### Release signal — directly supported by the report

Preview documentation includes:

- long-running agent APIs;
- resilience for long-running hosted agents;
- human-in-the-loop approval steps.

The report recommends a low-risk prototype that gathers documents, drafts a structured summary, pauses for approval, and publishes only after approval. It explicitly recommends testing restart, timeout, duplicate submission, rejection, and partial-failure paths.

#### Teaching context and inference

The report does not confirm an incident. The general lesson is that an agent that performs work across time and external systems should be treated as a durable workflow, not merely as a single request.

#### Intuition

A long-running workflow is a state machine:

```text
Received
   ↓
Documents gathered
   ↓
Draft generated
   ↓
Awaiting approval
   ├── rejected → revise or close
   └── approved → publish
```

The system must know where it is if the process restarts between any two states.

#### Precise technical explanation

A durable workflow generally needs:

- a persistent workflow identifier;
- explicit state transitions;
- durable storage for intermediate results;
- timeouts and expiry rules;
- retry policies;
- idempotent operations for externally visible effects;
- approval identity and authorization;
- audit history;
- compensation or recovery for partial failure.

For example, “publish report” should not happen twice merely because an approval callback was delivered twice. The publish operation should use an idempotency key or check the durable workflow state before applying the side effect.

#### Realistic data-engineering example

An internal workflow:

1. receives a set of project documents;
2. stores document identifiers and a workflow ID;
3. extracts fields and drafts a structured summary;
4. stores the draft and its evaluation results;
5. pauses for approval;
6. records the approver and decision;
7. publishes the approved draft once;
8. records the final publication result.

Tests simulate:

- service restart during extraction;
- timeout while awaiting approval;
- duplicate approval submission;
- rejected draft;
- missing document;
- publication failure after approval.

#### Why it matters

Long-running workflows cross process boundaries and time periods. Without durable state and idempotent side effects, retries can create duplicate records, accidental publication, inconsistent status, or unrecoverable partial work.

#### Common misconception

> “Adding retries makes a workflow resilient.”

Retries without idempotency and state management can amplify failures. A retry may repeat a side effect or conceal the actual point of failure.

#### Relationship to adjacent concepts

- **Software architecture:** state machines, interfaces, and workflow boundaries.
- **Distributed systems:** retries, duplicate delivery, coordination, and partial failure.
- **Observability:** traceable state transitions and failure diagnosis.
- **Security:** approval authorization and auditability.
- **AI systems:** agent planning and tool use.

#### Compact diagram

```text
Persisted workflow state
          ↓
  Step execution
          ↓
Success → next state
Failure → retry or recovery state
Restart → resume from persisted state
Approval → authorized transition
```

#### Recommended topic-file destination

- **Primary:** `learning/software-architecture.md`
  - Proposed category: **Durable workflows and human-in-the-loop state transitions**.
- **Supporting:** `learning/observability-and-reliability.md`
  - Proposed category: **Workflow resilience, retries, and audit trails**.
- **Optional indexed destination:** `distributed-systems.md`.
  - This file is listed in the authoritative index but was not supplied, so its existing contents and exact category cannot be compared.
- **Rationale:** the supplied architecture and reliability files provide suitable primary destinations without assuming content in the absent distributed-systems file.

#### Recommended review-card destination

- **Primary:** `review-cards/software-architecture.md`
  - Proposed category: **Durable workflow state and approval transitions**.
- **Supporting option:** `review-cards/observability-and-reliability.md`.
- **Uncertainty:** no supplied card file for the indexed distributed-systems category is available.

---

### 9. Data minimization and PII controls in AI pipelines

#### Classification and reason

**NEW CONCEPT**

The report repeatedly signals the need to handle sensitive content deliberately: pilot with anonymized documents, evaluate PII handling, and verify sensitive prompt/output treatment.

#### Prerequisites

- Data classification.
- Authentication and authorization.
- Least privilege.
- Retention and deletion concepts.
- Difference between anonymization, pseudonymization, masking, and redaction.
- Basic audit logging.

#### Release signal — directly supported by the report

The report recommends:

- using anonymized documents for the Content Understanding pilot;
- evaluating PII handling in Foundry cloud evaluations;
- verifying sensitive prompt and output handling in agent monitoring;
- using sanitized documents for `cu-cli` evaluation.

The report does not assert that a Kienbaum incident occurred. These are preventive controls and evaluation requirements.

#### Teaching context and inference

Data minimization is not only a security-control issue. It also improves testing discipline by making the test corpus safer to share, easier to reproduce, and less likely to expose unnecessary personal information.

#### Intuition

The safest sensitive data is data the system never receives:

```text
Collect only what is needed
        ↓
Remove or transform unnecessary identifiers
        ↓
Restrict access and retention
        ↓
Monitor prompts, outputs, and derived artifacts
```

#### Precise technical explanation

Relevant controls include:

- minimizing input fields;
- identifying direct and indirect identifiers;
- applying suitable de-identification before testing;
- separating production data from test fixtures;
- restricting service and pipeline permissions;
- controlling prompt, output, trace, and cache retention;
- preventing secrets or personal data from entering logs;
- testing both expected and adversarial PII cases;
- recording access and processing decisions.

Anonymization, pseudonymization, masking, and redaction are not interchangeable. Pseudonymized data may still be linkable if a re-identification key exists. A redacted document may also lose information needed for a valid extraction test.

#### Realistic data-engineering example

A document-processing pilot uses sanitized contracts. The team:

1. removes or transforms names, addresses, account numbers, and contact details;
2. retains structural patterns needed to test field extraction;
3. marks which fields were transformed;
4. ensures prompts and outputs are not retained beyond the test period;
5. checks traces and monitoring dashboards for unintended sensitive content;
6. evaluates whether the system correctly refuses or protects PII-related requests.

#### Why it matters

AI and document services can create multiple copies of information: original input, extracted text, chunks, prompts, tool arguments, outputs, traces, evaluation records, and cached artifacts. Security must cover the full data lifecycle.

#### Common misconception

> “Using an approved cloud service means PII risk has been solved.”

Service approval does not replace data minimization, access control, retention policy, logging review, or application-specific testing.

#### Relationship to adjacent concepts

- **Security:** least privilege, secrets, access, and retention.
- **AI systems:** prompts, outputs, retrieval context, and agents.
- **Validation:** PII test cases and policy assertions.
- **Observability:** sensitive data in traces and logs.
- **Deployment:** environment separation and configuration controls.

#### Compact diagram

```text
Source document
      ↓
Classify sensitive fields
      ↓
Minimize or de-identify
      ↓
Process with controlled access
      ↓
Validate output and traces
      ↓
Retain or delete according to policy
```

#### Recommended topic-file destination

- **Primary:** `learning/security.md`
  - Proposed category: **Data minimization, de-identification, and AI data handling**.
- **Supporting:** `learning/ai-and-llm-systems.md`
  - Proposed category: **PII controls for prompts, outputs, and agent traces**.
- **Supporting:** `learning/validation-and-data-quality.md`
  - Proposed category: **Security and policy test cases for data pipelines**.
- **Rationale:** security is the primary subject, while AI and validation provide the operational test context.

#### Recommended review-card destination

- **Primary:** `review-cards/security.md`
  - Proposed category: **Minimize and control sensitive data throughout an AI pipeline**.
- **Supporting option:** `review-cards/ai-and-llm-systems.md`.
- **Uncertainty:** the supplied security card file is empty, so the recommended category is not an existing section.

## Proposed permanent-resource changes

This section contains **draft recommendations only**. No permanent changes have been made.

### Learning-resource index

No new topic filename is needed. The current taxonomy is broad enough to cover all selected concepts.

If the Learning Coach approves later, the index could add explicit concept-to-destination mappings:

| Concept | Recommended index route |
|---|---|
| Runtime and dependency compatibility | `deployment-and-infrastructure.md`; supporting `software-architecture.md` and `validation-and-data-quality.md` |
| Data-correctness regression testing | `validation-and-data-quality.md`; supporting `data-processing.md` |
| Parallel-efficiency diagnostics | `concurrency-and-parallelism.md`; supporting `observability-and-reliability.md` |
| Document extraction and structured outputs | `ai-and-llm-systems.md`; supporting `apis-and-networking.md` and `validation-and-data-quality.md` |
| Configuration-driven analyzer workflows | `deployment-and-infrastructure.md`; supporting `software-architecture.md` and `validation-and-data-quality.md` |
| AI evaluation gates | `ai-and-llm-systems.md`; supporting `validation-and-data-quality.md` |
| AI and agent observability | `observability-and-reliability.md`; supporting `ai-and-llm-systems.md` |
| Durable workflows and approvals | `software-architecture.md`; supporting `observability-and-reliability.md` |
| PII controls in AI pipelines | `security.md`; supporting `ai-and-llm-systems.md` and `validation-and-data-quality.md` |

The index also lists `statistics-and-evaluation.md` and `distributed-systems.md`, but those files were not supplied for comparison. They should not be treated as existing populated destinations in this draft.

### Recommended existing topic-file additions

If approved later, add scoped sections to the existing files rather than creating new files:

- `learning/deployment-and-infrastructure.md`
  - Runtime compatibility and dependency release gates.
  - Configuration as code and reproducible analyzer workflows.
- `learning/software-architecture.md`
  - Dependency boundaries.
  - Durable workflows and human-in-the-loop state transitions.
- `learning/validation-and-data-quality.md`
  - Regression testing for data-producing systems.
  - Golden datasets and expected-output tests.
  - Validation of extracted business fields.
- `learning/data-processing.md`
  - Serialization, output boundaries, and stale-data prevention.
- `learning/concurrency-and-parallelism.md`
  - Parallel-efficiency diagnostics and coordination overhead.
- `learning/observability-and-reliability.md`
  - Performance telemetry for concurrent jobs.
  - AI and agent observability.
  - Workflow resilience and audit trails.
- `learning/ai-and-llm-systems.md`
  - Document extraction, semantic chunking, and structured outputs.
  - Evaluation gates for AI applications.
  - Operational telemetry for AI systems.
  - PII handling in prompts, outputs, and agent traces.
- `learning/apis-and-networking.md`
  - Versioned service contracts and synchronous/asynchronous API behavior.
- `learning/security.md`
  - Data minimization, de-identification, and AI data handling.

No new topic file is proposed.

### Recommended existing review-card-file additions

If approved later, add one compact card for each proposed concept to the corresponding existing review-card file:

- `review-cards/deployment-and-infrastructure.md`
  - Runtime and dependency compatibility.
  - Reproducible configuration-driven workflows.
- `review-cards/validation-and-data-quality.md`
  - Regression tests for values, types, shape, and absence.
  - Golden-set and expected-output validation.
- `review-cards/concurrency-and-parallelism.md`
  - Measuring actual parallel benefit.
- `review-cards/ai-and-llm-systems.md`
  - Document-to-validated-data pipelines.
  - AI evaluation gates.
  - AI operational observability.
  - PII controls for AI workflows.
- `review-cards/observability-and-reliability.md`
  - Performance telemetry for parallel jobs.
  - AI operational health versus output quality.
  - Durable workflow resilience.
- `review-cards/software-architecture.md`
  - Durable workflow state and approval transitions.
- `review-cards/security.md`
  - Data minimization and sensitive-data lifecycle controls.

The corresponding review-card files for the indexed `statistics-and-evaluation.md` and `distributed-systems.md` destinations were not supplied. No card placement inside those absent files is assumed.

## Proposed review cards

### Card 1 — Dependency, runtime, and binary compatibility

- **Recall:** What must be compatible besides the direct package version?
- **Mental hook:** A data application is a stack, not a package.
- **Why I care:** A resolver can succeed while imports, datetime paths, or compiled dependencies fail at runtime.
- **Contrast:** A stable release lane is different from an isolated beta-compatibility lane.
- **Tiny example:** Test pandas with the exact Python, NumPy, operating-system image, and ETL paths used in deployment.
- **Test yourself:** Why does a lockfile reduce uncertainty but not prove binary compatibility?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md`, proposed category **Runtime and dependency compatibility**.

### Card 2 — Regression testing for data correctness

- **Recall:** What should a data-output regression test verify besides process completion?
- **Mental hook:** “No exception” is not the same as “correct report.”
- **Why I care:** Date misinterpretation and stale cells can silently change business outputs.
- **Contrast:** A binary workbook snapshot is not always as useful as normalized checks for values, types, shape, and absence.
- **Tiny example:** Write 100 rows, then 80 rows, and assert that rows 81–100 are blank.
- **Test yourself:** Which edge case would expose a shrinking-output bug in an automated Excel report?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/validation-and-data-quality.md`, proposed category **Regression tests for data-producing systems**.

### Card 3 — Parallel-efficiency diagnostics

- **Recall:** What costs can offset the theoretical benefit of more workers?
- **Mental hook:** Parallelism is compute speedup minus coordination cost.
- **Why I care:** More workers can make a job slower and more expensive.
- **Contrast:** High CPU usage is not equivalent to high end-to-end efficiency.
- **Tiny example:** Eight workers underperform four because each task repeatedly serializes and transfers a large lookup table.
- **Test yourself:** Which measurements would distinguish poor worker utilization from excessive data-transfer overhead?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/concurrency-and-parallelism.md`, proposed category **Parallel-efficiency diagnostics**.

### Card 4 — Contract-first document-to-data pipelines

- **Recall:** What contracts exist between a source document and a downstream record?
- **Mental hook:** Structured output is a shape, not a guarantee of truth.
- **Why I care:** Extraction errors propagate into databases, reports, and decisions.
- **Contrast:** Semantic chunking improves context organization but does not replace schema validation or field-level evaluation.
- **Tiny example:** Extract a contract renewal date, validate its type and range, and route ambiguous values to review.
- **Test yourself:** What metadata would you store to reproduce an extraction result six weeks later?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/ai-and-llm-systems.md`, proposed category **Document-to-validated structured data**.

### Card 5 — Reproducible configuration-driven workflows

- **Recall:** What must be versioned for an analyzer test to be reproducible?
- **Mental hook:** If it cannot be recreated from the repository and known secrets, it is hidden state.
- **Why I care:** Manual cloud configuration makes analyzer changes difficult to review, reproduce, and roll back.
- **Contrast:** Configuration as code is not the same as committing secrets or assuming the environment is fixed.
- **Tiny example:** Store an analyzer definition, sanitized fixture, expected fields, and CI command in source control.
- **Test yourself:** Which dependency would remain uncontrolled if only the analyzer definition were versioned?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md`, proposed category **Reproducible configuration-driven workflows**.

### Card 6 — AI evaluation as a quality gate

- **Recall:** What makes an evaluation actionable before deployment?
- **Mental hook:** A golden set turns “it looks good” into a repeatable comparison.
- **Why I care:** A model, prompt, or application change can improve one behavior while degrading another.
- **Contrast:** An average score is not a substitute for failure analysis and subgroup checks.
- **Tiny example:** Block promotion when factuality or required-format compliance falls below an agreed threshold.
- **Test yourself:** Which cases should be added to a golden set after a production failure?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/ai-and-llm-systems.md`, proposed category **Golden-set evaluation and release gates**.

### Card 7 — AI and agent observability

- **Recall:** Which signals describe AI operational health and which describe output quality?
- **Mental hook:** A successful request can still be a bad task result.
- **Why I care:** Token cost, latency, reliability, quality, and sensitive-data exposure can change independently.
- **Contrast:** Monitoring availability is not the same as monitoring usefulness.
- **Tiny example:** Track p95 latency, token cost per task, evaluation score, and human rejection rate by deployment version.
- **Test yourself:** Why should an agent trace include the application, model, prompt, and deployment identifiers?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/observability-and-reliability.md`, proposed category **AI operational health versus output quality**.

### Card 8 — Durable workflows and human approval

- **Recall:** What protects a long-running workflow from restart, timeout, and duplicate submission?
- **Mental hook:** Treat the agent as a state machine with durable checkpoints.
- **Why I care:** Retries and partial failures can otherwise duplicate or prematurely publish business outputs.
- **Contrast:** Retrying a request is not the same as safely resuming a workflow.
- **Tiny example:** Persist `awaiting_approval`, require an authorized transition, and make publication idempotent.
- **Test yourself:** What should happen if the approval callback is delivered twice?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/software-architecture.md`, proposed category **Durable workflow state and approval transitions**.

### Card 9 — Data minimization and PII controls

- **Recall:** Where can sensitive data appear in an AI pipeline?
- **Mental hook:** Inputs become copies: chunks, prompts, outputs, traces, caches, and evaluation records.
- **Why I care:** Approved services do not eliminate application-specific data exposure or retention risk.
- **Contrast:** Pseudonymization is not necessarily anonymization; transformed data may remain linkable.
- **Tiny example:** Use sanitized contracts for testing and verify that traces do not retain personal identifiers.
- **Test yourself:** Which fields can be removed while preserving the structural pattern needed to test extraction?
- **First learned date:** 2026-09-11
- **Recommended review-card destination:** `review-cards/security.md`, proposed category **Sensitive-data lifecycle controls for AI pipelines**.

## Questions worth discussing

1. How large should a compatibility matrix be before it becomes too costly, and which dimensions should be prioritized first?
2. Which data-output assertions are most valuable for Excel reports: exact values, types, dimensions, formatting, absence checks, or all of these?
3. How can we distinguish poor parallelization from an inherently I/O-bound workload using measurements available in staging?
4. When does semantic chunking improve document extraction, and how should cross-chunk relationships be tested?
5. How should a golden evaluation set evolve after a new production failure without becoming unrepresentative or overly narrow?
6. Which AI observability signals should be treated as release blockers, and which should trigger investigation only?
7. What is the right workflow state model for a document agent that can pause for approval and resume after several days?
8. What is the minimum de-identification and retention policy needed before using consulting documents in a preview-service pilot?
