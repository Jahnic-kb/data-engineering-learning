# Learning workshop — 2026-11-09

## Why these concepts matter this week

The release report is mostly about **changes that can fail silently**: a credential that still works until its retirement date, a dependency release that may carry malicious code, a library default that changes what text or row order means, and an AI routing feature that returns valid-looking answers that may be wrong. The eight concepts below are the engineering ideas behind those changes. They are chosen for reuse across Python, R, Azure, and AI pipelines, not for the specific products.

Two operational items in the report are not learning tasks and should not wait for this workshop: the LiteLLM check for affected environments, and the Azure DevOps global PAT inventory before 27 Jan 2027. The LiteLLM poisoning report is marked [UNVERIFIED] and comes from a secondary newsletter, so this workshop does not treat it as confirmed.

**Source status and assumptions**

- I checked the working environment and could not find the repository. I relied on your statement that the eight named topic and review-card files are empty.
- Files not named in your input (`databases.md`, `apis-and-networking.md`, `validation-and-data-quality.md`, `statistics-and-evaluation.md`, `observability-and-reliability.md`, and others) were not supplied. Where a concept maps to one of them, the mapping is provisional and the classification is based on the absence of supplied finalized entries, not on a comparison with their content.
- Because no finalized entries exist for any of the candidate concepts, every selected concept is **NEW CONCEPT**. No concept is EXTEND EXISTING CONCEPT or NO UPDATE NEEDED. Material in every section below is therefore **not yet grounded in a reviewed topic file** and should be treated as draft.
- Exposure to the Polars, Python 3.15, LiteLLM, and Foundry changes is unknown, because the installed versions at Kienbaum are unknown.

## Learning map

```text
Chain A: Trust and access
Authentication vs authorization
↓
C1  Least-privilege credentials, expiry, rotation
↓
C8  Untrusted content and approval gates for agents

Chain B: Change and upgrades
Versioning and dependency specifications
↓
C2  Supply-chain integrity (provenance, lockfiles, pinning)
↓
C3  Breaking changes and regression testing
↓
C4  Text encoding (bytes vs text)
C5  Out-of-core execution and memory budgets
(both produce silent behavior changes that C3 tests for)

Chain C: Measuring AI decisions
Confusion matrix (TP, FP, FN, TN)
↓
C7  Threshold selection and labeled-set evaluation
↓
C6  Typed structured outputs from models
↓
C8  Approval gates for agent actions
```

## Concepts

### C1. Least-privilege credentials, expiry, and rotation

**Classification:** NEW CONCEPT. The security topic file is empty, so there is no finalized entry to extend.

**Prerequisites:** Authentication (who you are) vs authorization (what you may do); bearer tokens; the idea that a secret is any value that grants access; environment variables and pipeline variables as places where secrets are stored.

**Intuition:** A credential is a key. Its risk depends on three things: what doors it opens (scope), how long it still works (lifetime), and whether you can replace it without breaking the building (rotation).

**Precise explanation:** A credential has three properties. **Scope** is the set of actions and resources it permits. **Lifetime** is how long it remains valid. **Holder** is whether it identifies a person or a workload. Rotation means issuing a new credential, updating every consumer, and revoking the old one. Workload identities such as managed identities avoid storing a long-lived secret at all, because the platform issues short-lived tokens at runtime.

**Grounding in the report:**
- *Report:* Azure DevOps global personal access tokens (PATs) stop working on 27 Jan 2027. Pipelines, scripts, and extension publishing that use them will fail. The report recommends an inventory, then migration to org-scoped PATs or Entra-based auth. Filter "All accessible organizations" finds global tokens. The report notes an older sprint 270 note still shows 1 Dec 2026, so the newer date governs.
- *Report:* If an affected LiteLLM build ran, cloud, Git, or SSH credentials may be exposed and should be rotated.
- *Context:* The scope and lifetime framing and the managed-identity pattern are general teaching context, not report claims.

**Realistic data-engineering example:** A nightly Python job calls the Azure DevOps REST API to read pipeline run data. The job authenticates with a global PAT stored in a variable group. The token's scope covers every organization the owner can access, and it has no owner-specific expiry review. When the token is retired, the job fails at 02:00. A better design uses a workload identity that obtains a short-lived token at runtime, so no secret sits in the variable group. Rotation is still needed for any credential that remains.

**Why it matters:** Credential exposure is only as serious as the access the credential grants. A narrow, short-lived credential turns an incident into a bounded event.

