# Data Engineering Learning Resource

This directory is the authoritative, cumulative learning resource for the
Data Engineering Learning Pipeline.

Each topic file contains concepts that have passed human review and been
explicitly finalized. Existing finalized concept entries should be treated
as stable. New knowledge may be added without rewriting previously finalized
entries.

## Topic Index

- `databases.md`
  Database internals, relational databases, transactions, isolation,
  locking, indexes, query execution, connection management and pooling.

- `apis-and-networking.md`
  APIs, HTTP, REST/RPC, protocols, requests and responses, connections,
  networking fundamentals and client/server communication.

- `concurrency-and-parallelism.md`
  Threads, processes, asynchronous execution, event loops, futures,
  synchronization, race conditions and parallel computation.

- `distributed-systems.md`
  Distributed computation, consistency, coordination, retries,
  idempotency, messaging, replication and distributed failure modes.

- `software-architecture.md`
  Interfaces, abstractions, modularity, dependencies, application
  architecture and software-design concepts relevant to data systems.

- `data-processing.md`
  Data transformations, pipelines, batch and streaming processing,
  execution models, memory considerations and efficient data operations.

- `deployment-and-infrastructure.md`
  Deployment, environments, containers, CI/CD, cloud infrastructure,
  Azure and runtime configuration.

- `observability-and-reliability.md`
  Logging, metrics, tracing, monitoring, failure handling, resilience
  and reliability engineering.

- `validation-and-data-quality.md`
  Validation, schemas, data contracts, testing, correctness and
  data-quality concepts.

- `security.md`
  Authentication, authorization, secrets, permissions, least privilege
  and security concepts relevant to data and API systems.

- `statistics-and-evaluation.md`
  Statistical concepts, uncertainty, sampling, metrics, experimentation
  and evaluation methodology.

- `ai-and-llm-systems.md`
  AI and LLM system concepts including retrieval, embeddings, agents,
  model evaluation and AI-enabled data workflows.

## Routing Rules

Use the most specific suitable existing topic file.

Create a new topic file only when a subject:
1. does not fit an existing topic naturally;
2. represents a broad and reusable technical area; and
3. is likely to accumulate multiple concepts over time.

Do not create separate files for individual concepts merely because they
are new.

Review material mirrors this taxonomy under `review-cards/`.
For example, concepts stored in `learning/databases.md` have their review
material in `review-cards/databases.md`.
