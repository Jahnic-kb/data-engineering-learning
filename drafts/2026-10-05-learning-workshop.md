# Learning workshop — 2026-10-05

## Why these concepts matter this week

**DRAFT for discussion with a Learning Coach; no permanent resources have been changed.**

This report highlights high-impact risks in security, data correctness, asynchronous and parallel execution, deployment compatibility, and observability. The release notes are useful signals, but the reusable learning is broader: how to keep SQL values separate from executable code, preserve data semantics across library boundaries, design for concurrency failures, and verify that runtime configuration behaves as intended.

The report does **not** include Kienbaum’s installed versions, so exposure is unknown. Check lockfiles and deployed images before deciding whether a specific upgrade applies. The supplied relevant topic and review-card files exist but contain no concept material, so comparison against finalized learning is incomplete; the concepts below are treated as **NEW CONCEPTS**, not as replacements for existing entries.

The asyncmy security item needs special care: the report says the vulnerable helper was removed in 0.2.12, but the advisory lists no patched version. Do not treat the vulnerability as formally patched based on helper removal alone.

## Learning map

```text
SQL values vs. executable SQL
↓
Parameterized queries
↓
Reduced SQL injection risk

Relationship predicates
↓
ORM loading strategy
↓
Correct related-row results

Parsing and memory ownership
↓
DataFrame semantic correctness
↓
Assertions and regression tests

Worksheet replacement semantics
↓
No stale trailing rows in generated reports

Async event signaling
↓
Timeouts, cancellation, pool lifecycle
↓
Liveness and reliable shutdown

Worker processes and runtime environment
↓
Parallel job reliability
↓
Reproducible deployment

Optional telemetry configuration
↓
Graceful degradation at startup
↓
Observable—but not necessarily fully instrumented—service
```

## Concepts

### 1. Parameterized SQL and SQL injection