**Common misconception:** "Rotating a secret means generating a new one." In practice, rotation fails because consumers are unknown. The inventory must find every consumer, and the old credential should be revoked only after the new one is confirmed working. A second misconception is that an expiring token is therefore safe. Lifetime limits exposure time, but a broad scope still exposes everything reachable during that time.

**Relationship to adjacent concepts:** C2 explains how a malicious dependency can reach these credentials. C8 applies least privilege to agents that hold tools.

```text
Credential = scope × lifetime × holder
Rotation = issue new → update all consumers → verify → revoke old
```

**Recommended topic-file destination:** `learning/security.md`, a proposed subsection "Credentials and secrets" under the existing scope "secrets, permissions, least privilege." Rationale: this is the closest existing category and the concept is broadly reusable across data and API systems.

**Recommended review-card destination:** `review-cards/security.md`, credentials section. Uncertainty: the review-card file is empty, so no existing section can be confirmed.

### C2. Supply-chain integrity: provenance, lockfiles, and pinning

**Classification:** NEW CONCEPT. The security topic file is empty.

**Prerequisites:** Package managers and dependency resolution; version specifiers such as `>=` ranges; lockfiles; container images and how they are built; CI pipelines that publish packages.

**Intuition:** Installing a dependency means running someone else's code with your permissions. The supply chain is the path from the source repository to the artifact you actually execute. Each step on that path can be a point of compromise.

**Precise explanation:** A version range lets new releases in automatically. A **lockfile** records the exact resolved versions, and hash pinning verifies the artifact bytes, so an installed package can be checked against the expected content. **Provenance** (an attestation) records which build workflow and source commit produced an artifact. Provenance establishes origin; it does not establish safety. If the source repository or build workflow was itself compromised, provenance will faithfully describe a malicious build.

**Grounding in the report:**
- *Report:* A reported poisoned LiteLLM release is marked [UNVERIFIED]. PyPI shows version 1.108.3 released 7 Nov 2026. The recommended action is to search Oct–Nov 2026 environments, lockfiles, and container images for `litellm`.
- *Report:* Polars 2.1.0 was published to PyPI on 25 Oct 2026 "with provenance from GitHub Actions."
- *Context:* The distinction between origin and safety, and the pinning trade-off, are general teaching context.

**Realistic data-engineering example:** A container image for a data pipeline is built each Monday with `pip install litellm>=1.100`. Two images built a week apart contain different versions, and neither build records which version it used. The fix is a lockfile committed alongside the pipeline, a rebuild that installs only from the lockfile, and an inventory query across all images for the affected package name. The inventory is a direct application of this concept.

**Why it matters:** Most dependency incidents are discovered by inventory, not by code review. Without a lockfile or image manifest, the question "were we running that version?" cannot be answered quickly.

**Common misconception:** "Provenance means the package is safe." It means the package came from a particular workflow and commit. A third misconception is that pinning is always protective. Pinning freezes a project on a vulnerable version too, so pinning must be paired with a regular, tested update process (see C3).

**Relationship to adjacent concepts:** C1 describes what an attacker can do with the credentials a compromised dependency can read. C3 describes how to upgrade pins safely.

```text
Source repo → CI build (+ provenance) → registry → lockfile (+ hashes) → image → runtime
Each arrow is a trust decision.
```

**Recommended topic-file destination:** `learning/security.md`, proposed subsection "Software supply chain." Cross-reference `learning/deployment-and-infrastructure.md` for CI/CD and image builds. Rationale: supply-chain trust is a security concept, but its operational content (builds, images) belongs with deployment.

**Recommended review-card destination:** `review-cards/security.md`, supply-chain section. Uncertainty: the file is empty.

### C3. Breaking changes, compatibility contracts, and regression testing

**Classification:** NEW CONCEPT. The deployment topic file is empty.

**Prerequisites:** Semantic versioning as a convention (a major version number signals incompatible changes, but projects are not bound to it); dependency specifications; test suites with known-good outputs; environment matrices (Python version, OS, architecture).

**Intuition:** A version number is a promise you verify with tests, not a guarantee you can rely on by reading the number. Some breaks are loud (an import fails) and some are silent (the same code returns different rows or text).

**Precise explanation:** Compatibility covers three surfaces. **Interface** changes (signatures, names) usually fail at import or call time. **Runtime requirements** (minimum Python version, native libraries) fail at install or start. **Behavior** changes (defaults, ordering, coercion, encoding) can pass every interface check. A regression test runs the same inputs through the old and new versions and compares outputs. When packages are coupled (for example, an agent framework and its adapters), they must be upgraded together. When version streams are decoupled (for example, a runtime SDK and a CLI released on the same day), the exact combination must be recorded.

**Grounding in the report:**
- *Report:* Polars 2.1 drops Python 3.10. Mixed string and numeric arithmetic in eager mode now raises an error. Row order is no longer guaranteed for join, group_by, and unpivot unless `maintain_order=True`.
- *Report:* Python 3.15 changes the default encoding for file I/O to UTF-8.
- *Report:* Agent Framework Python 1.23.0 includes breaking MCP 2.x changes. Upgrade the framework and adapters together.
- *Report:* Foundry Local SDK/runtime 2.2.1 and CLI 0.12.1 shipped on the same day as separate streams. "Keep versions apart when planning upgrades."
- *Report:* A secondary source claims a default API-version change rejects `temperature`, `top_p`, or `top_k` [UNVERIFIED]. It is included here only as an illustration of how default versions can change request validity.

**Realistic data-engineering example:** A monthly revenue job is upgraded from one Polars major version to another. The unit tests pass because all function signatures still exist. The monthly output then differs because a join returns rows in a different order, and a downstream process that assumed order fails. A side-by-side run on last month's inputs, with row order normalized and then not normalized, would have exposed the difference before release.

**Why it matters:** Silent behavior changes are the most expensive kind of breaking change, because they pass review and reach production before anyone notices.

**Common misconception:** "A major version bump means breaking; minor and patch releases are safe." Conventions vary and are not guaranteed, and behavior changes can occur in any release. Python 3.15 and similar language releases do not follow a package versioning scheme at all.

**Relationship to adjacent concepts:** C2 covers pinning, which makes upgrades deliberate. C4 and C5 are two concrete sources of silent change. The validation topic file should hold the testing techniques (golden outputs, differential tests).

```text
Old version ─┐
             ├─ same inputs ─► outputs ─► diff (order-normalized and raw) ─► decision
New version ─┘
```

**Recommended topic-file destination:** `learning/deployment-and-infrastructure.md`, proposed subsection "Version management and upgrades." Cross-reference `learning/validation-and-data-quality.md` for regression-testing technique. Rationale: version pinning and environment matrices are deployment concerns; the test design is a validation concern. The validation file was not supplied, so the cross-reference is provisional.

**Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md`, version-management section. Uncertainty: the file is empty.

### C4. Text encoding and the bytes/text boundary

**Classification:** NEW CONCEPT. The data-processing topic file is empty.

**Prerequisites:** Bytes vs strings; a file is a sequence of bytes; Unicode characters (code points); the idea that an encoding maps between code points and bytes; CSV or text ingestion.

**Intuition:** A file is bytes. Text exists only after a decoder says how those bytes map to characters. The same bytes decoded with a different rule produce different text, and the program may not raise an error.

**Precise explanation:** Unicode assigns a number (code point) to each character. Encodings such as UTF-8, UTF-16, and Windows code page 1252 define how code points become bytes. When a program opens a file in text mode without naming an encoding, the decoder is chosen by the platform or locale, so the same code can behave differently on Linux and Windows. A mismatch often produces **mojibake**: text that looks plausible but contains wrong characters. Example: "Müller" encoded as UTF-8 and decoded as cp1252 shows as "MÃ¼ller". Row counts and file sizes do not change, so validation based on counts will not catch it.

**Grounding in the report:**
- *Report:* In Python 3.15, UTF-8 becomes the default for file I/O. Code that opens files without `encoding=` may read different text, "especially on Windows."
- *Report:* The release notes advise explicit encodings and an opt-in encoding warning. Run file-reading tests on Windows first.
- *Context:* The mojibake mechanism and the German-name example are general teaching context. The German example is relevant because client records at a German firm often contain umlauts.

**Realistic data-engineering example:** A pipeline reads a client export with `open(path)` on a Windows runner. The export was written as UTF-8. The default encoding there is cp1252, so names such as "Müller" and "Gößmann" are corrupted. A join on customer name then fails to match, and the match rate drops quietly by a few percent. The fix is `open(path, encoding="utf-8")` at the ingestion boundary, plus a validation check for common mojibake patterns and replacement characters.

**Why it matters:** Encoding errors are silent, cross-platform, and often appear only in the names and free-text fields that matter most for matching and reporting.

**Common misconception:** "UTF-8 is the default now, so I'm safe." The Python default changes, but every boundary (CSV exports, Excel, shell tools, databases, other languages) has its own default. A second misconception is that encoding problems are a Windows-only issue. Windows makes the failure more likely, but any mismatch between writer and reader causes it.

**Relationship to adjacent concepts:** C3 explains why the change appears as a silent behavior change in upgrades. C5 and the validation concepts should include encoding checks at the ingestion stage.

```text
bytes ──decode(encoding)──► text ──encode(encoding)──► bytes
        ↑ must be named explicitly at every boundary