- **Classification:** **NEW CONCEPT** — The supplied `security.md` and `databases.md` files are empty. This is a reusable security boundary relevant to any application that constructs SQL from external inputs.
- **Prerequisites:** Basic SQL statements; distinction between data values and SQL syntax.
- **Release evidence:** The report says asyncmy 0.2.12 removed `escape_dict` because dictionary keys could reach SQL unescaped. The cited advisory describes SQL injection through crafted keys, rates it Critical, and lists affected versions as `<= 0.2.11` with **Patched versions: None**. The report recommends replacing the call path with parameterized SQL and confirming remediation. [asyncmy release notes](https://github.com/long2ice/asyncmy/releases) · [GitHub Advisory Database: CVE-2025-65896](https://github.com/advisories/GHSA-qhqw-rrw9-25rm)
- **Intuition:** Keep the *query plan or template* separate from the *values*. A user-provided value should be treated as data, not allowed to change the structure of the SQL statement.
- **Precise explanation:** Parameterized queries send SQL structure and bound values separately to the database driver. Escaping is not a substitute for this separation: custom escaping can be incomplete, context-sensitive, or bypassed when code handles an unexpected input shape. Placeholders generally bind values, not table or column identifiers; dynamic identifiers need a strict allowlist and safe identifier handling.
- **Example:** Use the driver’s parameter-binding API rather than interpolating an input into a query. Placeholder syntax varies by driver.
  ```python
  await cursor.execute(
      "SELECT customer_id FROM customers WHERE tenant_id = %s",
      (tenant_id,),
  )
  ```
- **Why it matters:** Injection can turn a lookup into an unauthorized query or data modification. The report’s specific concern is untrusted dictionary keys reaching an unsafe helper; whether Kienbaum has that path is unknown.
- **Common misconception:** “The values look harmless” or “we escaped the values” is enough. A secure design relies on the database driver’s parameter binding for values and a separate allowlist for any dynamic SQL identifiers.
- **Relationship to adjacent concepts:** Complements authorization and least privilege. Parameterization protects query structure; database permissions limit the damage if another defect remains.
- **Recommended topic-file destination:** `learning/security.md` — a section on SQL injection prevention and separating code from data. This is a natural security topic; no new file is needed.
- **Recommended review-card destination:** `review-cards/security.md` — corresponding SQL injection / parameterized-query card. The file exists but is empty.

### 2. TLS negotiation and fail-closed security configuration

- **Classification:** **NEW CONCEPT** — No finalized material is present in the supplied security or networking files. Secure transport configuration is broadly reusable across database drivers and other clients.
- **Prerequisites:** Basic client/server communication; what it means for traffic to be encrypted in transit.
- **Release evidence:** The report says asyncmy 0.2.14 changed `ssl=True` to build a default TLS context and negotiate TLS. It also says invalid SSL arguments now raise `ValueError` rather than silently disabling TLS. The report recommends checking configuration and verifying encrypted handshakes. [asyncmy release notes](https://github.com/long2ice/asyncmy/releases)
- **Intuition:** A security setting should produce the promised protection—or make the failure visible. It should not silently fall back to a less secure connection.
- **Precise explanation:** TLS negotiation establishes an encrypted transport session. Correct configuration also matters: a TLS connection that fails to verify the server certificate may not provide the intended server identity assurance. “Fail closed” means that invalid or unavailable security configuration causes a clear failure rather than silently continuing without protection.
- **Example:** In an integration test, connect using the production-like TLS settings, verify that the server reports an encrypted session, and separately verify that invalid TLS configuration fails visibly. The exact inspection API depends on the driver and database.
- **Why it matters:** The report confirms plaintext behavior under a particular earlier asyncmy configuration. If that configuration was used, database credentials and query traffic could have crossed the connection unencrypted. Kienbaum exposure is unknown.
- **Common misconception:** “I set `ssl=True`, so the connection must be secure.” A flag is an input to configuration; the negotiated connection and certificate-verification behavior still need verification.
- **Relationship to adjacent concepts:** TLS protects the transport; authentication and authorization govern who may connect and what they may do. They are complementary controls.
- **Recommended topic-file destination:** `learning/security.md` — a section on secure transport and fail-closed security configuration. `apis-and-networking.md` could be cross-referenced, but security is the closest primary fit.
- **Recommended review-card destination:** `review-cards/security.md` — corresponding TLS / fail-closed configuration card. The supplied file is empty.

### 3. ORM relationship filters must survive loading-strategy changes

- **Classification:** **NEW CONCEPT** — `databases.md` is empty, and the report points to a reusable correctness issue: an ORM loading strategy must not change which related rows satisfy a relationship’s criteria.
- **Prerequisites:** Basic relational joins and predicates; parent/child relationships; familiarity with eager loading is helpful.
- **Release evidence:** SQLAlchemy 2.1.2 fixes a regression where many-to-many `selectinload()` could ignore additional `relationship.primaryjoin` criteria and include related rows that should have been excluded. The report recommends testing those relationship filters and updating the 2.1 series. [SQLAlchemy 2.1 changelog](https://www.sqlalchemy.org/docs/latest/changelog/changelog_21.html#change-2.1.2-orm)
- **Intuition:** Fetching related objects in a different number of SQL queries should change *how* rows are loaded, not *which* rows are valid members of the relationship.
- **Precise explanation:** An ORM maps relationship semantics onto SQL joins and filters. A predicate in a relationship definition constrains the related rows that qualify. Eager-loading strategies may issue different SQL shapes, so correctness should be verified at the result level rather than assumed from the selected loader option.
- **Example:** Suppose a user’s groups relationship is constrained to active memberships. Test that the returned group IDs include only active memberships whether the relationship is loaded lazily or with `selectinload()`. Compare the result to an independently written SQL query or a known expected set.
- **Why it matters:** An over-inclusive relationship result can leak or misattribute data, even when the parent query appears correct. The report confirms the upstream regression but does not confirm Kienbaum exposure or identify the first affected version.
- **Common misconception:** “Eager loading is only a performance choice.” It is intended to affect loading behavior, but regressions in query generation can affect result correctness.
- **Relationship to adjacent concepts:** Connects relational predicate semantics to ORM abstractions and regression testing. Test both the *set of returned IDs* and the query path that produced them.
- **Recommended topic-file destination:** `learning/databases.md` — a section on ORM relationship predicates and eager-loading correctness.
- **Recommended review-card destination:** `review-cards/databases.md` — corresponding ORM relationship-filter card. The file exists but is empty.

### 4. DataFrame transformations need semantic checks at parsing and memory boundaries

- **Classification:** **NEW CONCEPT** — The supplied data-processing and validation files are empty. The reusable concept is validating data semantics where parsing, dtype-specific operations, and memory ownership can affect results.
- **Prerequisites:** Basic DataFrame operations; awareness that columns have dtypes and that some libraries use optional Arrow-backed storage.
- **Release evidence:** pandas 3.0.6 fixes several regressions: separator inference with `read_csv(sep=None)`; unsupported engines silently treating each line as one column; PyArrow-backed linear interpolation leaving missing values or truncating integer results; `RangeIndex` corruption after external mutation because of missing Copy-on-Write tracking; and a PyArrow-backed assignment that could share memory unexpectedly. The report says affected-version ranges are not specified for each issue. [pandas 3.0.6 release notes](https://pandas.pydata.org/docs/whatsnew/v3.0.6.html)
- **Intuition:** A successful function call is not proof that the data means what you think it means. Check structure, types, values, and whether an operation changed or shared data unexpectedly.
- **Precise explanation:** These are different failure modes at library boundaries:
  - **Parsing:** delimiter inference and engine support determine the resulting columns and rows.
  - **Dtype-specific computation:** missing-value and numeric behavior may differ with the storage backend and dtype.
  - **Memory ownership:** views, copies, and external mutation can affect whether changes are visible elsewhere. Copy-on-Write is a behavioral contract, not a reason to assume every external mutation is tracked.
  
  The release report identifies the affected examples; the general teaching point is to test observable output contracts rather than just whether code completed.
- **Example:** For an ingestion pipeline, assert the expected column names and count after parsing; for an interpolation step, assert representative values, missingness, and dtype; after assignment, verify whether the source object is intended to remain unchanged.
  ```python
  df = pandas.read_csv(path, sep=None, engine="python")
  assert list(df.columns) == expected_columns
  assert len(df.columns) > 1
  ```
- **Why it matters:** These regressions can produce silent wrong results rather than a clear exception. That makes schema and value checks especially important before downstream aggregation or reporting.
- **Common misconception:** “No exception means the DataFrame is correct.” Parsing can yield a structurally wrong but valid DataFrame; a calculation can return values with unexpected missingness or dtype.
- **Relationship to adjacent concepts:** Connects data processing with validation and data quality. Runtime pinning reduces exposure, while semantic tests detect wrong outputs that version checks alone cannot catch.
- **Recommended topic-file destination:** `learning/data-processing.md` — a section on DataFrame parsing, dtype/backend semantics, and memory ownership. Add test examples that assert output contracts rather than relying on successful execution.
- **Recommended review-card destination:** `review-cards/data-processing.md` — corresponding DataFrame semantic-correctness card. The supplied file is empty.

### 5. Replacing a dataset is different from writing over its first rows

- **Classification:** **NEW CONCEPT** — The supplied data-processing and validation files have no content. The report’s stale-row scenario illustrates a general replacement-semantics problem in generated reports.
- **Prerequisites:** Basic tabular data and spreadsheet worksheets; understanding of recurring or scheduled report generation.
- **Release evidence:** `openxlsx` 4.2.9 adds opt-in `overwrite = TRUE` to `writeData()`. The report says existing worksheet cell data is cleared before writing, preventing old trailing rows from remaining when the replacement dataset is shorter. The flag defaults to `FALSE`; the report recommends piloting it on intended full-sheet rewrites and checking styles, formulas, and other workbook content. [openxlsx NEWS](https://cran.r-project.org/web/packages/openxlsx/news/news.html) · [Implementation and example](https://github.com/ycphs/openxlsx/pull/536)
- **Intuition:** Writing 100 new rows over the top of last week’s 150 rows does not automatically mean the last 50 old rows disappeared.
- **Precise explanation:** A write operation may update only the cells covered by the new data. If the new dataset has fewer rows, cells beyond that range can retain prior values unless the operation explicitly clears the intended range. A clean replacement should define the scope of what is removed as well as what is written.
- **Example:** In a test workbook, write a longer dataset, then write a shorter dataset with `overwrite = TRUE`. Confirm that the former tail is cleared and that expected formulas, styles, and unrelated workbook content remain intact.
  ```r
  openxlsx::writeData(
    wb, sheet = "Report", x = current_data, overwrite = TRUE
  )
  ```
- **Why it matters:** Stale rows can make a report appear to contain current records when they actually belong to an older, longer output. The report describes a documented upstream scenario, not a confirmed Kienbaum incident.
- **Common misconception:** “The new data replaced the old data because it started at the same cell.” The write range and clearing behavior determine whether old cells remain.
- **Relationship to adjacent concepts:** Related to idempotent pipeline outputs and data-quality checks. “Run the export twice” and “export a smaller dataset” are useful tests for stale-state defects.
- **Recommended topic-file destination:** `learning/data-processing.md` — a section on replacement semantics and stale data in generated outputs. This is a natural fit for data-output processing; no new file is warranted.
- **Recommended review-card destination:** `review-cards/data-processing.md` — corresponding worksheet replacement / stale-tail card. The supplied file is empty.

### 6. Async I/O reliability depends on event signaling and lifecycle correctness

- **Classification:** **NEW CONCEPT** — There is no finalized material in the supplied concurrency or database files. The report exposes general concurrency concerns: races, notification of waiters, cancellation, and resource-pool shutdown.
- **Prerequisites:** Basic asynchronous execution; familiarity with timeouts, tasks, and connection pools is helpful.
- **Release evidence:** asyncmy 0.2.15 fixes a `read_timeout` timing race that could lose data, EOF, or error notifications; pool shutdown and cancellation hangs; and pool waiters not being notified after a connection attempt fails. It also changes writes to closed transports to raise `OperationalError`, which allows SQLAlchemy `pool_pre_ping` to recognize and replace a disconnected connection. The report recommends exercising timeout, cancellation, shutdown, failed connection creation, and reconnect paths. [asyncmy 0.2.15 release notes](https://github.com/long2ice/asyncmy/releases/tag/v0.2.15)
- **Intuition:** In asynchronous systems, it is not enough for the operation eventually to fail; all waiting tasks must be notified, and every shutdown path must release resources and finish.
- **Precise explanation:** An event loop coordinates tasks that suspend while waiting for I/O. A timing race can occur when a connection’s state changes near a timeout or cancellation boundary. If the state change is not delivered to the waiting task, the task may wait indefinitely or miss the relevant data/error signal. Pool lifecycle logic also needs to account for tasks waiting to acquire connections, connection attempts that fail, and cancellation during shutdown. `pool_pre_ping` can detect certain stale connections at checkout; it does not make every failure during active use impossible.
- **Example:** A database integration test can: force a read timeout; cancel a task while it waits for a connection; make a connection attempt fail while other tasks wait; and shut down the pool. Assert that each task completes or raises within a deadline and that later work can acquire a healthy connection.
- **Why it matters:** A lost notification may appear as a hung request rather than a clean error. Shutdown hangs can stall deployments or worker termination. The report confirms these upstream conditions; it does not confirm Kienbaum exposure.
- **Common misconception:** “Async means non-blocking, so it cannot hang.” Async code can still deadlock or wait indefinitely if state transitions, notifications, and cancellation are mishandled.
- **Relationship to adjacent concepts:** Related to race conditions, resource lifecycle management, timeouts, and resilience. A timeout limits waiting; correct notification and cleanup make recovery possible.
- **Recommended topic-file destination:** `learning/concurrency-and-parallelism.md` — a section on async I/O signaling, cancellation, and liveness in pooled resources. Connection-pool details can be cross-referenced from `databases.md`.
- **Recommended review-card destination:** `review-cards/concurrency-and-parallelism.md` — corresponding async I/O / pool-lifecycle card. The supplied file is empty.

### 7. Parallel workers have runtime and environment assumptions

- **Classification:** **NEW CONCEPT** — The supplied concurrency and deployment files are empty. This is a reusable lesson about process-based parallel work: worker configuration and execution environment are part of the job’s behavior.
- **Prerequisites:** Basic distinction between sequential and parallel execution; awareness that multisession or cluster workers may run in separate processes.
- **Release evidence:** `future` 1.76.0 fixes crashes when deprecated `gc = TRUE` is used with `multisession` or `cluster` backends. It also fixes failures when the current working directory cannot be entered, a regression introduced in 1.40.0. The report recommends checking calls, testing worker startup and task execution, and upgrading if applicable. [future NEWS](https://cran.r-project.org/web/packages/future/news/news.html) · [CRAN package index](https://cran.r-project.org/web/packages/future/index.html)
- **Intuition:** A worker is not simply the parent process running faster. It has its own startup and execution context, and a job can fail because of configuration or assumptions inherited from the parent.
- **Precise explanation:** Process-based parallel backends create or communicate with worker processes. Deprecated or backend-specific arguments can fail differently in those workers. A task that assumes a particular working directory can also fail if a worker cannot enter that directory. Reproducible parallel execution should make required inputs and paths explicit and test both worker startup and task execution.
- **Example:** Run a small representative `future` task using the production backend in CI. Check that inputs are available without relying on an implicit working directory, and check for deprecated arguments such as `gc = TRUE`.
- **Why it matters:** A pipeline can work sequentially but fail when deployed with a multisession or cluster backend. The report identifies upstream crash and working-directory cases; it does not establish Kienbaum exposure.
- **Common misconception:** “If the code works in the interactive session, it will work on the worker.” A worker may have a different process state, startup path, or backend behavior.
- **Relationship to adjacent concepts:** Connects parallelism to deployment reproducibility and environment configuration. It is distinct from asynchronous I/O: workers execute tasks concurrently in processes, while async tasks generally coordinate around I/O within an event-loop model.
- **Recommended topic-file destination:** `learning/concurrency-and-parallelism.md` — a section on worker execution contexts, backend-specific configuration, and parallel tests. Cross-reference deployment concerns rather than creating a separate file.
- **Recommended review-card destination:** `review-cards/concurrency-and-parallelism.md` — corresponding worker-environment card. The supplied file is empty.

### 8. Native package compatibility is a runtime property, not just a source-code property

- **Classification:** **NEW CONCEPT** — `deployment-and-infrastructure.md` is empty. Binary compatibility across the Python runtime, NumPy, and compiled package wheels is broadly relevant to data environments.
- **Prerequisites:** Basic understanding that packages have versions; awareness that some packages include compiled native code.
- **Release evidence:** pandas 3.0.5 fixes a segmentation fault in pandas 3.0.4 wheels on Python 3.14, attributed in the report to a build against an incompatible NumPy version. The report recommends checking Python and pandas versions together and upgrading to 3.0.5 or later where applicable. [pandas 3.0.5 release notes](https://pandas.pydata.org/docs/whatsnew/v3.0.5.html)
- **Intuition:** A package can be “installed” and still be incompatible with the runtime and binary libraries it was built against.
- **Precise explanation:** Compiled extensions depend on interfaces and binary assumptions provided by the interpreter and libraries such as NumPy. A mismatch can fail at import, raise runtime errors, or—in severe cases—crash the process, bypassing ordinary Python exception handling. A wheel’s compatibility with a runtime is therefore part of the deployment artifact, not merely a source dependency concern.
- **Example:** In CI, build and test the exact Python/pandas/NumPy combination used in the container image. Include a representative import and pipeline test, then verify the same locked environment in the deployed image.
- **Why it matters:** A segmentation fault terminates the process, potentially interrupting a pipeline without the normal exception-handling and cleanup path. The report’s scope is the specified pandas 3.0.4 wheel and Python 3.14 combination; Kienbaum’s runtime and pins are unknown.
- **Common misconception:** “If pip or the package manager installed it successfully, the combination is runtime-safe.” Installation success is not equivalent to exercising the compiled code paths.
- **Relationship to adjacent concepts:** Related to lockfiles, build provenance, and reproducible deployment. Version checks narrow risk; representative tests validate behavior in the actual runtime.
- **Recommended topic-file destination:** `learning/deployment-and-infrastructure.md` — a section on compiled package compatibility and testing the Python/runtime dependency matrix.
- **Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md` — corresponding native-extension compatibility card. The supplied file is empty.

### 9. Optional observability failures and graceful startup

- **Classification:** **NEW CONCEPT** — The supplied observability and API files are empty. The reusable concept is deciding whether a non-core startup dependency should block service availability, and making any degraded mode visible.
- **Prerequisites:** Basic API startup lifecycle; understanding that telemetry includes signals such as traces and metrics.
- **Release evidence:** FastAPI 0.142.2 allows application startup to proceed when automatic OpenTelemetry configuration fails. The report recommends testing startup with both valid telemetry configuration and an unavailable or invalid exporter. It confirms the upstream failure mode but no Kienbaum incident. [FastAPI 0.142.2 release note](https://github.com/fastapi/fastapi/releases/tag/0.142.2)
- **Intuition:** A service may be able to serve requests without telemetry, but “started successfully” does not mean “fully observable.”
- **Precise explanation:** Startup policy depends on whether a component is essential to the service’s correctness or merely supports operations. If telemetry is best-effort, allowing startup to continue can preserve service availability during exporter/configuration failure. That choice should not hide the failure: operators may need a log, metric, health signal, or other indication that telemetry is degraded. If telemetry is required for a specific compliance or operational guarantee, failing startup may be appropriate instead.
- **Example:** Test startup once with a valid exporter and once with an unreachable or invalid exporter. In the failure case, verify that the API’s intended routes start if appropriate, and verify that the telemetry setup problem is still diagnosable.
- **Why it matters:** A startup failure caused by nonessential telemetry can reduce availability; silently missing telemetry can reduce incident-detection capability. The right policy depends on service requirements, not simply on whether startup succeeds.
- **Common misconception:** “Graceful startup means the telemetry problem is solved.” It means the application can proceed; instrumentation may still be absent or degraded.
- **Relationship to adjacent concepts:** Connects observability to reliability and fault isolation. It is a policy decision about dependency criticality and degraded operation, not a guarantee that every startup error should be ignored.
- **Recommended topic-file destination:** `learning/observability-and-reliability.md` — a section on optional observability dependencies, startup failure policy, and degraded-mode signaling. Cross-reference API lifecycle material in `apis-and-networking.md`.
- **Recommended review-card destination:** `review-cards/observability-and-reliability.md` — corresponding telemetry startup / graceful-degradation card. The supplied file is empty.

## Proposed permanent-resource changes

These are **proposals only**; no permanent files have been changed.

- **Learning-resource index (`learning/README.md`):** No taxonomy or new topic file is recommended. The existing categories adequately route every concept. The index does not list individual concepts, so adding concept-level entries is not necessary.
- **`learning/security.md`:** Add reviewed sections on (1) parameterized SQL and injection prevention, including the distinction between bound values and dynamic identifiers; and (2) TLS negotiation, certificate verification, and fail-closed configuration.
- **`learning/databases.md`:** Add a section on ORM relationship predicates and eager-loading correctness, with tests that compare expected related-row IDs across loading strategies.
- **`learning/data-processing.md`:** Add sections on (1) DataFrame semantic checks across parsing, dtype/backend operations, and memory ownership; and (2) replacement semantics in generated outputs, including stale trailing data and tests for shorter replacement datasets.
- **`learning/concurrency-and-parallelism.md`:** Add sections on (1) async I/O signaling, timeouts, cancellation, and pooled-resource liveness; and (2) process-worker execution context, backend-specific configuration, and testing worker startup and tasks.
- **`learning/deployment-and-infrastructure.md`:** Add a section on compiled package/runtime compatibility and validating the exact interpreter and binary dependency combination in deployment.
- **`learning/observability-and-reliability.md`:** Add a section on optional telemetry startup failures, degraded-mode visibility, and choosing whether telemetry configuration should block service startup.
- **Review cards:** Add the corresponding proposed cards to `review-cards/security.md`, `review-cards/databases.md`, `review-cards/data-processing.md`, `review-cards/concurrency-and-parallelism.md`, `review-cards/deployment-and-infrastructure.md`, and `review-cards/observability-and-reliability.md`. These existing files are empty; no new review-card file is proposed.

## Proposed review cards

All cards below are drafts for human review. First learned date uses the supplied run date, **2026-10-05**.

### Card 1 — Parameterized SQL and injection prevention

- **Recall:** How should application values be supplied to SQL?
- **Mental hook:** “Keep data out of the SQL grammar.”
- **Why I care:** Prevents untrusted values from changing query meaning.
- **Contrast:** Parameterize values; allowlist dynamic identifiers.
- **Tiny example:** `execute("... WHERE tenant_id = %s", (tenant_id,))`
- **Test yourself:** Why can’t a value placeholder safely stand for a table name?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/security.md` — SQL injection / parameterized queries.

### Card 2 — TLS and fail-closed configuration

- **Recall:** What should happen when required TLS configuration is invalid?
- **Mental hook:** “Secure or stop—not silent plaintext.”
- **Why I care:** Prevents accidental exposure of credentials and database traffic.
- **Contrast:** A configured TLS flag is not proof of a verified encrypted session.
- **Tiny example:** Test that TLS is negotiated and invalid TLS settings raise an error.
- **Test yourself:** What does checking the connection prove that checking a config flag does not?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/security.md` — TLS / fail-closed security configuration.

### Card 3 — ORM relationship-filter correctness

- **Recall:** Should a different eager-loading strategy change which related rows qualify?
- **Mental hook:** “Change the fetch plan, not the relationship meaning.”
- **Why I care:** Prevents over-inclusive or misattributed ORM results.
- **Contrast:** Loading strategy is about how data is fetched; relationship predicates define what qualifies.
- **Tiny example:** Compare group IDs loaded with `selectinload()` against expected active-membership IDs.
- **Test yourself:** What should you assert besides “the query completed”?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/databases.md` — ORM relationship filters and loading correctness.

### Card 4 — DataFrame semantic checks

- **Recall:** What should be checked after parsing or a dtype-specific transformation?
- **Mental hook:** “A returned DataFrame is not proof of correct data.”
- **Why I care:** Parsing, missing-value handling, or memory sharing can silently alter results.
- **Contrast:** Successful execution is not the same as a correct schema and values.
- **Tiny example:** Assert expected columns after separator inference; assert interpolation values and missingness.
- **Test yourself:** Which outputs—columns, values, dtypes, index, or mutation behavior—matter for your pipeline?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/data-processing.md` — DataFrame semantic correctness.

### Card 5 — Worksheet replacement semantics

- **Recall:** What happens to old rows beyond a shorter new write?
- **Mental hook:** “Overwrite the intended range, not just the new prefix.”
- **Why I care:** Prevents stale records from remaining in recurring reports.
- **Contrast:** Writing new cells is not automatically equivalent to clearing old output.
- **Tiny example:** Write a long dataset, then a shorter one with `overwrite = TRUE`; inspect the old tail and workbook formatting.
- **Test yourself:** What unrelated workbook content should remain untouched?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/data-processing.md` — worksheet replacement / stale output.

### Card 6 — Async I/O and pool lifecycle

- **Recall:** What must happen when a timeout, cancellation, or connection failure changes task state?
- **Mental hook:** “State changes must wake the waiters.”
- **Why I care:** Missed notifications can become hangs; bad cleanup can stall shutdown.
- **Contrast:** A timeout bounds waiting; correct signaling and cleanup enable recovery.
- **Tiny example:** Cancel a task waiting for a pool connection and assert it exits within a deadline.
- **Test yourself:** Do failed connection attempts notify tasks already waiting for a connection?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/concurrency-and-parallelism.md` — async I/O / pool lifecycle.

### Card 7 — Parallel worker environment

- **Recall:** Why can a parallel task fail when an interactive sequential run works?
- **Mental hook:** “A worker is another execution context.”
- **Why I care:** Backend-specific options and implicit paths can break production jobs.
- **Contrast:** Async tasks and process-based workers have different execution models.
- **Tiny example:** Run a representative task on the production backend with explicit inputs and paths.
- **Test yourself:** Which assumptions does the task make about its working directory and worker configuration?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/concurrency-and-parallelism.md` — worker execution context.

### Card 8 — Native package/runtime compatibility

- **Recall:** Why can an installed Python package still crash a process?
- **Mental hook:** “The wheel must fit the runtime and its binary dependencies.”
- **Why I care:** Compiled incompatibilities can cause process-level failures, not ordinary exceptions.
- **Contrast:** Successful installation does not prove the deployed combination is runtime-safe.
- **Tiny example:** Test the locked Python/pandas/NumPy combination in the deployment image.
- **Test yourself:** Which exact interpreter and binary dependency versions are in the deployed artifact?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/deployment-and-infrastructure.md` — native package compatibility.

### Card 9 — Telemetry failure and graceful startup

- **Recall:** If optional telemetry configuration fails, should the application start?
- **Mental hook:** “Availability can continue, but degraded observability must be visible.”
- **Why I care:** Balances service availability against the ability to diagnose failures.
- **Contrast:** Startup success does not prove that instrumentation is working.
- **Tiny example:** Test startup with both a valid exporter and an invalid or unavailable one.
- **Test yourself:** Is telemetry optional for this service, and how will operators detect its failure?
- **First learned date:** 2026-10-05
- **Recommended review-card destination:** `review-cards/observability-and-reliability.md` — telemetry startup / degraded mode.

## Questions worth discussing

1. Which lockfiles, runtime images, or deployed environments should be checked first to establish actual exposure?
2. For the asyncmy SQL-injection advisory, what evidence from the maintainer or a verified advisory update would be sufficient to declare remediation?
3. What is the best way to test parameterized queries and dynamic identifiers in the team’s actual database access layer?
4. Which pandas output contracts—schema, dtype, missingness, index, and mutation behavior—are important enough to assert in pipeline tests?
5. For generated workbooks, what exactly counts as a full-sheet replacement, and which formulas, styles, or other cells must be preserved?
6. Which application dependencies are essential enough to block startup, and which should degrade gracefully with an explicit health or telemetry signal?