```

**Recommended topic-file destination:** `learning/data-processing.md`, proposed subsection "Ingestion, file formats, and text." Rationale: this is a data-ingestion concept, not a Python-specific one. Cross-reference `learning/validation-and-data-quality.md` for validation checks (provisional, file not supplied).

**Recommended review-card destination:** `review-cards/data-processing.md`, ingestion section. Uncertainty: the file is empty.

### C5. Out-of-core execution and memory budgets

**Classification:** NEW CONCEPT. The data-processing topic file is empty.

**Prerequisites:** Memory hierarchy (RAM is fast and limited, disk is slower and larger); lazy vs eager evaluation; query plans; aggregation state (a group-by must keep a running result per key); the difference between streaming (bounded working set) and blocking operators (need the full input or state).

**Intuition:** A group-by needs a running total for each key. When the keys do not fit in RAM, the engine writes partial results to disk, processes the rest, and merges them later. The job finishes without crashing, but it pays in disk input and output and run time.

**Precise explanation:** Streaming execution processes data in batches with a bounded working set, provided each operator can work on batches. Blocking operators such as full sorts, high-cardinality group-bys, and some joins need more state than a batch. **Spilling** writes intermediate state to disk when a memory budget is exceeded. The budget limits what the engine's operators may hold, not necessarily total process memory, so the setting must be checked against what it actually bounds. Peak memory and run time trade against each other, so both must be measured.

**Grounding in the report:**
- *Report:* Polars 2.0 made the streaming engine the default for `collect()` and enabled out-of-core spill-to-disk. Polars 2.1 added out-of-core group-by.
- *Report:* The pilot suggests setting `POLARS_OOC_MEMORY_BUDGET_MB` to test spilling, and recording run time and peak memory.
- *Report:* Out-of-core behavior is listed as relevant to "larger-than-RAM" pipelines that fail with memory errors.

**Realistic data-engineering example:** A monthly job aggregates 40 GB of transaction CSVs by customer. The eager version exceeds the 32 GB VM and fails. A lazy scan with a streaming group-by completes within a bounded memory footprint, but takes about three times as long because of disk I/O. Whether that is acceptable is a scheduling and cost question, which is why both peak memory and run time belong in the test record.

**Why it matters:** Out-of-memory failures are reliability incidents. Moving to out-of-core execution changes performance characteristics and sometimes output order (see C3), so it is a behavior change as well as a resource fix.

**Common misconception:** "Out-of-core means unlimited data." Data is still bounded by disk and by the size of the aggregation state; very high-cardinality keys can still be expensive. A second misconception is that spilling is free. Spilling trades memory for I/O, and disk performance can dominate.

**Relationship to adjacent concepts:** C3 covers output-order changes that accompany execution-model changes. The concurrency topic file (not supplied) would cover parallel execution, which interacts with memory budgets when several jobs share a machine.

```text
Eager:    load all → compute → result          (needs RAM ≥ data)
Streaming: batch → partial state → spill if over budget → merge → result
```

**Recommended topic-file destination:** `learning/data-processing.md`, proposed subsection "Execution models and memory." Rationale: the execution model and memory management are general data-processing concepts that apply to Polars, Spark, DuckDB, and similar engines.

**Recommended review-card destination:** `review-cards/data-processing.md`, execution section. Uncertainty: the file is empty.

### C6. Typed structured outputs from models

**Classification:** NEW CONCEPT. The AI and LLM topic file is empty.

**Prerequisites:** JSON and schemas; classification vs free-text generation; the difference between a model-reported probability and a calibrated probability (basic statistics, covered further in C7); parsing failure modes; authentication (see C1).

**Intuition:** If the question has a closed set of answers, ask for an answer in that form. The system's output can then be checked mechanically, instead of being read by a person or extracted with a regular expression.

**Precise explanation:** A structured output is a response that conforms to a declared type, enforced by the service or by validation code. The report describes three shapes: a **predicate** (yes/no with a probability), a **choice** (one option from a fixed list), and a **score** (a position on an ordered rubric). Typing guarantees the shape of the answer, not its correctness. A wrong answer in the correct type is still wrong, and it will pass schema validation. A reported probability is useful only if, across many cases, events given probability p occur about p of the time on your data. That property is calibration, and it must be measured.

**Grounding in the report:**
- *Report:* Microsoft Foundry's Decisions API returns predicate, choice, and score answers, so "code does not have to parse prose." It is generally available as of 22 Oct 2026. It requires an Azure resource, a model deployment, and Entra ID authentication.
- *Report:* The vendor demo shows one case, not accuracy. "Probabilities must be checked against own labeled data." Refusals must go to a human.
- *Context:* The shape-versus-correctness distinction and calibration are general teaching context.

**Realistic data-engineering example:** Incoming client requests are routed into categories (Audit, Tax, Advisory, Other), and a predicate asks whether the text contains personal data. The routing code consumes only typed answers. A schema-valid "Advisory" for a request that is really an audit is not caught by the parser. It is caught only by comparing against a labeled set.

**Why it matters:** Typed outputs remove a large class of parsing bugs and make routing decisions testable and logged. They do not remove the need for evaluation, and they can create false confidence.

**Common misconception:** "Schema-valid means correct." It means well-formed. A second misconception is that the model's probability can be thresholded directly without checking calibration on your own data.

**Relationship to adjacent concepts:** C7 shows how to evaluate the typed outputs. C8 covers what the system is allowed to do with them.

```text
Free text ─► parse (fragile) ─► decision
Typed answer ─► validate shape ─► evaluate against labels ─► decision (after threshold review)
```

**Recommended topic-file destination:** `learning/ai-and-llm-systems.md`, proposed subsection "Structured outputs and AI-enabled workflows." Rationale: this is the AI-systems category, and the concept is reusable beyond one vendor.

**Recommended review-card destination:** `review-cards/ai-and-llm-systems.md`, structured-outputs section. Uncertainty: the file is empty.

### C7. Threshold selection and labeled-set evaluation

**Classification:** NEW CONCEPT. The statistics-and-evaluation topic file was not supplied, so this classification is provisional.

**Prerequisites:** Confusion matrix (true positives, false positives, false negatives, true negatives); precision and recall; base rates (how common the positive class is); sampling variability (estimates from small samples are uncertain).

**Intuition:** Setting a threshold means deciding which mistakes to accept. The right threshold depends on what each mistake costs, not on accuracy alone.

**Precise explanation:** For a classifier with a review threshold, the expected cost of a configuration is the sum of each error type times its cost:

$$
\text{Expected cost} = c_{FP} \cdot FP + c_{FN} \cdot FN
$$

Precision is $$\frac{TP}{TP + FP}$$ and recall is $$\frac{TP}{TP + FN}$$. Small samples are uncertain. With 200 labeled cases and 90% accuracy, a normal-approximation 95% interval is roughly ±4 percentage points. If the positive class is rare (for example, 5% of requests), 200 cases contain only about 10 positives. A recall estimate from 10 positives is extremely noisy, and the pilot cannot show much about that class without more positive examples.

**Grounding in the report:**
- *Report:* The pilot labels about 200 anonymized or synthetic requests from a past project, runs predicate and choice questions, and records false positives and false negatives.
- *Report:* "Set review threshold by cost of each error. No automatic action until error table reviewed."
- *Context:* The interval arithmetic, the base-rate point, and the cost formula are general teaching context, not report claims.

**Realistic data-engineering example:** The predicate "contains personal data" decides whether a request is sent to a model or routed to a manual queue. A missed personal-data case (false negative) is a compliance incident. An unnecessary manual review (false positive) is a cost. The threshold is therefore set to meet a recall target first, and the resulting review load is accepted. The labeled set must be stratified so that positive cases are represented, not just sampled at random.

**Why it matters:** A pilot is only as informative as its design. Without enough positive examples and a cost-based threshold, the pilot can report a high accuracy figure while missing the errors that matter.

**Common misconception:** "95% accuracy is good." On an imbalanced problem, a model that always predicts the majority class can score high accuracy while having zero recall on the important class. A second misconception is that one threshold fits all use cases. A third is that a vendor's demo accuracy transfers to your data.

**Relationship to adjacent concepts:** C6 produces the outputs being evaluated. C8 describes what happens after the threshold is set (human review or automation).

```text
Labeled set → predictions → confusion matrix (per class) → cost per threshold → chosen threshold → error table review → (only then) automation
```

**Recommended topic-file destination:** `learning/statistics-and-evaluation.md`, proposed subsection "Classification evaluation and thresholds." Rationale: the concepts are general statistics and evaluation methodology, reusable across any classifier. The file was not supplied, so this destination is provisional.

**Recommended review-card destination:** A review-card file for statistics-and-evaluation was not named in your input, so I cannot confirm that it exists. The closest supplied destination is `review-cards/ai-and-llm-systems.md`, evaluation section, as a temporary placement. Recommend re-routing once the statistics review-card file is confirmed.

### C8. Untrusted content and approval gates for tool-using agents

**Classification:** NEW CONCEPT. The AI and security topic files are empty.

**Prerequisites:** The agent loop (model reads input, requests a tool call, receives the result, and continues); tool permissions (read vs write); least privilege (C1); trust boundaries between data and instructions.

**Intuition:** Any text an agent reads can contain instructions. The agent cannot reliably separate instructions from data, so safety must come from limiting what the agent is allowed to do, not from trusting what it reads.

**Precise explanation:** **Prompt injection** is adversarial text inside data that the model processes, such as a web page, email, ticket, or document, that attempts to change the model's behavior. Filtering text is a weak control on its own. Stronger controls are architectural: restrict tool permissions (read-only roles), validate tool arguments outside the model, require human approval for side-effecting actions, log traces, and keep sessions isolated. Sensitive-content protection for traces, where available, is a separate control, and the report notes it is still in preview.

**Grounding in the report:**
- *Report:* Foundry Browser Automation is generally available, and agents run browser workflows through a Playwright Workspace and Toolbox MCP endpoint. The report's Watch rating states that "web pages are untrusted input and could carry prompt injection."
- *Report:* The recommended pilot sequence is a read-only lookup on a site you control, a comparison of returned fields with the source, a test with a page containing injected instructions, and human approval before any write.
- *Report:* For AI review in pull requests, "AI comments can be wrong. Does not replace human approval." For routing, refusals go to a human.
- *Context:* The architectural-control framing is general teaching context.

**Realistic data-engineering example:** An agent retrieves a contract status field from a supplier portal. The page contains hidden text saying to submit a form. A design that has only a browser read tool, a fixed extraction schema, no submit tool, and a human approval step for any write cannot be made to submit the form, however the page is written. A design that gives the agent a general browser with submit rights relies on the model resisting the injection, which is not a dependable control.

**Why it matters:** Each new tool expands what a successful injection can do. The blast radius is set by permissions, not by the model's behavior.

**Common misconception:** "Telling the model to ignore injected instructions is enough." Instructions are not a security boundary. A second misconception is that only malicious websites inject content. Tickets, emails, shared documents, and database fields written by users are all untrusted input.

**Relationship to adjacent concepts:** C1 determines what credentials the agent holds. C6 determines the shape of outputs that gates can check. C7 measures whether gates are working.

```text
Untrusted input ─► model proposes action ─► validator (outside model) ─► read-only? execute : human approval ─► execute ─► log
```

**Recommended topic-file destination:** `learning/ai-and-llm-systems.md`, proposed subsection "Agents, tool use, and untrusted input." Cross-reference `learning/security.md` for permissions and trust boundaries. Rationale: the agent-architecture aspect belongs to AI systems; the permission model belongs to security. Both files are empty.

**Recommended review-card destination:** `review-cards/ai-and-llm-systems.md`, agents section; cross-reference `review-cards/security.md`. Uncertainty: both files are empty.

### Considered but not selected

- **Copilot code review in Azure Repos:** The product is specific. The general lesson (AI review supplements and does not replace human approval) is covered in C8.
- **Foundry Local GPU runtimes (CUDA, cuDNN, execution providers, `AddDllDirectory`):** Hardware- and vendor-specific. If the Foundry Local pilot proceeds, a general "native runtime dependency matrix" concept could be considered later, possibly under `learning/deployment-and-infrastructure.md`.
- **Playwright and browser automation mechanics:** Tool-specific. The concept-level lesson is covered in C8.
- **Model Context Protocol (MCP):** The report uses MCP terms without explaining them. It may warrant a future entry under `learning/apis-and-networking.md`, but that file was not supplied, so I did not classify it.
- **Azure DevOps PAT retirement date:** Covered as the worked example in C1, not as a separate concept.

## Proposed permanent-resource changes

These are proposals only. Nothing has been created or modified.

**Learning-resource index (`learning/README.md`)**
- Consider adding named subsections to the existing topic scopes so that future concepts route correctly:
  - `security.md`: "Credentials and secrets" (C1); "Software supply chain" (C2).
  - `deployment-and-infrastructure.md`: "Version management and upgrades" (C3).
  - `data-processing.md`: "Ingestion, file formats, and text" (C4); "Execution models and memory" (C5).
  - `ai-and-llm-systems.md`: "Structured outputs and AI-enabled workflows" (C6); "Agents, tool use, and untrusted input" (C8).
  - `statistics-and-evaluation.md`: "Classification evaluation and thresholds" (C7), provisional until the file is reviewed.
- Consider extending the scope descriptions for `security.md` (supply chain) and `validation-and-data-quality.md` (behavioral regression testing) to reflect these concepts.
- No new topic files are proposed. Each concept fits an existing category.

**Topic files (recommended, not created)**
- `security.md`: add "Credentials and secrets" (C1) and "Software supply chain" (C2).
- `deployment-and-infrastructure.md`: add "Version management and upgrades" (C3).
- `data-processing.md`: add "Ingestion, file formats, and text" (C4) and "Execution models and memory" (C5).
- `ai-and-llm-systems.md`: add "Structured outputs and AI-enabled workflows" (C6) and "Agents, tool use, and untrusted input" (C8).
- `statistics-and-evaluation.md`: add "Classification evaluation and thresholds" (C7). This file was not supplied, so verify its contents before finalizing.
- `validation-and-data-quality.md` (not supplied): consider a cross-reference to C3 for regression-testing technique.

**Review-card files (recommended, not created)**
- `review-cards/security.md`: cards for C1 and C2.
- `review-cards/deployment-and-infrastructure.md`: card for C3.
- `review-cards/data-processing.md`: cards for C4 and C5.
- `review-cards/ai-and-llm-systems.md`: cards for C6 and C8.
- C7: no confirmed destination. Re-route once the statistics review-card file is confirmed.

## Proposed review cards

**Card C1: Least-privilege credentials, expiry, and rotation**
- **Recall:** A credential's risk equals its scope times its lifetime, and rotation means updating every consumer before revoking the old credential.
- **Mental hook:** A key that opens every door in the building, and never expires.
- **Why I care:** Global PATs retire on 27 Jan 2027, and a leaked credential is only as dangerous as what it can reach.
- **Contrast:** Rotation (replacing a credential) vs revocation (disabling it). Rotation without consumer discovery breaks production.
- **Tiny example:** A variable group holds a global PAT. Replace it with a workload identity token obtained at runtime, then delete the stored PAT.
- **Test yourself:** Why is an inventory of consumers the first step in rotation, not the last?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** `review-cards/security.md`, credentials section (file empty).

**Card C2: Supply-chain integrity**
- **Recall:** A lockfile and hashes say which bytes you installed; provenance says where they were built, not whether they are safe.
- **Mental hook:** A sealed package with a shipping label, not a safety inspection.
- **Why I care:** Inventory across images and lockfiles is how you answer "were we running that version?"
- **Contrast:** Pinning (fixed versions) vs range-based installs (automatic updates). Both carry risk.
- **Tiny example:** `pip install litellm>=1.100` on Monday and Friday can install different versions. A lockfile makes both builds identical.
- **Test yourself:** A package has valid provenance from its official repository. What does that prove, and what does it not prove?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** `review-cards/security.md`, supply-chain section (file empty).

**Card C3: Breaking changes and regression testing**
- **Recall:** Breaks come in three kinds: interface, runtime requirement, and behavior. Behavior breaks pass signature checks and need output comparison.
- **Mental hook:** A version number is a promise you verify with a diff.
- **Why I care:** Polars and Python 3.15 changes can alter outputs without any error.
- **Contrast:** A loud break (import fails) vs a silent break (same code, different rows).
- **Tiny example:** Run last month's inputs through old and new versions, normalize row order, then compare the raw outputs as well.
- **Test yourself:** Why might a major-version bump be insufficient warning for a behavior change?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md`, version-management section (file empty).

**Card C4: Text encoding and the bytes/text boundary**
- **Recall:** Bytes become text only through an explicit encoding. Mismatches can succeed silently and corrupt characters.
- **Mental hook:** The same bag of letters, read with the wrong glasses.
- **Why I care:** Python 3.15 changes the default, but other boundaries still differ, and umlauts in client names are common.
- **Contrast:** Character count vs byte count. Row counts can stay the same while text is corrupted.
- **Tiny example:** `open(path, encoding="utf-8")` at the ingestion boundary.
- **Test yourself:** Why would a row-count check fail to catch "Müller" becoming "MÃ¼ller"?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** `review-cards/data-processing.md`, ingestion section (file empty).

**Card C5: Out-of-core execution and memory budgets**
- **Recall:** Out-of-core execution trades disk I/O for bounded memory. Measure both peak memory and run time.
- **Mental hook:** A running tally that gets written to a notebook on the desk when the desk fills up.
- **Why I care:** Out-of-memory failures are reliability incidents, and the fix changes performance and possibly output order.
- **Contrast:** Streaming (bounded working set) vs blocking operators (need full state).
- **Tiny example:** A 40 GB group-by on a 32 GB machine: eager fails; streaming completes at higher run time.
- **Test yourself:** Why does an out-of-core group-by not remove the cost of high-cardinality keys?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** `review-cards/data-processing.md`, execution section (file empty).

**Card C6: Typed structured outputs from models**
- **Recall:** A typed answer guarantees shape, not correctness. Probabilities are useful only when calibrated on your data.
- **Mental hook:** A form with checkboxes, which can still be ticked wrongly.
- **Why I care:** Typed routing is easier to test and log, but schema validation alone does not show the routing is right.
- **Contrast:** Free text parsed with regex (fragile) vs typed answer validated against labels (testable).
- **Tiny example:** A predicate "contains personal data" returns a valid yes/no. Only a labeled set shows whether the yes/no is right.
- **Test yourself:** What evidence would show that a model's reported 0.9 probability really means about 90%?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** `review-cards/ai-and-llm-systems.md`, structured-outputs section (file empty).

**Card C7: Threshold selection and labeled-set evaluation**
- **Recall:** Choose the threshold by the cost of each error type, and check that the labeled set contains enough positive cases to estimate the errors that matter.
- **Mental hook:** A smoke detector set by how expensive a missed fire is versus a false alarm.
- **Why I care:** The pilot can look successful while missing rare, important errors.
- **Contrast:** Accuracy (overall correctness) vs recall (share of true positives found).
- **Tiny example:** A 5% positive class in 200 cases leaves about 10 positives, so recall is very uncertain.
- **Test yourself:** Why might a 95% accuracy be useless for a rare-class routing decision?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** Provisional: `review-cards/ai-and-llm-systems.md`, evaluation section (file empty). Re-route to the statistics review-card file once it is confirmed.

**Card C8: Untrusted content and approval gates for agents**
- **Recall:** Any text an agent reads can contain instructions. Safety comes from limited tool permissions, validation outside the model, and approval before writes.
- **Mental hook:** A visitor badge that opens only the lobby, not the server room.
- **Why I care:** Each new tool expands what a successful injection can do.
- **Contrast:** Telling the model to ignore instructions (weak) vs removing the write permission (strong).
- **Tiny example:** An agent with read-only browser access and a fixed extraction schema cannot submit a form, whatever the page says.
- **Test yourself:** Which permission should an agent lack if it only needs to read a supplier portal?
- **First learned date:** 2026-11-09
- **Recommended review-card destination:** `review-cards/ai-and-llm-systems.md`, agents section; cross-reference `review-cards/security.md` (both files empty).

## Questions worth discussing

1. How would you design a rotation drill that proves every consumer of a credential was found, not just the ones listed in the inventory?
2. When does provenance give real assurance, and what should a team do if the build pipeline that produced a signed artifact was itself compromised?
3. Which silent behavior changes in the Polars and Python 3.15 notes would your existing tests catch, and which would need output comparison against golden data?
4. For a rare-class routing decision, how many positive cases would you need in the labeled set before you trust a recall estimate, and what interval would you accept?
5. What memory-budget policy makes sense for a shared VM where several pipelines run concurrently, and who should own that setting?
6. How should an agent with a write-capable tool separate reading untrusted text from deciding to act, and where should the human approval point sit?
