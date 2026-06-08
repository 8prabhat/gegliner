Universal Agentic Platform Technical Specification

> Version: 4.0.0-draft
> Status: Improved production technical specification
> Date: 2026-06-08
> Scope: Universal, framework-agnostic, adapter-driven agentic AI platform

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Non-Scope](#2-scope-and-non-scope)
3. [Architectural Philosophy](#3-architectural-philosophy)
4. [Core Platform Capabilities](#4-core-platform-capabilities)
5. [Logical Architecture](#5-logical-architecture)
6. [Component Architecture](#6-component-architecture)
7. [Universal Adapter Model](#7-universal-adapter-model)
8. [Contracts and Interfaces](#8-contracts-and-interfaces)
9. [Domain Models and Schemas](#9-domain-models-and-schemas)
10. [Request Execution Flow](#10-request-execution-flow)
11. [Folder Structure Recommendation](#11-folder-structure-recommendation)
12. [API Design](#12-api-design)
13. [Runtime Modes](#13-runtime-modes)
14. [Multi-User and Multi-Tenant Design](#14-multi-user-and-multi-tenant-design)
15. [State, Memory, and Persistence Design](#15-state-memory-and-persistence-design)
16. [Tool and Skill Design](#16-tool-and-skill-design)
17. [Workflow and Orchestration Design](#17-workflow-and-orchestration-design)
18. [LLM Gateway Design](#18-llm-gateway-design)
19. [Security Design](#19-security-design)
20. [Observability Design](#20-observability-design)
21. [Evaluation and Testing Strategy](#21-evaluation-and-testing-strategy)
22. [Deployment Architecture](#22-deployment-architecture)
23. [Configuration Management](#23-configuration-management)
24. [Error Handling and Failure Recovery](#24-error-handling-and-failure-recovery)
25. [Development Standards](#25-development-standards)
26. [Suggested Implementation Phases](#26-suggested-implementation-phases)
27. [Trade-Offs and Design Decisions](#27-trade-offs-and-design-decisions)
28. [Anti-Patterns to Avoid](#28-anti-patterns-to-avoid)
29. [Final Recommended Architecture](#29-final-recommended-architecture)
30. [LLM-Friendly Implementation Playbook](#30-llm-friendly-implementation-playbook)
31. [Production Acceptance Checklist and Definitions](#31-production-acceptance-checklist-and-definitions)
A. [Appendix A: End-to-End Request Walkthrough](#appendix-a-end-to-end-request-walkthrough)

---

## 1. Executive Summary

The Universal Agentic Platform (UAP) is a production-grade platform for building, deploying, operating, observing, and governing agentic AI systems across organizations that use different agent frameworks, LLM providers, tool systems, memory stores, authorization models, workflow engines, and user interfaces.

The platform is not an agent framework. It is the stable runtime, API, contract, governance, and operations layer underneath agent frameworks. LangGraph, Google ADK, AutoGen, CrewAI, OpenAI Agents SDK, LlamaIndex workflows, custom in-house runtimes, and simple function-based agents must all be able to plug into the platform without forcing the platform core to depend on any one of them.

### What Makes the Platform Universal

| Property | Meaning |
|---|---|
| Universal | Supports many organizations with different frameworks, tools, schemas, auth systems, deployment environments, and operational practices. |
| Modular | Every concern has an explicit boundary: API, application services, runtime, adapters, contracts, infrastructure, security, observability, evaluation. |
| Framework-agnostic | Core platform code does not import LangGraph, ADK, AutoGen, CrewAI, OpenAI Agents SDK, LlamaIndex, or provider SDKs. |
| Adapter-driven | Organization-specific implementations plug in through versioned contracts and adapters. |
| Production-grade | Designed for durable execution, queues, workers, health checks, secrets, audit, rate limits, cost limits, backup/restore, and horizontal scaling. |
| Multi-user ready | Identity, sessions, per-user memory, permissions, audit, and concurrent execution are first-class. |
| Multi-tenant ready | Tenants have isolated data, policies, tools, skills, model routes, quotas, audit logs, and encryption options. |
| UI independent | CLI, TUI, web, desktop, and SDK clients call the same backend APIs and do not own orchestration logic. |
| Testable | Contracts, adapters, APIs, runtime flows, security rules, streaming, failure recovery, and evals are testable from the beginning. |
| Secure | Auth, authorization, sandboxing, prompt-injection controls, data permissions, secrets, and audit are structural requirements. |
| Observable | Every significant runtime event has logs, metrics, traces, audit records, run history, cost records, and correlation IDs. |

### Primary Outcome

An engineering team should be able to use this specification to implement an end-to-end agentic platform that supports:

- Interactive chat-style agents.
- Long-running workflows.
- Tool and skill execution.
- Human approvals.
- Streaming responses.
- Multi-framework adapters.
- Multi-tenant production deployment.
- Evaluation-driven release governance.
- Safe local development without local-only production assumptions.

---

## 2. Scope and Non-Scope

### 2.1 What This Specification Is

This is an improved technical specification produced by reviewing and strengthening an existing draft for a universal agentic AI platform.

It defines:

- Platform architecture.
- Component boundaries.
- Runtime lifecycle.
- Adapter model.
- Contracts and domain schemas.
- API design.
- State and persistence design.
- Security model.
- Observability and audit model.
- Evaluation and testing strategy.
- Deployment and operational requirements.
- Development standards and implementation phases.

### 2.2 What This Specification Is Not

| Not This | Reason |
|---|---|
| Code review | It does not inspect or evaluate a specific codebase. |
| Migration document | It does not describe migration from any existing system. |
| Refactoring plan | It does not prescribe changes to any existing repository. |
| Framework-specific guide | It is not LangGraph-only, ADK-only, AutoGen-only, CrewAI-only, OpenAI-only, or LlamaIndex-only. |
| UI-first architecture | UI clients are presentation layers, not runtime owners. |
| Local-only tool design | Local mode exists, but production architecture is not derived from local shortcuts. |
| A complete product roadmap | It defines technical phases and gates, not business timelines. |

### 2.3 In Scope

- Agent execution.
- Workflow and orchestration execution.
- Tool and skill execution.
- LLM gateway and prompt management.
- Planning and routing.
- Memory and knowledge retrieval.
- Human-in-the-loop.
- Streaming, async, and long-running execution.
- Multi-user and multi-tenant isolation.
- Auth, authorization, secrets, sandboxing, audit, and data governance.
- Adapter contracts and compatibility.
- API, SDK, CLI, TUI, and web client boundaries.
- Docker, Compose, Kubernetes readiness, workers, queues, storage, and observability.
- Evaluation, tests, regression, golden traces, and release gates.

### 2.4 Out of Scope

- A single universal reasoning algorithm.
- A single universal prompt style.
- A required programming language.
- A required database, vector database, queue, or cloud.
- A required UI framework.
- A requirement that every organization replace its existing agent packages.
- A requirement that all workflows use one engine.

### 2.5 Review Summary: Missing or Incorrect Areas Fixed

This specification corrects common weaknesses in agentic platform drafts:

| Area | Correction |
|---|---|
| Core/framework boundary | Core platform contracts do not import framework-specific types. Frameworks are integrated only through adapters. |
| UI/runtime boundary | UI clients call APIs and render state; they do not orchestrate agents, call tools, call models, or own memory logic. |
| Tool/skill boundary | Tools are atomic operations; skills are governed composite capabilities. Both are versioned and tested separately. |
| Orchestration/agent boundary | Orchestration coordinates steps and state; agents reason and propose actions within limits. |
| Local/production boundary | Local mode is a constrained profile, not a separate architecture that leaks unsafe assumptions into production. |
| Security | Authorization, reference monitor, tenant isolation, sandboxing, egress, secrets, audit, and prompt-injection defenses are first-class. |
| Runtime durability | Accepted runs use durable state, queues, leases, heartbeats, retries, events, and recovery. |
| Observability | Logs, metrics, traces, events, audit, cost, token usage, and correlation IDs are required. |
| Testing | Contract, adapter, API, streaming, security, multi-tenant, failure, load, eval, and deployment tests are architectural requirements. |
| Development quality | SOLID, DRY, KISS, YAGNI, dependency rules, code review gates, static analysis, and definitions of done are specified. |

---


## 3. Architectural Philosophy

### 3.1 Core Principles

| Principle | Requirement |
|---|---|
| Modularity | Each platform concern must live in a separately testable module with a narrow responsibility. |
| Independence | Components must communicate through contracts, not through shared mutable state or hidden imports. |
| Replaceability | Major implementations must be swappable through adapters without rewriting the platform. |
| Contracts over implementations | Cross-boundary behavior is defined by stable interfaces and schemas. |
| Ports and adapters | Core services depend on ports/contracts; adapters implement them for external systems. |
| Framework-agnostic core | Agent frameworks live behind adapters only. |
| API-first backend | All clients use backend APIs; runtime logic does not live in UI clients. |
| UI separation | CLI/TUI/Web/Desktop render and send commands; they do not orchestrate agents or run tools. |
| Production readiness | Local mode is a subset of production behavior, not a separate architecture. |
| Testability | Every contract, adapter, API, and runtime path has tests from the beginning. |
| Security by design | Auth, authorization, isolation, sandboxing, secrets, validation, and audit are structural. |
| Observability by design | Events, traces, metrics, logs, audit, cost, and run history are emitted by design. |
| Multi-tenant by design | Tenant context is present in all protected data, policy, usage, and runtime operations. |
| Failure-aware design | Timeouts, retries, cancellation, partial failure, idempotency, and compensation are explicit. |
| No god objects | No single class owns routing, planning, execution, policy, memory, tools, and formatting. |

### 3.2 Dependency Direction

```text
contracts/models/errors
  <- domain services
  <- application services
  <- API and workers

contracts/models/errors
  <- adapters

runtime services -> contracts only
adapters -> contracts + external SDK only
UI clients -> API/SDK only
```

Forbidden dependencies:

- Core -> LangGraph, ADK, AutoGen, CrewAI, OpenAI Agents SDK, LlamaIndex, provider SDKs.
- Adapter -> adapter.
- Adapter -> API.
- UI -> tool executor, model gateway, memory store, workflow engine.
- Tool -> raw database or secret store unless mediated by platform contracts.
- Agent framework -> protected capability without reference monitor.

### 3.3 Trust and Governance Principles

- Model output is untrusted.
- Retrieved content is untrusted unless verified.
- Memory is contextual input, not a system of record.
- Human approval is not sandboxing.
- LLM calls are not naturally idempotent.
- External side effects cannot be assumed exactly-once.
- Default deny is required for protected actions.
- Every side effect needs attribution, policy decision, and effect record.

---

## 4. Core Platform Capabilities

| Capability | Description | Core responsibility |
|---|---|---|
| Agent execution | Register, select, invoke, stream, monitor, version, evaluate, and replace agents. | Runtime contracts and governance. |
| Workflow execution | Execute short-running and long-running durable workflows. | State, transitions, checkpoints, retries. |
| Skill execution | Run reusable higher-level capabilities composed from tools, models, and logic. | Skill registry, executor, policy. |
| Tool execution | Run atomic operations with schemas, authorization, sandboxing, retries, and audit. | Tool registry and executor. |
| LLM invocation | Route requests to local or hosted models with fallback, streaming, structured output, cost and token tracking. | LLM gateway. |
| Prompt management | Version, test, render, and release prompt templates. | Prompt store and release governance. |
| Memory management | Store and retrieve scoped contextual memory with governance. | Memory service and adapters. |
| Knowledge retrieval | Ingest, parse, chunk, index, retrieve, rerank, cite, and delete knowledge. | RAG service and retrieval contracts. |
| Planning/decomposition | Break requests into plans and select execution strategies. | Planner contracts and runtimes. |
| Routing | Route requests to agent, workflow, skill, or tool based on policy and intent. | Router runtime. |
| Human-in-the-loop | Approval, review, correction, escalation, pause/resume, timeout. | Approval service. |
| State/session management | Manage sessions, conversations, workflow state, checkpoints, idempotency, cleanup. | State and session services. |
| Streaming | Stream model chunks, run events, tool events, workflow progress, approval events. | SSE/WebSocket/event stream. |
| Async execution | Submit runs, schedule work, process via workers, recover after crash. | Scheduler and workers. |
| Long-running execution | Durable waits, approvals, checkpoints, retries, scheduled/event-driven work. | Workflow runtime. |
| Observability | Logs, metrics, traces, run history, debug replay, cost, token tracking. | Observability pipeline. |
| Evaluation | Prompt, agent, tool, skill, workflow, security, regression, golden trace, load evals. | Evaluation service. |
| Security | Auth, authz, tenant isolation, secrets, sandboxing, validation, PII, egress. | Security services. |
| Multi-tenancy | Tenant-scoped data, policies, quotas, tools, skills, memory, audit, config. | Tenant services. |
| Deployment | Docker, Compose, Kubernetes readiness, workers, queues, storage, health checks. | Infrastructure. |
| Admin/configuration | Manage adapters, tenants, policies, releases, quotas, feature flags. | Admin services. |
| Auditability | Immutable audit trail for protected actions, admin changes, approvals, side effects. | Audit sink/service. |
| Cost governance | Estimate, reserve, track, limit, and report cost by tenant/user/run/model/tool. | Usage ledger. |

---

## 5. Logical Architecture

### 5.1 Layers

| Layer | Responsibility | Belongs here | Must not belong here | Allowed dependencies | Forbidden dependencies |
|---|---|---|---|---|---|
| Client layer | User interaction and presentation. | CLI, TUI, Web, Desktop, SDK clients. | Orchestration, tool execution, authz decisions. | API/SDK schemas. | Runtime internals, adapters, DB. |
| API layer | Versioned external interface. | REST, WebSocket, SSE, middleware, request validation. | Agent reasoning, tool logic, provider SDK calls. | Application services, API schemas. | Framework adapters, DB direct for business logic. |
| Application service layer | Use-case coordination. | Chat service, run service, session service, admin service. | Low-level provider calls, UI rendering. | Runtime contracts, domain models. | Provider SDKs, UI code. |
| Runtime orchestration layer | Execute runs and steps. | Agent runtime, workflow runtime, tool runtime, skill runtime, LLM gateway, planner, router. | API protocol handling, UI state. | Core contracts, state services, reference monitor. | UI clients, provider-specific code except through adapters. |
| Core domain layer | Stable platform language. | Contracts, domain models, errors, events, schemas. | Infrastructure setup, API handlers, framework code. | Standard library or language-native types only where practical. | Frameworks, provider SDKs, DB drivers. |
| Adapter layer | External integration implementations. | LangGraph adapter, OpenAI adapter, OIDC adapter, Qdrant adapter. | Platform policy, core business rules. | Core contracts, external SDK. | Other adapters, API, UI. |
| Infrastructure layer | Concrete storage and infrastructure. | DB repositories, queue, cache, object store, secret provider. | Agent reasoning, UI, business orchestration. | Contracts, config. | API route logic, framework logic. |
| Security layer | Cross-cutting security. | AuthN/AuthZ, policy engine, sandbox, redaction, egress, secrets. | UI rendering, provider-specific workflows. | Domain models, policy contracts. | Bypass paths around runtime. |
| Observability layer | Telemetry and audit. | Logs, metrics, traces, events, audit, cost ledger. | Business decisions. | Event contracts. | Raw secrets, unredacted sensitive payloads by default. |
| Evaluation/testing layer | Quality and conformance. | Eval suites, contract tests, golden traces, load tests. | Production runtime control flow. | Public contracts and test fixtures. | Hidden internal state dependence. |

### 5.2 Logical Data Flow

```text
Client -> API -> Application Service -> Runtime -> Reference Monitor
      -> Agent/Workflow/Tool/Skill/LLM/Memory/Retrieval Gateways
      -> Adapters -> External systems
      -> Events/Audit/State/Artifacts -> Client stream and history
```

### 5.3 Non-Bypassable Boundary

All protected actions must pass through the reference monitor:

- Tool and skill execution.
- LLM provider calls involving protected data.
- Memory reads/writes.
- Knowledge retrieval.
- Artifact reads/writes.
- Secret binding.
- Workflow side effects.
- Agent delegation.
- Admin operations.

---

## 6. Component Architecture

### 6.1 Agent Runtime

The Agent Runtime manages agent lifecycle, not framework-specific reasoning. It represents agents as versioned descriptors and invokes them through `AgentAdapter` implementations.

Responsibilities:

- Register and activate `AgentDescriptor` revisions.
- Select agents through explicit target, router, or workflow step.
- Instantiate framework-specific agents through adapters.
- Filter available tools, skills, memory, and models by policy.
- Invoke agents with `ExecutionContext`.
- Stream agent output.
- Track turns, tokens, cost, tool calls, delegated agents, and errors.
- Enforce limits: max turns, max steps, max depth, max fanout, timeout, cost.
- Evaluate agents before activation and during regression.
- Replace agents by activating a new immutable revision.

Agent representation:

```yaml
AgentDescriptor:
  agent_id: string
  revision: string
  name: string
  description: string
  adapter_type: string
  model_route_id: string
  prompt_revisions: [string]
  allowed_tools: [string]
  allowed_skills: [string]
  allowed_child_agents: [string]
  memory_policy: object
  knowledge_policy: object
  limits: object
  eval_requirements: object
  tenant_id: string | null
```

Agent replacement:

1. Create new agent revision.
2. Run required contract and eval suites.
3. Create release manifest.
4. Activate revision for tenant/cohort.
5. Route new runs to new revision.
6. Existing runs continue on resolved revisions unless policy forces suspension.

### 6.2 Orchestration Runtime

The Orchestration Runtime coordinates execution strategies. It does not replace agent frameworks. It normalizes lifecycle, state, policy, streaming, and observability for plans, chains, graphs, planner-executor loops, and external engines.

Orchestration differs from agent execution:

| Dimension | Agent execution | Orchestration |
|---|---|---|
| Purpose | Produce next model/action/final response. | Coordinate steps, dependencies, state, retries, waits. |
| Unit | Agent turn or child agent call. | Plan, graph, workflow, or state machine. |
| State | Conversation and agent turn state. | Step graph, transitions, checkpoints, retries. |
| Failure handling | Re-prompt, fallback, fail, or request input. | Retry, compensate, branch, resume, dead-letter. |

Supported orchestration models:

- Direct single-agent request.
- Sequential chain.
- Graph/DAG.
- State machine.
- Planner-executor.
- Hierarchical supervisor-worker.
- Event-driven workflow.
- External workflow engine through adapter.

### 6.3 LLM Gateway

The LLM Gateway is the only platform path for model invocation unless an explicitly approved adapter exception exists.

Responsibilities:

- Register providers and model endpoints.
- Discover provider capabilities.
- Route based on tenant policy, model capability, data classification, cost, latency, health, and availability.
- Support hosted APIs, local models, gateway services, and organization-specific clients.
- Support streaming, structured outputs, embeddings, tool calling, multimodal requests.
- Track tokens, latency, retries, fallback, and cost.
- Enforce model rate limits, budgets, timeouts, provider data handling, and prompt/version policy.
- Validate output and handle structured-output repair loops.

### 6.4 Tool Runtime

A tool is an atomic callable operation with clear input/output schemas and bounded side effects.

Tool Runtime responsibilities:

- Register versioned `ToolDescriptor` records.
- Validate input and output schemas.
- Evaluate tool permissions.
- Apply sandbox and egress policy.
- Execute through `ToolAdapter`.
- Enforce timeout, retry, idempotency, and cancellation.
- Record effects, artifacts, usage, events, and audit.
- Disable tools globally, per tenant, per release, or by kill switch.

Tool lifecycle:

`draft -> validated -> active -> deprecated -> disabled -> retired`

Dangerous tools require explicit policy, approval, sandboxing, effect records, and in many cases compensation or manual reconciliation.

### 6.5 Skill Runtime

A skill is a reusable higher-level capability that composes tools, model calls, retrieval, logic, and possibly sub-agents into a user-meaningful operation.

Skills differ from tools:

| Dimension | Tool | Skill |
|---|---|---|
| Granularity | Atomic operation. | Composite capability. |
| State | Usually stateless. | May have internal step state. |
| Duration | Usually short. | Can be longer but should be bounded. |
| Composition | Does not call other protected capabilities directly. | May call tools, models, memory, retrieval through platform gateways. |
| Testing | Schema/unit/integration. | Scenario, dependency, regression, safety evals. |

Skill Runtime responsibilities:

- Register and version `SkillDescriptor` records.
- Validate dependencies on tools, models, prompts, memory, retrieval, and policies.
- Execute skill plans through governed runtime services.
- Enforce inherited and skill-specific permissions.
- Evaluate skill quality and safety.
- Observe each internal step.

### 6.6 Workflow Runtime

The Workflow Runtime manages durable processes that may run for seconds, hours, days, or longer.

It supports:

- Short-running request workflows.
- Long-running workflows.
- Checkpointed workflows.
- Retryable workflows.
- Scheduled workflows.
- Event-driven workflows.
- Human-approved workflows.
- External workflow engines through adapters.

Workflow representation:

```yaml
WorkflowDescriptor:
  workflow_id: string
  revision: string
  steps:
    - step_id: string
      type: agent | tool | skill | approval | condition | parallel | join | wait | compensation
      target_id: string | null
      input_mapping: object
      output_mapping: object
      retry_policy: object
      timeout_seconds: integer
      transitions: object
  checkpoint_policy: every_step | on_wait | on_failure | manual
  compensation_policy: object
```

Workflow state is separate from conversation state. A conversation may start or observe a workflow, but workflow progress is stored as workflow execution state.

### 6.7 Memory System

Memory is governed contextual state used to improve future behavior. It is not the authoritative source of business truth.

Memory scopes:

- Run memory.
- Session memory.
- Conversation memory.
- User memory.
- Team memory.
- Tenant memory.
- Task memory.
- Workflow memory.
- Retrieval-derived memory.

Memory scope values:

```yaml
MemoryScope:
  enum: [run, session, conversation, user, team, tenant, task, workflow, retrieval]
```

Memory governance:

- Memory writes require policy.
- Memory records include provenance, confidence, classification, retention, owner, and scope.
- Model-generated memory should be proposed or verified before becoming active.
- Users must be able to view, correct, export, revoke, or delete applicable memory.
- Cross-tenant memory access is denied.

### 6.8 Knowledge/RAG System

The Knowledge/RAG System manages source material and retrieval.

Responsibilities:

- Source registration and access policy.
- Document ingestion.
- Malware/content validation.
- Parsing.
- Chunking.
- Metadata extraction.
- Embedding.
- Indexing.
- Hybrid retrieval.
- Reranking.
- Metadata filtering.
- Source citation.
- Freshness checks.
- Deletion propagation.
- Tenant and data-permission enforcement.

Retrieval must apply access filters before returning or exposing content, scores, or metadata.

### 6.9 Planning and Routing

Routing determines where a request goes. Planning determines how a task should be decomposed.

Routing responsibilities:

- Explicit target handling.
- Intent classification.
- Agent/workflow/skill/capability matching.
- Tenant policy filtering.
- Fallback behavior.

Planning responsibilities:

- Task decomposition.
- Required capability identification.
- Cost and risk estimation.
- Approval point identification.
- Execution strategy selection.
- Stop condition definition.

Planning output is a proposal. Each executable step still requires authorization at execution time.

### 6.10 Human-in-the-Loop

Human-in-the-loop includes:

- Approval.
- Review.
- Manual correction.
- Clarifying input.
- Escalation.
- Override.
- Emergency stop.

Approval requirements:

- Bound to exact action hash.
- Shows target, scope, diff, risk, cost, rollback/compensation, and expiry.
- Requires authorized approver.
- Supports timeout, rejection, request changes, reassignment, escalation.
- Persists audit event.
- Invalidated if material action changes.

### 6.11 State and Session Management

State types:

- User login sessions.
- Conversation state.
- Run state.
- Step and attempt state.
- Workflow execution state.
- Checkpoints.
- Idempotency records.
- Approval state.
- Artifact state.
- Effect state.

Requirements:

- State isolation by tenant and user.
- Durable state for accepted runs.
- Idempotency keys for mutating requests.
- Checkpoints versioned and migratable.
- Cleanup and retention policies.
- Resume validates policy and revision compatibility.

### 6.12 Auth and Security

Auth and security are cross-cutting but must not be scattered through business logic.

Responsibilities:

- Authentication via OIDC/JWT/API keys/workload identity/local dev auth.
- RBAC for broad permissions.
- ABAC for resource/context/data/risk decisions.
- Tenant isolation.
- Session isolation.
- Secret brokering.
- Tool and skill permissions.
- Data access controls.
- Sandbox and egress.
- Secure configuration.
- Input/output validation.
- PII handling.
- Audit.

### 6.13 Observability

Observability must cover:

- Structured logs.
- Metrics.
- Distributed traces.
- Agent traces.
- Tool traces.
- Skill traces.
- LLM traces.
- Workflow traces.
- Runtime events.
- Audit logs.
- Token usage.
- Cost usage.
- Latency.
- Errors and retry counts.
- Human approval events.
- Run history.
- Debug replay.
- Correlation IDs.

Every event should carry tenant, user, session, run, step, attempt, trace, and resource revision identifiers where applicable.

### 6.14 Evaluation and Quality

Evaluation must cover:

- Prompt evals.
- Agent evals.
- Tool evals.
- Skill evals.
- Workflow evals.
- Regression tests.
- Golden trace tests.
- Simulation tests.
- Safety tests.
- Prompt injection tests.
- Security tests.
- Load tests.
- Human evaluation where needed.

No production release should activate agent, prompt, model route, tool, skill, policy, or workflow revisions without required eval evidence.

### 6.15 API Layer

The API layer exposes platform capabilities through versioned protocols.

Supported protocols:

- REST for CRUD, admin, run submission, queries.
- SSE for one-way streaming.
- WebSocket for bidirectional interactive sessions.
- gRPC where low-latency internal or typed service boundaries are useful.

API responsibilities:

- Versioning.
- Auth middleware.
- Request validation.
- Idempotency.
- Pagination.
- Streaming cursors.
- Error schema.
- SDK compatibility.

### 6.16 UI Clients

Supported clients:

- CLI.
- TUI.
- Web.
- Desktop.
- External SDKs.

Rules:

- UI clients do not call tools directly.
- UI clients do not invoke model providers directly.
- UI clients do not own orchestration logic.
- UI clients render state from APIs and streams.
- Client SDKs wrap API calls and streaming, not runtime internals.

### 6.17 Deployment and Operations

The platform must support:

- Docker.
- Docker Compose.
- Kubernetes readiness.
- API containers.
- Worker containers.
- Web container.
- Database.
- Vector database.
- Cache.
- Queue.
- Object storage.
- Secrets management.
- Observability stack.
- Health/readiness checks.
- Graceful shutdown.
- Horizontal scaling.
- Backup/restore.
- Database migrations.
- Rolling deployments.

---

## 7. Universal Adapter Model

### 7.1 What an Adapter Is

An adapter is a concrete implementation of a platform contract for a specific external system, framework, provider, protocol, or organization-specific package.

Adapters translate between platform schemas and external semantics while preserving platform governance.

### 7.2 What Adapters Are Allowed To Do

- Import external SDKs and framework libraries.
- Translate platform requests into external requests.
- Translate external responses into platform responses.
- Expose capability metadata.
- Validate adapter-specific configuration.
- Manage connection pools or clients.
- Perform external health checks.
- Map external errors to platform errors.

### 7.3 What Adapters Must Not Do

- Bypass authorization, audit, or reference monitor.
- Contain core platform business rules.
- Import other adapters.
- Depend on API/UI modules.
- Store authoritative platform state privately.
- Leak provider-specific schemas into core schemas.
- Expand permissions beyond execution context.
- Hide material semantic differences from capability discovery.

### 7.4 Adapter Registration

Adapters register through manifests and factories.

```yaml
AdapterManifest:
  adapter_id: string
  adapter_type: agent_framework | llm_provider | auth | authorization | tool_system | skill_system | workflow_engine | memory_store | retriever | storage | event_bus | observability | ui | deployment
  name: string
  version: string
  contracts: [string]
  capabilities: object
  config_schema: object
  dependencies: [AdapterDependency]
  min_platform_version: string
  max_platform_version: string | null
  isolation_level: in_process | process | container | external_service
  health_check: object
  publisher: object
```

### 7.4.1 Adapter Loading Sequence

Adapters are discovered and loaded at platform startup:

1. Platform reads the list of enabled adapters from deployment config.
2. For each enabled adapter, platform locates the adapter package through a discovery mechanism (e.g., classpath scan, plugin entry points, or explicit config path).
3. Loads and validates `AdapterManifest` from the package.
4. Validates adapter config against the adapter-declared `config_schema`.
5. Checks adapter dependencies against available adapters.
6. Verifies platform version compatibility (`min_platform_version`, `max_platform_version`).
7. Calls adapter factory to create an instance.
8. Runs adapter health check.
9. Registers adapter in the platform adapter registry with its capabilities.
10. Logs adapter activation event.

Startup behavior:

| Condition | Behavior |
|---|---|
| Required adapter fails to load | Startup fails with `AdapterConfigError`. |
| Optional adapter fails to load | Marked unhealthy, logged, startup continues. |
| Adapter has unresolved dependency | Refuses activation with `AdapterDependencyError`. |
| Platform version incompatible | Refuses activation with `AdapterCompatibilityError`. |
| Config validation fails | Refuses activation with `ConfigValidationError`. |

### 7.5 Adapter Configuration

Adapter config is validated against adapter-declared schema. Config values that are secrets must be references, not plaintext.

```yaml
adapter_id: openai_primary
adapter_type: llm_provider
enabled: true
config:
  api_key_secret_ref: secret://tenant/openai
  default_model: gpt-4.1
  timeout_seconds: 30
  max_retries: 3
  rate_limit_rpm: 500
```

### 7.6 Adapter Testing

Every adapter must have:

- Manifest validation test.
- Contract conformance test.
- Config validation test.
- Unit tests with mocked external dependency.
- Integration tests where credentials/environment exist.
- Timeout and cancellation tests.
- Error mapping tests.
- Capability discovery tests.
- Version compatibility tests.
- Security tests for side effects, credentials, and data handling.

### 7.7 Capability Discovery

Adapters must declare:

- Supported operations.
- Required inputs.
- Output schemas.
- Streaming support.
- Cancellation support.
- Idempotency support.
- Retry semantics.
- Data handling limitations.
- Provider-specific limitations.
- Health status.

### 7.7.1 Standard Capability Schema

Adapter capabilities should use the following standard keys where applicable:

```yaml
capabilities:
  streaming: boolean           # Supports streaming responses.
  structured_output: boolean   # Supports structured/JSON output mode.
  tool_calling: boolean        # Supports tool/function calling.
  embeddings: boolean          # Supports embedding generation.
  multimodal: boolean          # Supports non-text input modalities.
  cancellation: boolean        # Supports in-flight cancellation.
  idempotency: boolean         # Supports idempotent operations.
  max_context_tokens: integer  # Maximum context window in tokens, or null.
  supported_modalities:        # List of supported input modalities.
    - text
    - image
    - audio
    - video
```

Adapters may add custom capability keys namespaced by adapter type (e.g., `langraph.checkpointing: true`). Custom keys must not conflict with standard keys.

### 7.8 Versioning and Compatibility

Adapter compatibility is checked at startup and activation time:

- Adapter version.
- Contract version.
- Platform version.
- External SDK version.
- Schema version.
- Capability version.
- Deprecation status.
- Replacement adapter.

Breaking adapter changes require new major adapter versions or compatibility mappers.

### 7.9 Failure Handling

| Failure | Required behavior |
|---|---|
| Startup config invalid | Fail startup in production if required adapter. |
| Optional adapter unavailable | Mark unhealthy and route away. |
| Runtime timeout | Cancel operation and emit typed timeout error. |
| External provider error | Map to platform error and apply retry/fallback policy. |
| Adapter crash | Isolate failure, mark unhealthy, alert. |
| Version incompatible | Refuse activation. |
| Security violation | Disable adapter or capability and emit audit event. |

### 7.10 Adapter Categories

| Category | Examples |
|---|---|
| Agent frameworks | LangGraph, Google ADK, AutoGen, CrewAI, OpenAI Agents SDK, LlamaIndex, custom function agent. |
| LLM providers | OpenAI, Anthropic, Gemini, Cohere, Mistral, Ollama, vLLM, local gateway. |
| Auth providers | OIDC, Auth0, Keycloak, Cognito, LDAP, local dev auth. |
| Authorization providers | OPA, Cedar, custom RBAC/ABAC, cloud IAM. |
| Tool systems | MCP, HTTP tools, Python functions, shell tools, browser/computer use, SaaS APIs. |
| Skill systems | Custom packages, DSPy-style modules, workflow-backed skills. |
| Workflow engines | Temporal, Prefect, Airflow, LangGraph, LlamaIndex workflows, custom. |
| Memory stores | PostgreSQL, Redis, DynamoDB, SQLite. |
| Vector databases | Qdrant, pgvector, Pinecone, Weaviate, Chroma, FAISS. |
| SQL databases | PostgreSQL, MySQL, SQLite. |
| Document databases | MongoDB, DynamoDB, Firestore. |
| Event buses | Kafka, NATS, RabbitMQ, Redis Streams, database outbox. |
| File/object stores | Local filesystem, S3, GCS, Azure Blob, MinIO. |
| UI clients | CLI, TUI, Web, Desktop, SDK. |
| Observability backends | OpenTelemetry, Prometheus, Grafana, Datadog, CloudWatch, stdout. |
| Deployment targets | Docker, Compose, Kubernetes, ECS, Cloud Run, on-prem. |

---

## 8. Contracts and Interfaces

Contracts are stable platform boundaries. The syntax below is pseudocode; implementations may use TypeScript interfaces, Python protocols, Java interfaces, Rust traits, OpenAPI, protobuf, or equivalent typed contracts.

### 8.1 Contract Definition Template

Every contract must define:

- Purpose.
- Key methods.
- Input schema.
- Output schema.
- Error model.
- Lifecycle.
- Test requirements.

### 8.1.1 Contract Input, Output, and Error Matrix

| Contract | Input schema | Output schema | Error model |
|---|---|---|---|
| `Agent` | `AgentRequest(messages, tools, memory_refs, limits)` | `AgentResponse(messages, tool_calls, final_output, usage)` | `AgentError(code, retryable, safe_message)` |
| `AgentRuntime` | `ExecutionRequest`, `AgentDescriptor`, `ExecutionContext` | `ExecutionResponse` or stream of `RuntimeEvent` | `RuntimeError`, `PolicyError`, `LimitExceededError` |
| `AgentAdapter` | `AgentDescriptor`, adapter config | `Agent` instance or factory result | `AdapterConfigError`, `AdapterCompatibilityError` |
| `Orchestrator` | `ExecutionPlan`, `ExecutionContext` | `OrchestrationResult(step_results, final_state)` | `OrchestrationError(step_id, retryable)` |
| `OrchestratorRuntime` | request or plan, runtime context | run/step state updates | `PlanningError`, `StateTransitionError` |
| `OrchestratorAdapter` | platform plan or workflow step | external execution result translated to platform result | `AdapterTranslationError`, `ExternalEngineError` |
| `Workflow` | workflow definition body | validated immutable workflow revision | `WorkflowValidationError` |
| `WorkflowRuntime` | `WorkflowStartRequest`, signal, resume/cancel command | `WorkflowExecutionState`, events | `WorkflowError`, `CheckpointError` |
| `WorkflowAdapter` | workflow command and engine config | external workflow handle/state | `ExternalWorkflowError` |
| `Tool` | schema-valid JSON object | schema-valid JSON result plus artifacts/effects | `ToolExecutionError` |
| `ToolRegistry` | `ToolDescriptor` revisions and filters | descriptors, activation records | `RegistryError`, `VersionConflictError` |
| `ToolExecutor` | `tool_ref`, JSON input, `ExecutionContext` | `ToolResult(output, artifacts, effects, usage)` | `ToolAuthzError`, `SandboxError`, `ToolTimeoutError` |
| `ToolAdapter` | prepared invocation with scoped credentials | adapter-native result translated to `ToolResult` | `AdapterRuntimeError`, `ExternalToolError` |
| `Skill` | schema-valid JSON input | `SkillResult(output, step_trace, artifacts, usage)` | `SkillExecutionError` |
| `SkillRegistry` | `SkillDescriptor` revisions and filters | descriptors, dependency validation result | `RegistryError`, `DependencyError` |
| `SkillExecutor` | `skill_ref`, JSON input, `ExecutionContext` | `SkillResult` | `SkillAuthzError`, `SkillDependencyError` |
| `SkillAdapter` | skill package config and invocation | skill package result translated to platform result | `AdapterRuntimeError` |
| `LLMClient` | `LLMRequest` | `LLMResponse` or stream of `LLMChunk` | `LLMProviderError`, `RateLimitError`, `ContextLimitError` |
| `LLMGateway` | `LLMRequest`, routing hints, policy context | normalized `LLMResponse`, usage/cost records | `GatewayRoutingError`, `ModelPolicyError` |
| `LLMAdapter` | provider config | `LLMClient`, model capability descriptors | `AdapterConfigError`, `ProviderHealthError` |
| `PromptProvider` | prompt refs and variable values | rendered trusted/untrusted message blocks | `PromptRenderError`, `PromptPolicyError` |
| `PromptTemplateStore` | prompt draft/revision | immutable prompt revision | `PromptValidationError`, `VersionConflictError` |
| `MemoryStore` | scope, key/query, value, metadata | `MemoryRecord` or result list | `MemoryAccessError`, `MemoryConflictError` |
| `MemoryAdapter` | backend query/write/delete operation | backend result normalized to memory model | `StorageAdapterError` |
| `Retriever` | `RetrievalRequest(query, filters, limit)` | `RetrievalResult(chunks, scores, citations)` | `RetrievalAccessError`, `IndexUnavailableError` |
| `RetrieverAdapter` | index/search/delete command | indexed IDs or search hits | `VectorStoreError`, `IndexingError` |
| `Planner` | request, available resources, context | `ExecutionPlan` | `PlanningError`, `UnsupportedTaskError` |
| `Router` | request, tenant/user/session context | `RoutingDecision(target, confidence, reason)` | `RoutingError`, `NoRouteError` |
| `SessionStore` | session create/update/list command | `SessionContext` or session list | `SessionError`, `SessionAccessError` |
| `StateStore` | state key, payload, metadata, version | saved/loaded state and version | `StateConflictError`, `StateStoreError` |
| `CheckpointStore` | checkpoint payload and schema version | checkpoint ref or loaded checkpoint | `CheckpointMigrationError` |
| `AuthProvider` | credentials/token | `UserContext` or service principal | `AuthenticationError` |
| `AuthorizationPolicy` | identity, action, resource, context | `PolicyDecision` | `AuthorizationError`, `PolicyEvaluationError` |
| `SecretProvider` | secret ref and binding context | scoped credential or secret handle | `SecretAccessError`, `SecretRotationError` |
| `EventBus` | `RuntimeEvent` or subscription request | append ack or event stream | `EventPublishError`, `CursorGapError` |
| `ObservabilitySink` | metric/log/trace event | emit ack or best-effort status | `TelemetryError` |
| `AuditSink` | `AuditEvent` | immutable audit id | `AuditWriteError`, `AuditAccessError` |
| `EvaluationRunner` | eval suite, target revisions, config | `EvaluationReport` | `EvaluationError`, `ThresholdFailure` |
| `ConfigProvider` | environment and tenant scope | typed config object | `ConfigValidationError` |
| `TenantResolver` | request and identity context | `TenantContext` | `TenantResolutionError`, `TenantSuspendedError` |
| `ReferenceMonitor` | `AuthorizationRequest(identity, action, resource, context, action_hash)` | `PolicyDecision(allowed, obligations, require_approval, deny_reason)` | `AuthorizationError`, `PolicyEvaluationError`, `ApprovalRequiredError` |
| `RunService` | `ExecutionRequest`, filters, cancellation reason | `RunRecord`, `Page<RunRecord>` | `ValidationError`, `RunNotFoundError`, `IdempotencyConflictError` |
| `Scheduler` | `Step`, priority, worker_id, lease_id | `StepRef`, `LeasedStep`, ack | `LeaseExpiredError`, `StepNotFoundError`, `DeadLetterError` |
| `WorkerRuntime` | `LeasedStep`, shutdown command | `StepResult`, shutdown ack | `LeaseExpiredError`, `StepExecutionError`, `ShutdownTimeoutError` |
| `ArtifactService` | `Artifact` metadata, content bytes, run/step context | `Artifact` with storage ref | `ArtifactAccessError`, `ArtifactNotFoundError`, `StorageLimitError` |

### 8.2 Core Runtime Contracts

| Contract | Purpose | Key methods | Lifecycle | Tests |
|---|---|---|---|---|
| `Agent` | Executable agent instance. | `invoke(request, context)`, `stream(request, context)`, `capabilities()` | Per request or pooled by adapter. | Invoke, stream, limits, errors. |
| `AgentRuntime` | Platform service that selects and runs agents. | `execute_agent(run, agent_ref)`, `delegate_agent(...)` | Singleton service. | Routing, policy, delegation, streaming. |
| `AgentAdapter` | Creates framework-specific agents. | `create_agent(descriptor)`, `validate_config()`, `health_check()` | Singleton per adapter config. | Config, factory, health, compatibility. |
| `Orchestrator` | Executes an execution plan. | `execute(plan, context)`, `pause()`, `resume()`, `cancel()` | Per plan/run. | Plan, pause/resume, failure. |
| `OrchestratorRuntime` | Selects orchestration strategy. | `execute_plan()`, `select_orchestrator()`, `recover()` | Singleton. | Strategy, recovery, checkpoints. |
| `OrchestratorAdapter` | Bridges external graph/planner engines. | `translate_plan()`, `execute_external()`, `translate_result()` | Singleton per engine. | Translation, tool wrapper, state. |
| `Workflow` | Versioned workflow definition. | `validate()`, `next_steps()`, `compensation_plan()` | Immutable revision. | Validation, transitions. |
| `WorkflowRuntime` | Durable workflow execution. | `start()`, `signal()`, `resume()`, `cancel()`, `get_state()` | Singleton service. | Long-run, retry, wait, checkpoint. |
| `WorkflowAdapter` | External workflow engine bridge. | `start_external()`, `signal_external()`, `query_external()` | Singleton per engine. | Engine compatibility, failure. |
| `WorkerRuntime` | Leases and executes steps from the scheduler. | `start()`, `shutdown(grace_seconds)`, `process_step(leased_step)` | Long-running process. | Lease expiry, crash recovery, graceful shutdown, heartbeat. |

### 8.3 Tool and Skill Contracts

| Contract | Purpose | Key methods | Lifecycle | Tests |
|---|---|---|---|---|
| `Tool` | Atomic callable operation. | `execute(input, context)`, `schema()`, `describe()` | Stateless or pooled. | Schema, authz, timeout, output. |
| `ToolRegistry` | Stores tool descriptors. | `register()`, `get()`, `list()`, `activate()`, `disable()` | Singleton. | CRUD, versioning, tenant filtering. |
| `ToolExecutor` | Governed tool execution. | `execute(tool_ref, input, context)` | Singleton. | Policy, sandbox, idempotency, effects. |
| `ToolAdapter` | Tool implementation bridge. | `prepare()`, `invoke()`, `verify_effect()`, `compensate()` | Per adapter. | Failure, cancellation, credentials. |
| `Skill` | Composite capability. | `execute(input, context)`, `requirements()`, `schema()` | Per invocation or pooled. | Scenario, dependency, failure. |
| `SkillRegistry` | Stores skill descriptors. | `register()`, `get()`, `list()`, `activate()`, `disable()` | Singleton. | CRUD, dependency validation. |
| `SkillExecutor` | Governed skill execution. | `execute(skill_ref, input, context)` | Singleton. | Tool/model calls through gateways. |
| `SkillAdapter` | External skill package bridge. | `load_skill()`, `invoke_skill()`, `health_check()` | Per package. | Compatibility, dependency, safety. |

### 8.4 LLM and Prompt Contracts

| Contract | Purpose | Key methods | Lifecycle | Tests |
|---|---|---|---|---|
| `LLMClient` | Provider-specific model client. | `complete()`, `stream()`, `embed()`, `count_tokens()` | Pooled singleton. | Completion, stream, rate, errors. |
| `LLMGateway` | Policy-aware model routing. | `complete(request)`, `stream(request)`, `usage()` | Singleton. | Routing, fallback, cost, policy. |
| `LLMAdapter` | Provider adapter. | `describe_models()`, `create_client()`, `health_check()` | Singleton per provider. | Capability discovery, error mapping. |
| `PromptProvider` | Resolves prompts for runtime. | `resolve(prompt_ref)`, `render()` | Singleton. | Rendering, trust blocks, variables. |
| `PromptTemplateStore` | Stores prompt revisions. | `create_revision()`, `get()`, `activate()`, `list_versions()` | Persistent service. | Versioning, compatibility, tests. |

### 8.5 Memory and Retrieval Contracts

| Contract | Purpose | Key methods | Lifecycle | Tests |
|---|---|---|---|---|
| `MemoryStore` | Scoped memory persistence. | `store()`, `retrieve()`, `search()`, `delete()`, `correct()` | Persistent adapter. | Scope isolation, retention. |
| `MemoryAdapter` | Backend-specific memory implementation. | `connect()`, `query()`, `write()`, `delete()` | Per backend. | Backend errors, performance. |
| `Retriever` | Retrieves knowledge chunks. | `retrieve()`, `hybrid_retrieve()`, `rerank()` | Singleton. | Accuracy, filters, authz. |
| `RetrieverAdapter` | Vector/search backend bridge. | `index()`, `search()`, `delete()`, `health_check()` | Per backend. | Tenant isolation, deletion. |

### 8.6 Planning, Routing, State, and Session Contracts

| Contract | Purpose | Key methods | Lifecycle | Tests |
|---|---|---|---|---|
| `Planner` | Creates execution plans. | `plan(request, context)` | Singleton or per request. | Decomposition, risk, cost. |
| `Router` | Selects target agent/workflow/skill/tool. | `route(request, context)` | Singleton. | Explicit and intent routes. |
| `SessionStore` | Manages user sessions. | `create()`, `get()`, `list_by_user()`, `delete()` | Persistent service. | User/tenant isolation. |
| `StateStore` | Persists run and conversation state. | `save()`, `load()`, `compare_and_set()` | Persistent service. | Concurrency, recovery. |
| `CheckpointStore` | Stores resumable checkpoints. | `save_checkpoint()`, `load_checkpoint()`, `migrate()` | Persistent service. | Versioning, migration. |
| `RunService` | Manages run lifecycle. | `submit(request, context)`, `get(run_id)`, `cancel(run_id, reason)`, `list(filters, page)` | Singleton service. | Submit, cancel, idempotency, status transitions, tenant isolation. |
| `Scheduler` | Enqueues and leases work steps for workers. | `enqueue(step, priority)`, `lease(worker_id, queues, lease_seconds)`, `heartbeat(lease_id)`, `complete(lease_id, result)`, `fail(lease_id, error)` | Singleton service. | Lease expiry, heartbeat, dead-letter, concurrent leasing. |

### 8.7 Security, Observability, Evaluation, and Config Contracts

| Contract | Purpose | Key methods | Lifecycle | Tests |
|---|---|---|---|---|
| `ReferenceMonitor` | Non-bypassable authorization checkpoint for all protected actions. | `authorize_action(identity, action, resource, context)`, `authorize_data_access(identity, data_ref, context)` | Singleton service. | Every protected action, bypass attempts, deny logging. |
| `AuthProvider` | Authenticates identities. | `authenticate()`, `validate_token()`, `refresh()` | Per provider. | Token, expiry, invalid auth. |
| `AuthorizationPolicy` | Authorizes actions. | `authorize(identity, action, resource, context)` | Singleton/adapter. | Allow, deny, obligations. |
| `SecretProvider` | Provides scoped secrets. | `get_secret_ref()`, `issue_credential()`, `revoke()` | Singleton. | Redaction, rotation, audit. |
| `EventBus` | Publishes/subscribes events. | `publish()`, `subscribe()`, `append()` | Singleton. | Ordering, duplicate, replay. |
| `ObservabilitySink` | Emits logs/metrics/traces. | `emit()`, `emit_batch()`, `flush()` | Singleton or adapter. | Redaction, failure isolation. |
| `AuditSink` | Writes audit records. | `append()`, `query()`, `export()` | Persistent service. | Immutability, access control. |
| `ArtifactService` | Stores and retrieves execution artifacts. | `store(artifact, content, context)`, `get(artifact_id, context)`, `list(run_id, context)`, `delete(artifact_id, context)` | Singleton service. | Store, retrieve, access control, retention, deletion. |
| `EvaluationRunner` | Runs eval suites. | `run_suite()`, `compare()`, `report()` | Worker service. | Regression, thresholds. |
| `ConfigProvider` | Resolves typed config. | `load()`, `validate()`, `watch()` | Singleton. | Precedence, invalid config. |
| `TenantResolver` | Resolves tenant context. | `resolve(request, identity)` | Singleton. | Spoofing, cross-tenant. |

### 8.8 Interface-Style Examples

The following pseudocode defines key method signatures. Implementations may use TypeScript interfaces, Python protocols, Java interfaces, Rust traits, or equivalent typed contracts.

```text
interface ToolExecutor {
  execute(
    toolRef: ResourceRef,
    input: JsonObject,
    context: ExecutionContext
  ) -> Result<ToolResult, PlatformError>
}

interface AuthorizationPolicy {
  authorize(
    identity: IdentityContext,
    action: string,
    resource: ResourceRef,
    context: PolicyContext
  ) -> PolicyDecision
}

interface ReferenceMonitor {
  authorize_action(
    identity: IdentityContext,
    action: string,
    resource: ResourceRef,
    context: ExecutionContext
  ) -> PolicyDecision

  authorize_data_access(
    identity: IdentityContext,
    data_ref: ResourceRef,
    context: ExecutionContext
  ) -> PolicyDecision
}

interface AgentRuntime {
  execute_agent(
    run: RunRecord,
    agent_ref: ResourceRef,
    context: ExecutionContext
  ) -> ExecutionResponse | Stream<RuntimeEvent>

  delegate_agent(
    parent_run: RunRecord,
    child_ref: ResourceRef,
    input: JsonObject,
    context: ExecutionContext
  ) -> ExecutionResponse
}

interface WorkflowRuntime {
  start(
    request: WorkflowStartRequest,
    context: ExecutionContext
  ) -> WorkflowExecutionState

  signal(
    run_id: string,
    signal: WorkflowSignal,
    context: ExecutionContext
  ) -> WorkflowExecutionState

  resume(run_id: string, context: ExecutionContext) -> WorkflowExecutionState
  cancel(run_id: string, reason: string) -> WorkflowExecutionState
  get_state(run_id: string) -> WorkflowExecutionState
}

interface LLMGateway {
  complete(
    request: LLMRequest,
    context: ExecutionContext
  ) -> LLMResponse

  stream(
    request: LLMRequest,
    context: ExecutionContext
  ) -> Stream<LLMChunk>
}

interface MemoryStore {
  store(scope: MemoryScope, record: MemoryRecord, context: ExecutionContext) -> MemoryRecord
  retrieve(scope: MemoryScope, key: string, context: ExecutionContext) -> MemoryRecord | null
  search(scope: MemoryScope, query: string, limit: int, context: ExecutionContext) -> [MemoryRecord]
  delete(scope: MemoryScope, key: string, context: ExecutionContext) -> void
  correct(scope: MemoryScope, key: string, corrected: MemoryRecord, context: ExecutionContext) -> MemoryRecord
}

interface Retriever {
  retrieve(request: RetrievalRequest, context: ExecutionContext) -> RetrievalResult
  hybrid_retrieve(request: RetrievalRequest, context: ExecutionContext) -> RetrievalResult
  rerank(results: RetrievalResult, query: string, limit: int) -> RetrievalResult
}

interface RunService {
  submit(request: ExecutionRequest, context: ExecutionContext) -> RunRecord
  get(run_id: string) -> RunRecord
  cancel(run_id: string, reason: string) -> RunRecord
  list(filters: RunFilters, page: PageRequest) -> Page<RunRecord>
}

interface Scheduler {
  enqueue(step: Step, priority: int) -> StepRef
  lease(worker_id: string, queues: [string], lease_seconds: int) -> LeasedStep | null
  heartbeat(lease_id: string) -> void
  complete(lease_id: string, result: StepResult) -> void
  fail(lease_id: string, error: PlatformError) -> void
}

interface EventBus {
  publish(event: RuntimeEvent) -> EventId
  subscribe(filter: EventFilter) -> Stream<RuntimeEvent>
  append(event: RuntimeEvent, outbox_tx: Transaction) -> EventId
  replay(cursor: EventCursor, limit: int) -> [RuntimeEvent]
}

interface SecretProvider {
  get_secret_ref(ref: string, context: ExecutionContext) -> SecretHandle
  issue_credential(principal: IdentityContext, scopes: [string], ttl_seconds: int) -> ScopedCredential
  revoke(handle: SecretHandle) -> void
}
```

---

## 9. Domain Models and Schemas

### 9.1 Model Catalog

| Model | Purpose | Required fields | Optional fields | Ownership | Persistence |
|---|---|---|---|---|---|
| `ExecutionRequest` | Incoming request to execute agent/workflow/skill/tool. | target, input, mode, limits. | session_id, idempotency_key, labels. | Caller tenant/user. | Persist as run input artifact for accepted runs. |
| `ExecutionResponse` | Initial or final execution response. | run_id, status, result/error. | stream_url, usage, artifacts. | Platform. | Persist final response or result ref. |
| `ExecutionContext` | Runtime context passed across services. | run_id, tenant, user, session, trace, limits. | parent_run_id, approvals, feature flags. | Platform. | Serialized safe subset only. |
| `TenantContext` | Tenant configuration and isolation metadata. | tenant_id, status, policies. | encryption, residency, quotas. | Tenant admin/platform. | Persistent. |
| `UserContext` | User identity and permissions. | user_id, tenant_id, roles. | groups, attributes, preferences. | Identity provider/platform. | Persistent or derived. |
| `SessionContext` | User session/conversation context. | session_id, user_id, tenant_id. | conversation_id, client metadata. | User. | Persistent with expiry. |
| `AgentDescriptor` | Versioned agent definition. | id, revision, adapter, model route, prompts, limits. | eval requirements, tags. | Tenant/platform. | Persistent immutable revision. |
| `ToolDescriptor` | Versioned tool definition. | id, revision, schemas, permissions, side effects. | sandbox, retry, health. | Tenant/platform. | Persistent immutable revision. |
| `SkillDescriptor` | Versioned skill definition. | id, revision, input/output schema, dependencies. | evals, prompts. | Tenant/platform. | Persistent immutable revision. |
| `WorkflowDescriptor` | Versioned workflow definition. | id, revision, steps, transitions. | schedules, compensation. | Tenant/platform. | Persistent immutable revision. |
| `LLMRequest` | Gateway model request. | messages, model route, limits. | tools, output schema, modality. | Runtime. | Metadata persisted; payload by policy. |
| `LLMResponse` | Gateway model result. | content/tool calls, usage, finish reason. | safety, raw ref. | Runtime. | Persist by policy. |
| `MemoryRecord` | Governed contextual memory. | scope, subject, content, provenance. | confidence, expiry. | User/tenant/platform. | Persistent. |
| `RetrievalRequest` | Knowledge search request. | query, tenant, filters, limit. | rerank, hybrid options. | Runtime. | Event/audit metadata. |
| `RetrievalResult` | Retrieved chunks. | chunks, scores, source refs. | citations, freshness. | Runtime. | Event metadata; chunks are references. |
| `ApprovalRequest` | Bound human approval request. | action hash, run_id, approvers, expiry. | summary, diff. | Runtime. | Persistent. |
| `ApprovalDecision` | Human decision. | approval_id, decision, decided_by. | reason, modifications. | Approver. | Persistent/audited. |
| `RuntimeEvent` | Run/step/platform event. | event_id, type, run_id, sequence, payload. | attempt_id, redaction. | Platform. | Persistent event log. |
| `AuditEvent` | Security/governance record. | actor, action, resource, outcome, time. | policy decision, payload ref. | Platform. | Tamper-evident persistent log. |
| `ErrorResponse` | Safe error to clients. | code, message, trace_id, retryable. | details, retry_after. | Platform. | Persist for failed runs. |
| `CostRecord` | Usage and cost accounting. | tenant, run, model/tool, tokens/cost. | estimated/billed flag. | Platform. | Persistent ledger. |
| `Message` | Single message in a conversation. | message_id, conversation_id, role (user/assistant/system/tool), content. | tool_calls, tool_call_id, attachments, metadata, created_at. | User/system. | Persistent. |
| `Conversation` | Threaded sequence of messages within a session. | conversation_id, session_id, tenant_id, user_id, created_at. | title, message_count, last_message_at, metadata. | User. | Persistent. |
| `ToolResult` | Result of tool execution. | output (JsonObject), status (success/failure). | artifacts, effects, usage, duration_ms, warnings. | Tool runtime. | Persisted per step/attempt. |
| `SkillResult` | Result of skill execution. | output (JsonObject), status (success/failure), step_trace. | artifacts, usage, duration_ms. | Skill runtime. | Persisted per step/attempt. |
| `EffectRecord` | Record of an external side effect for governance and compensation. | effect_id, run_id, step_id, effect_type, target, status (confirmed/uncertain/failed/compensated). | compensation_ref, verified_at, metadata. | Platform. | Persistent. |
| `WorkflowStartRequest` | Request to start a workflow run. | workflow_ref (ResourceRef), input (JsonObject), mode (sync/async/stream). | limits, labels, idempotency_key. | Caller. | Persist as run input. |
| `WorkflowSignalRequest` | Signal sent to a running workflow. | run_id, signal_type, payload. | timeout_seconds. | Caller/system. | Persist as event. |
| `WorkflowExecutionState` | Durable state of one workflow run. | run_id, workflow_ref, status, current_step_id, step_states (map). | checkpoints, started_at, updated_at, completed_at, error. | Platform. | Persistent. |
| `ReleaseManifest` | Bundle of resource revisions for coordinated activation. | manifest_id, created_at, created_by, resources (list of ResourceRef with revisions). | eval_evidence, rollback_manifest_id, notes. | Release author. | Persistent immutable. |
| `TokenUsage` | Token accounting for one LLM call. | prompt_tokens, completion_tokens, total_tokens. | model, provider, cached_tokens, cost_usd. | LLM gateway. | Embedded in LLMResponse and CostRecord. |
| `ModelRoute` | Named routing rule mapping logical model needs to physical providers. | route_id, model_id, provider_id. | fallback_routes, data_classification_filter, cost_limit, tenant_id, latency_target_ms, required_capabilities. | Tenant/platform. | Persistent. |
| `ExecutionPlan` | Structured plan produced by a planner for orchestrated execution. | plan_id, request_ref, steps (ordered list of planned steps with dependencies). | estimated_cost, estimated_duration, risk_level, approval_points, stop_conditions. | Planner. | Persist for accepted plans. |
| `LLMChunk` | One chunk in a streaming LLM response. | chunk_id, sequence, content_delta, finish_reason. | tool_call_delta, usage_delta. | LLM gateway. | Transient (streamed). |
| `StepResult` | Result of executing one step. | step_id, status (completed/failed), output_ref. | error, effects, artifacts, usage, duration_ms. | Runtime. | Persistent. |
| `Page` | Generic paginated response wrapper. | data (list), limit (int). | next_cursor, total_count. | API layer. | Transient. |
| `RunRecord` | Persistent record of one execution run. | run_id, status, target (ResourceRef), tenant_id, user_id, session_id, created_at, updated_at. | result, error, usage, artifacts, events_cursor, idempotency_key, labels. | Platform. | Persistent with retention. |
| `Step` | One unit of work within a run. | step_id, run_id, type (agent/tool/skill/approval/condition/parallel/join/wait/compensation), target_ref (ResourceRef), status, sequence. | input_ref, output_ref, attempt_count, retry_policy, timeout_seconds, parent_step_id, dependencies. | Platform. | Persistent. |
| `Attempt` | One execution attempt of a step. | attempt_id, step_id, run_id, attempt_number, status, started_at. | completed_at, error, effect_records, usage, duration_ms. | Platform. | Persistent. |
| `ResourceRef` | Universal typed reference to a platform resource. | kind (agent/tool/skill/workflow/prompt/model_route), id. | revision, tenant_id. | Caller. | Value object. |
| `Artifact` | File or data produced during execution. | artifact_id, run_id, step_id, name, content_type, storage_ref, size_bytes, checksum. | metadata, labels, retention_policy. | Run owner/tenant. | Persistent with retention. |
| `PolicyDecision` | Authorization decision from policy engine. | allowed (bool), decision_id, evaluated_at. | obligations, require_approval (bool), deny_reason, deny_code, constraints. | Policy engine. | Transient (audited). |
| `IdentityContext` | Unified identity for authorization checks. | principal_type (user/service/agent), principal_id, tenant_id. | roles, groups, attributes, delegated_by, session_id. | Auth provider. | Derived per request. |
| `PolicyContext` | Context for policy evaluation beyond identity. | run_id, step_id, action, resource_ref. | data_classification, risk_level, feature_flags, tenant_policies, resource_metadata. | Platform. | Transient. |

### 9.2 ExecutionRequest Example Schema

```json
{
  "type": "object",
  "required": ["target", "input", "mode", "limits"],
  "properties": {
    "idempotency_key": {"type": ["string", "null"]},
    "target": {
      "type": "object",
      "required": ["kind", "id"],
      "properties": {
        "kind": {"enum": ["agent", "workflow", "skill", "tool"]},
        "id": {"type": "string"},
        "revision": {"type": ["string", "null"]}
      }
    },
    "mode": {"enum": ["sync", "async", "stream"]},
    "input": {"type": "object"},
    "limits": {
      "type": "object",
      "properties": {
        "deadline_seconds": {"type": "integer", "minimum": 1},
        "max_steps": {"type": "integer", "minimum": 1},
        "max_cost_usd": {"type": "number", "minimum": 0},
        "max_tokens": {"type": "integer", "minimum": 1}
      }
    }
  }
}
```

### 9.3 ExecutionContext Example

```json
{
  "run_id": "run_01J",
  "tenant": {"tenant_id": "tenant_acme"},
  "user": {"user_id": "user_123", "roles": ["user"]},
  "session": {"session_id": "sess_456"},
  "trace_id": "trace_789",
  "correlation_id": "corr_abc",
  "limits": {"deadline_seconds": 900, "max_cost_usd": 5.0},
  "resolved_revisions": [
    {"kind": "agent", "id": "researcher", "revision": "agentrev_1"}
  ]
}
```

### 9.4 RuntimeEvent Example

```json
{
  "event_id": "evt_01J",
  "type": "tool.completed",
  "schema_version": "1.0",
  "tenant_id": "tenant_acme",
  "user_id": "user_123",
  "session_id": "sess_456",
  "run_id": "run_01J",
  "step_id": "step_002",
  "attempt_id": "attempt_001",
  "sequence": 42,
  "trace_id": "trace_789",
  "timestamp": "2026-06-07T00:00:00Z",
  "payload": {
    "tool_id": "web_search",
    "status": "success",
    "duration_ms": 321
  }
}
```

### 9.5 ErrorResponse Example

```json
{
  "error": {
    "code": "TOOL_AUTHORIZATION_DENIED",
    "message": "You are not allowed to execute this tool.",
    "category": "authorization",
    "retryable": false,
    "trace_id": "trace_789",
    "details": {
      "resource": "tool:shell_run"
    }
  }
}
```

### 9.6 Event Type Catalog

All platform components must emit the following event types where applicable. Events follow dot-notation namespacing.

| Namespace | Event types |
|---|---|
| Run lifecycle | `run.submitted`, `run.queued`, `run.started`, `run.completed`, `run.failed`, `run.cancelled`, `run.timed_out` |
| Step lifecycle | `step.started`, `step.completed`, `step.failed`, `step.retrying`, `step.waiting`, `step.resumed`, `step.skipped` |
| Agent execution | `agent.turn_started`, `agent.turn_completed`, `agent.delegated` |
| Tool execution | `tool.started`, `tool.completed`, `tool.failed`, `tool.approval_required` |
| Skill execution | `skill.started`, `skill.step_completed`, `skill.completed`, `skill.failed` |
| LLM invocation | `llm.request_started`, `llm.request_completed`, `llm.request_failed`, `llm.fallback` |
| Workflow | `workflow.started`, `workflow.step_completed`, `workflow.paused`, `workflow.resumed`, `workflow.completed`, `workflow.failed`, `workflow.compensating` |
| Human approval | `approval.requested`, `approval.decided`, `approval.expired` |
| Memory | `memory.written`, `memory.retrieved`, `memory.deleted` |
| Retrieval | `retrieval.searched`, `retrieval.results_returned` |
| Streaming | `stream.chunk`, `stream.terminal` |
| System | `system.health_changed`, `system.adapter_loaded`, `system.adapter_failed` |

Every event must carry the identifiers specified in Section 20.2 where applicable.

---

## 10. Request Execution Flow

### 10.1 Complete Request Lifecycle

1. Request received from CLI/TUI/Web/API/SDK.
2. API authenticates caller.
3. Authorization checks caller can submit this request.
4. Tenant, user, session, and conversation are resolved.
5. Input schema, size, content type, and idempotency key are validated.
6. Intent is classified and routed if target is not explicit.
7. Planner or orchestrator is selected if decomposition is needed.
8. Agent runtime, workflow runtime, skill runtime, or tool runtime is selected.
9. Tool and skill access policy filters available capabilities.
10. Memory and retrieval access policy loads authorized context.
11. Execution starts with streaming events where requested.
12. Human approval is requested if policy requires it.
13. Errors are classified, retried, compensated, or surfaced.
14. Final response is formatted and validated.
15. Conversation, run, events, artifacts, effects, memory, and usage are persisted.
16. Observability and audit records are emitted.

### 10.2 Text Sequence Diagram

```text
Client
  -> API: submit request
  -> AuthProvider: authenticate
  -> AuthorizationPolicy: authorize run.submit
  -> TenantResolver: resolve tenant/user/session
  -> RunService: create durable run
  -> EventBus: append run.submitted
  -> Scheduler: enqueue first step
  -> Worker: lease step
  -> ReferenceMonitor: authorize step
  -> Router/Planner: route or plan if needed
  -> AgentRuntime: invoke agent
  -> Memory/Retriever: load authorized context
  -> LLMGateway: model call
  -> ToolExecutor/SkillExecutor: governed capability call if requested
  -> ApprovalService: pause/resume if approval required
  -> StateStore: persist progress
  -> EventBus: stream events
  -> AuditSink: append audit records
  -> Client: receive chunks/final response
```

### 10.3 Synchronous Execution Flow

Synchronous execution is allowed for bounded, short-running operations.

Requirements:

- Request timeout is short and explicit.
- Execution still uses auth, policy, runtime, state, and observability.
- If execution exceeds sync limit, platform returns async run reference.
- Side-effecting operations still require idempotency and effect records.

### 10.4 Streaming Execution Flow

Streaming returns incremental chunks and events.

Requirements:

- Stream has event IDs and cursor.
- Client can reconnect with cursor.
- Terminal event is explicit.
- Stream is not authoritative state; durable events are.
- Authorization is rechecked on reconnect.

Streaming wire protocols:

SSE (Server-Sent Events) for one-way streaming:

```text
Format:   event: <event_type>\nid: <sequence_number>\ndata: <JSON RuntimeEvent>\n\n
Reconnect: Client sends Last-Event-ID header with last received sequence.
Buffer:    Server maintains per-run event buffer (default 1000 events, configurable).
Cursor gap: If requested cursor is behind buffer start, return HTTP 410 Gone
            with earliest available cursor in response body.
Terminal:  event: run.terminal\ndata: {"status": "completed|failed|cancelled"}\n\n
Keep-alive: Server sends comment line (: keepalive\n\n) every 15 seconds.
```

WebSocket for bidirectional interactive sessions:

```text
Format:         JSON messages with type field.
Client -> Server: user input, approval decisions, cancel requests.
Server -> Client: RuntimeEvent stream.
Auth:           Validated on connection upgrade and re-validated on reconnect.
```

Authorization is re-checked on every reconnect. Expired tokens cause disconnect with 401.

Stream is not authoritative state. Durable run events in EventBus are authoritative. Clients that need guaranteed delivery must query the EventBus replay endpoint.

### 10.5 Long-Running Workflow Flow

Long-running workflows:

1. Persist workflow execution.
2. Queue first step.
3. Worker executes step under lease.
4. Checkpoint after configured boundary.
5. Pause for timers, approvals, or external signals.
6. Resume after signal with fresh policy check.
7. Retry or compensate failures.
8. Complete with final state and artifacts.

### 10.6 Human Approval Flow

1. Reference monitor returns `require_approval`.
2. Runtime creates `ApprovalRequest` with exact action hash.
3. Run moves to `waiting_for_approval`.
4. Approval event is streamed.
5. Authorized approver decides.
6. Decision is persisted and audited.
7. Runtime validates action hash and expiry.
8. Approved action resumes; rejected action stops, replans, or compensates.

### 10.7 Failed Execution and Retry Flow

1. Runtime catches error.
2. Error is normalized into platform error.
3. Retry policy is evaluated.
4. If retryable and safe, scheduler queues retry with backoff.
5. If side effect is uncertain, verification or reconciliation runs first.
6. If non-retryable, step fails.
7. Workflow decides fail, compensate, or request human input.
8. Run terminal state and error are persisted.

### 10.8 State Machines

All runtime state transitions must be explicit and enforced. Invalid transitions must be rejected with `StateTransitionError`.

Run states:

```text
submitted -> queued -> running -> completed
submitted -> queued -> running -> failed
submitted -> queued -> running -> waiting_for_approval -> running -> completed | failed
submitted -> queued -> running -> timed_out
submitted -> cancelled                          (pre-start cancellation)
running -> cancelled                             (in-flight cancellation)

Terminal states: completed, failed, cancelled, timed_out
Non-terminal states: submitted, queued, running, waiting_for_approval
```

Step states:

```text
pending -> leased -> running -> completed
pending -> leased -> running -> failed -> retrying -> pending    (retry loop)
pending -> leased -> running -> waiting -> resumed -> running
pending -> skipped

Terminal states: completed, failed (non-retryable), skipped
Non-terminal states: pending, leased, running, waiting, retrying
```

Workflow execution states:

```text
pending -> running -> paused -> running -> completed
pending -> running -> failed -> compensating -> compensated
pending -> running -> cancelled

Terminal states: completed, failed, compensated, cancelled
Non-terminal states: pending, running, paused, compensating
```

Every state transition must:

- Persist the new state before acknowledging.
- Emit a `RuntimeEvent` with the old and new state.
- Validate the transition is allowed.
- Check policy and limits at resume boundaries.

---

## 11. Folder Structure Recommendation

### 11.1 Recommended Structure

```text
src/
  universal_agentic_platform/
    core/
      contracts/
      errors/
      events/
      schemas/
    domain/
      agents/
      tools/
      skills/
      workflows/
      memory/
      retrieval/
      security/
      tenants/
    application/
      execution/
      sessions/
      approvals/
      administration/
    runtime/
      agent_runtime/
      orchestration_runtime/
      workflow_runtime/
      tool_runtime/
      skill_runtime/
      llm_gateway/
      planning/
      routing/
    adapters/
      agent_frameworks/
      llm_providers/
      auth/
      authorization/
      memory/
      retrieval/
      storage/
      observability/
      events/
      ui/
    infrastructure/
      database/
      cache/
      queue/
      object_store/
      secrets/
      migrations/
    api/
      rest/
      websocket/
      sse/
      middleware/
      schemas/
    clients/
      cli/
      tui/
      web/
      sdk/
    security/
    observability/
    evaluation/
    config/
tests/
  unit/
  integration/
  contract/
  adapter/
  api/
  e2e/
  load/
deploy/
  docker/
  compose/
  kubernetes/
docs/
examples/
```

### 11.2 Folder Rules

| Folder | Purpose | Belongs there | Must not belong there |
|---|---|---|---|
| `core/contracts` | Stable interfaces. | Protocols/ports. | Provider SDKs, DB code. |
| `core/errors` | Platform error taxonomy. | Error classes/codes. | HTTP-specific responses only. |
| `core/events` | Event types and schemas. | Runtime event contracts. | Telemetry backend code. |
| `domain` | Domain models and invariants. | Agent/tool/workflow/memory models. | API handlers. |
| `application` | Use-case services. | Run service, session service, admin service. | Provider SDK calls. |
| `runtime` | Execution engines. | Agent/workflow/tool/skill/model runtimes. | UI rendering. |
| `adapters` | External implementations. | Provider/framework/backend bridges. | Core business rules. |
| `infrastructure` | Concrete persistence and infra. | Repositories, queue, cache, migrations. | Agent reasoning. |
| `api` | External protocol layer. | Routes, middleware, API schemas. | Tool execution logic. |
| `clients` | Thin clients. | CLI/TUI/Web/SDK. | Runtime orchestration. |
| `security` | Security services. | Policy, sandbox, redaction, egress. | UI state. |
| `observability` | Telemetry and audit. | Logging, metrics, traces, audit writers. | Business decisions. |
| `evaluation` | Evaluation framework. | Evals, graders, reports. | Production request handling. |
| `config` | Typed config and composition. | Config models, validation, loader. | Secrets in plaintext. |
| `tests` | Test suites. | Unit, contract, integration, e2e, load. | Production code. |
| `deploy` | Deployment assets. | Docker, Compose, K8s. | Runtime source code. |
| `docs` | Human documentation. | Architecture/API/ops docs. | Secrets. |
| `examples` | Reference implementations. | Example agents/tools/skills. | Required production code. |

---

## 12. API Design

### 12.1 API Versioning

- All APIs are versioned under `/api/v1` or equivalent.
- Breaking changes require `/api/v2` or compatible version negotiation.
- SDKs pin supported API versions.
- Errors include API version and trace ID.

### 12.2 API Groups

| Group | Purpose | Example endpoints | Auth | Streaming | Tests |
|---|---|---|---|---|---|
| Chat/execution | Submit bounded agent requests. | `POST /api/v1/executions`, `GET /api/v1/executions/{id}` | User/service. | Optional. | Request/response, auth, idempotency. |
| Streaming execution | Stream run events/chunks. | `GET /api/v1/executions/{id}/stream` | Run read permission. | SSE/WebSocket. | Reconnect, cursor, terminal event. |
| Workflow execution | Start/signal/cancel workflows. | `POST /api/v1/workflows/{id}/runs`, `POST /runs/{id}/signal` | Workflow permissions. | Yes. | Long-run, retry, approval. |
| Session management | Manage sessions/conversations. | `POST /sessions`, `GET /sessions`, `DELETE /sessions/{id}` | User. | No. | Isolation, pagination. |
| Tool registry | Manage tools. | `GET /tools`, `POST /tools`, `POST /tools/{id}/disable` | Admin/tool author. | No. | Schema, version, permissions. |
| Skill registry | Manage skills. | `GET /skills`, `POST /skills`, `POST /skills/{id}/test` | Admin/skill author. | Optional. | Dependencies, evals. |
| Agent registry | Manage agents. | `GET /agents`, `POST /agents`, `POST /agents/{id}/activate` | Admin/agent author. | No. | Version/eval gates. |
| Memory | Read/write governed memory. | `GET /memory`, `POST /memory`, `DELETE /memory/{id}` | Memory permissions. | No. | Scope isolation, deletion. |
| Knowledge/RAG | Ingest/search sources. | `POST /knowledge/sources`, `POST /knowledge/search` | Knowledge permissions. | Optional ingestion events. | Access filters, citations. |
| Human approval | List/decide approvals. | `GET /approvals`, `POST /approvals/{id}/decide` | Approver. | Events. | Hash binding, expiry. |
| Admin/config | Manage tenants/policies/config. | `GET /admin/tenants`, `POST /admin/policies` | Admin. | No. | RBAC/ABAC, audit. |
| Health checks | Liveness/readiness. | `/health/live`, `/health/ready` | Public/live, protected/deep. | No. | Dependency failure. |
| Metrics | Expose metrics. | `/metrics` | Ops/auth or network-limited. | No. | Scrape, redaction. |
| Evaluation runs | Run/list evals. | `POST /eval/runs`, `GET /eval/runs/{id}` | Eval/admin. | Optional. | Thresholds, reports. |
| Audit logs | Query audit. | `GET /audit` | Audit/admin. | No. | Access control, redaction. |
| Tenant admin | Tenant config/users/quotas. | `POST /tenants`, `PATCH /tenants/{id}` | Platform admin. | No. | Tenant isolation. |

### 12.2.1 API Request, Response, and Error Schemas by Group

| Group | Request schema | Response schema | Error model |
|---|---|---|---|
| Chat/execution | `ExecutionRequest` | `ExecutionResponse` or accepted `RunRecord` | `ErrorResponse` with validation/auth/policy/runtime categories. |
| Streaming execution | cursor, run id, stream options | `RuntimeEvent` stream with terminal event | `CursorGapError`, `AuthorizationError`, `RunNotFoundError`. |
| Workflow execution | `WorkflowStartRequest`, `WorkflowSignalRequest` | `WorkflowExecutionState`, `RunRecord` | `WorkflowValidationError`, `StateTransitionError`. |
| Session management | `SessionCreateRequest`, update/list filters | `SessionContext`, conversation summaries | `SessionAccessError`, `ValidationError`. |
| Tool registry | `ToolDescriptor` draft/revision, activation command | descriptor, validation result, activation record | `RegistryError`, `SchemaValidationError`, `CompatibilityError`. |
| Skill registry | `SkillDescriptor` draft/revision, test command | descriptor, dependency validation, eval result | `DependencyError`, `SkillValidationError`. |
| Agent registry | `AgentDescriptor` draft/revision, activation command | descriptor, eval evidence, activation record | `AgentValidationError`, `EvaluationThresholdError`. |
| Memory | `MemoryWriteRequest`, search filters, delete command | `MemoryRecord`, search result list | `MemoryAccessError`, `RetentionPolicyError`. |
| Knowledge/RAG | source registration, ingest command, `RetrievalRequest` | ingestion run, `RetrievalResult` | `RetrievalAccessError`, `IndexingError`. |
| Human approval | decision command with approval id and action hash | `ApprovalRequest`, `ApprovalDecision` | `ApprovalExpiredError`, `ApprovalHashMismatchError`. |
| Admin/config | typed config/policy/tenant update | updated resource revision or config snapshot | `ConfigValidationError`, `AdminAuthorizationError`. |
| Health checks | none or deep-check query | health status and dependency checks | readiness failure payload. |
| Metrics | scrape request | metrics exposition format | auth/network denial where protected. |
| Evaluation runs | eval suite id, target revisions, config | `EvaluationReport` or eval run state | `EvaluationError`, `ThresholdFailure`. |
| Audit logs | filters, cursor, export format | redacted audit event list/export handle | `AuditAccessError`, `ExportPolicyError`. |
| Tenant administration | tenant create/update, quota/policy commands | tenant config and status | `TenantConflictError`, `TenantSuspendedError`. |

### 12.3 Common Request and Response Patterns

Mutating request headers:

```text
Authorization: Bearer ...
Idempotency-Key: client-generated-key
X-Request-ID: request-id
```

Common success response:

```json
{
  "data": {},
  "meta": {
    "request_id": "req_123",
    "trace_id": "trace_123",
    "api_version": "v1"
  }
}
```

Common paginated response:

```json
{
  "data": [],
  "page": {
    "limit": 50,
    "next_cursor": "cursor_abc"
  }
}
```

### 12.4 Error Model

All API errors use `ErrorResponse` from Section 9.

Required tests:

- Invalid schema.
- Missing auth.
- Authenticated but unauthorized.
- Cross-tenant access.
- Idempotency replay.
- Rate limit.
- Internal error redaction.

### 12.5 Idempotency Semantics

All mutating API requests should support optional idempotency keys.

Idempotency key contract:

| Property | Value |
|---|---|
| Format | Client-generated opaque string, max 128 characters. |
| Scope | Unique per (tenant_id, idempotency_key). |
| TTL | 24 hours by default, configurable per tenant. |
| Match with terminal run | Return stored `RunRecord` and final result. |
| Match with active run | Return current `RunRecord` with in-progress status. |
| No match | Create new run normally. |
| Concurrent duplicates | First request wins. Subsequent requests receive 409 Conflict or the in-progress `RunRecord`. |
| Storage | `IdempotencyStore` with run_id, key, tenant_id, created_at, expires_at. |

Required tests:

- Duplicate key returns same `RunRecord`.
- Expired key allows new run.
- Concurrent duplicate requests do not create two runs.
- Cross-tenant key collision is impossible.

---

## 13. Runtime Modes

| Mode | Purpose | Required components | Disabled components | Config differences | Deployment | Risks | Recommended defaults |
|---|---|---|---|---|---|---|---|
| Local developer | Fast iteration. | API, worker, local DB, mock auth, echo model. | Production auth, external secrets optional. | Debug logs, synthetic data. | Single process or Compose. | Unsafe if exposed. | Bind localhost, warn non-prod. |
| Single-user local | Personal assistant. | API, worker, SQLite/Postgres, local files, local auth. | Multi-tenant admin optional. | Persistent local state. | Docker Compose/local. | Weak isolation. | No production credentials. |
| Server multi-user | Team/org production. | API, workers, DB, queue, cache, object store, auth, secrets, observability. | None. | Strict auth, tenant config. | Compose or Kubernetes. | Ops burden. | Fail closed. |
| Worker | Background execution. | Queue, runtime, adapters, state, secrets. | API routes, UI. | Queue names, concurrency. | Container/process. | Poison jobs. | Leases, heartbeats. |
| Batch/job | Scheduled bulk work. | Worker runtime, scheduler, eval or ingestion services. | Interactive UI optional. | Batch limits. | Job container. | Resource contention. | Separate queues. |
| Evaluation | Quality and regression. | Eval runner, fixtures, model/tool adapters. | Production side effects by default. | Deterministic fixtures. | CI/job. | Eval cost/flakiness. | Synthetic data. |
| Offline | Air-gapped/local-only. | Local models, local stores, local auth, local audit. | Cloud providers. | No external egress. | On-prem/local. | Update and model limits. | Signed bundles. |
| Hybrid local/cloud | Local runtime with cloud models/tools. | Local API, cloud LLM adapter, local state. | Multi-user production features optional. | Secret refs, cloud egress. | Local/Compose. | Credential leakage. | Scoped dev credentials. |

Production modules must not depend on local-only behavior. Local mode may disable controls only when explicitly marked non-production.

---

## 14. Multi-User and Multi-Tenant Design

### 14.1 Tenant Model

```yaml
Tenant:
  tenant_id: string
  name: string
  status: active | suspended | deactivated
  data_residency: string | null
  encryption_policy: object
  quotas: object
  rate_limits: object
  cost_limits: object
  model_routes: object
  allowed_tools: [string]
  allowed_skills: [string]
  policies: [string]
  audit_level: minimal | standard | verbose
```

### 14.2 Isolation Requirements

| Area | Requirement |
|---|---|
| Database | Tenant-owned rows include tenant scope or equivalent partition. |
| Object storage | Tenant-scoped prefixes/buckets/policies. |
| Vector indexes | Tenant namespace or hard filter before retrieval. |
| Cache | Tenant-prefixed keys and no cross-tenant shared mutable values. |
| Queue | Tenant and priority metadata; no tenant starvation. |
| Audit | Tenant-scoped query permissions. |
| Secrets | Tenant-scoped secret paths and access. |
| Telemetry | Tenant labels with access-controlled dashboards. |
| Admin | Tenant admins cannot access other tenants. |

### 14.3 Users, Roles, and Permissions

Baseline roles:

- `viewer`: read own allowed sessions/runs.
- `user`: execute allowed agents/tools/skills.
- `power_user`: execute higher-risk capabilities subject to approval.
- `tenant_admin`: manage tenant users, resources, policies.
- `platform_admin`: manage platform-wide config and tenants.
- `auditor`: read audit/evidence within scope.

Permissions are evaluated with RBAC plus ABAC.

### 14.4 Per-User and Per-Tenant State

- Sessions are scoped to user and tenant.
- Conversation state is scoped to conversation, user, and tenant.
- User memory is visible only under user/tenant policy.
- Tenant memory requires tenant policy and provenance.
- Tools, skills, model settings, policies, quotas, and audit logs may vary by tenant.

### 14.5 Rate, Cost, and Quota Controls

Controls must exist at:

- Platform level.
- Tenant level.
- User level.
- Agent/workflow level.
- Run level.
- Tool/skill/model provider level.

Limits include requests/minute, concurrent runs, queue depth, tokens, cost/day, cost/month, storage, retrieval calls, and dangerous tool calls.

---

## 15. State, Memory, and Persistence Design

### 15.1 Persistence Strategy Matrix

| Data | Local mode | Production mode | Enterprise mode |
|---|---|---|---|
| Conversation state | SQLite/Postgres | PostgreSQL | PostgreSQL partitioned/replicated |
| Workflow state | SQLite/Postgres | PostgreSQL + queue | PostgreSQL + Temporal or durable workflow engine |
| Checkpoints | Local files/DB | DB + object store | Object store + DB metadata |
| Agent state | In-memory for active turn, persisted run state | DB/cache | DB/cache with HA |
| Tool execution logs | SQLite/Postgres | PostgreSQL | Partitioned DB + archive |
| Skill execution logs | SQLite/Postgres | PostgreSQL | Partitioned DB + archive |
| User memory | SQLite/Postgres | PostgreSQL | PostgreSQL + policy index |
| Long-term memory | SQLite + local vector | PostgreSQL + vector DB | Dedicated vector/search cluster |
| Vector indexes | FAISS/Chroma/pgvector | Qdrant/pgvector/Weaviate | Clustered vector DB |
| Document store | Local filesystem | S3/MinIO/GCS | Versioned object store |
| Relational data | SQLite/Postgres | PostgreSQL | HA PostgreSQL |
| Audit logs | Local DB | Append-only DB/object archive | Tamper-evident archive |
| Event logs | DB table | DB outbox/Redis Streams/Kafka | Kafka/NATS + archive |
| Cache | In-memory | Redis | Redis Cluster |
| Idempotency keys | SQLite/Postgres | PostgreSQL/Redis | PostgreSQL + Redis |

### 15.2 Required Persistence Entities

- Tenants.
- Principals and role assignments.
- Resource definitions and revisions.
- Active resource revisions.
- Runs.
- Steps.
- Attempts.
- Events.
- Outbox/inbox.
- Approvals.
- Effects.
- Artifacts.
- Memory records.
- Knowledge sources/chunks.
- Usage/cost records.
- Audit events.
- Config and feature flags.
- Idempotency records.

### 15.3 Outbox/Inbox

Use transactional outbox when state change and event publication must be consistent.

Required boundaries:

- Run creation and `run.submitted`.
- Step completion and successor enqueue.
- Approval decision and wake command.
- Effect record and audit event.
- Resource activation and release event.

### 15.4 Cleanup and Retention

Retention policies must define:

- Conversation retention.
- Memory expiry.
- Artifact retention.
- Audit retention.
- Event retention.
- Cache TTL.
- Vector deletion propagation.
- Backup expiry.
- Legal hold.
- User deletion/export/correction.

---

## 16. Tool and Skill Design

### 16.1 Tool Definition

A tool is a low-level callable operation with a narrow schema and bounded behavior.

Example low-level tool:

```yaml
ToolDescriptor:
  tool_id: web_search
  revision: toolrev_001
  description: Search approved web indexes.
  input_schema:
    type: object
    required: [query]
    properties:
      query: {type: string, maxLength: 500}
      limit: {type: integer, minimum: 1, maximum: 20}
  output_schema:
    type: object
    properties:
      results: {type: array}
  permission: web.search
  side_effect_class: read
  timeout_seconds: 20
  retryable: true
```

Side effect class values:

```yaml
SideEffectClass:
  enum: [none, read, write, delete, external_read, external_write, dangerous]
```

`none`: Pure computation with no external effects. `read`: Reads platform-internal data. `write`: Writes platform-internal data. `delete`: Deletes platform-internal data. `external_read`: Reads from external systems. `external_write`: Writes to external systems. `dangerous`: Requires explicit approval and may need compensation.

### 16.2 Skill Definition

A skill is a higher-level reusable capability that may compose tools, prompts, model calls, retrieval, and logic.

Example skill:

```yaml
SkillDescriptor:
  skill_id: research_topic
  revision: skillrev_001
  description: Research a topic and produce a cited summary.
  required_tools: [web_search, fetch_url, summarize_text]
  required_model_routes: [default_reasoning]
  input_schema:
    type: object
    required: [topic]
  output_schema:
    type: object
    required: [summary, citations]
  permissions: [skill.research_topic]
```

Skill composed from tools:

```text
research_topic skill
  -> web_search tool
  -> fetch_url tool
  -> summarize_text tool or LLM call
  -> citation formatter
  -> output validator
```

### 16.3 Registration and Versioning

Tools and skills:

- Register as drafts.
- Validate schemas and config.
- Run contract tests.
- Run security review for side effects.
- Create immutable revision.
- Activate via release manifest.
- Disable through policy or kill switch.

### 16.4 Permission Enforcement

- Tool permission is checked before each invocation.
- Skill permission is checked before skill execution.
- Skill internal tool calls still go through tool runtime and policy.
- Dangerous tools require approval unless policy explicitly prohibits or tightly permits.

### 16.5 Failure Handling

- Validation failure: no execution.
- Timeout: cancel and classify retryability.
- Retryable external error: retry with backoff if safe.
- Side-effect uncertainty: verify or reconcile.
- Non-retryable error: fail step and workflow decides next transition.

### 16.6 Required Tests

- Schema validation.
- Authorization.
- Tenant isolation.
- Timeout.
- Retry.
- Cancellation.
- Output validation.
- Sandbox.
- Dangerous approval.
- Version activation/disable.
- Eval/scenario tests for skills.

---

## 17. Workflow and Orchestration Design

### 17.1 Supported Execution Patterns

- Simple request/response agents.
- Multi-step workflows.
- Graph-based orchestration.
- Planner-executor pattern.
- Human approval steps.
- Long-running workflows.
- Retryable workflows.
- Checkpointed workflows.
- Scheduled workflows.
- Event-driven workflows.
- Organization-specific workflow engines.
- External orchestrators through adapters.

### 17.2 Core Distinctions

| Concept | Meaning |
|---|---|
| Platform orchestration runtime | Governs lifecycle, state, policy, events, retries, checkpoints. |
| External framework adapter | Bridges framework-specific semantics into platform contracts. |
| Workflow definition | Versioned step graph/state machine. |
| Workflow execution state | Durable state of one workflow run. |
| Agent execution | One or more model-driven turns within limits. |
| Tool execution | Atomic governed operation. |
| Skill execution | Composite governed capability. |

### 17.3 Framework Support Without Coupling

| Framework | Integration approach |
|---|---|
| LangGraph | Adapter maps platform workflow/agent calls to graph execution; tools remain platform-wrapped. |
| Google ADK | Adapter wraps ADK agents and maps sessions/tools through platform services. |
| AutoGen | Adapter maps multi-agent conversations to platform runs and events. |
| CrewAI | Adapter maps crews/tasks/process to workflow steps and agent descriptors. |
| OpenAI Agents SDK | Adapter maps agents, handoffs, and tools through platform contracts. |
| LlamaIndex workflows | Adapter maps workflows/retrievers to platform workflow and retrieval contracts. |
| Custom orchestrator | Implements `OrchestratorAdapter` or `WorkflowAdapter`. |

### 17.4 Durable Workflow Requirements

- State persisted before and after side-effecting steps.
- Steps have timeouts and retry policy.
- Human waits release worker resources.
- Resume revalidates policy.
- Checkpoints have schema versions.
- Duplicate step execution is fenced.
- Compensation is defined for reversible side effects.

---

## 18. LLM Gateway Design

### 18.1 Provider Model

The platform supports multiple providers:

- Hosted APIs.
- Local models.
- Organization-specific model gateways.
- Embedding models.
- Multimodal models.
- Specialized fine-tuned models.

### 18.2 Model Routing

Routing inputs:

- Tenant model policy.
- Required capabilities.
- Data classification.
- Residency.
- Cost budget.
- Latency target.
- Provider health.
- Fallback policy.
- Eval approval status.

### 18.3 Gateway Pipeline

```text
LLMRequest
  -> input validation
  -> prompt/context assembly
  -> data policy
  -> rate limit and budget check
  -> provider/model routing
  -> provider adapter call
  -> streaming or response collection
  -> output validation/guardrails
  -> usage and cost record
  -> trace/event/audit metadata
```

### 18.4 Required Features

- Streaming.
- Structured outputs.
- Tool calling.
- Prompt templates.
- Prompt/version management.
- Token counting.
- Cost tracking.
- Provider health checks.
- Retries with backoff.
- Fallback with policy constraints.
- Timeout and cancellation.
- Capability discovery.
- Output validation.
- Guardrails and safety filters.

### 18.5 Fallback Rules

Fallback must not silently violate:

- Data residency.
- Provider allowlist.
- Required output schema.
- Required tool-calling capability.
- Cost budget.
- Safety/eval approval.

---

## 19. Security Design

### 19.1 Security Architecture

Security is enforced at API, runtime, data, adapter, and infrastructure boundaries.

Controls:

- Authentication.
- Authorization.
- RBAC.
- ABAC.
- Tenant isolation.
- Session isolation.
- Tool permissions.
- Skill permissions.
- Data permissions.
- Secrets management.
- Sandboxing.
- Prompt injection protection.
- Retrieval security.
- Audit logs.
- Secure config.
- Input validation.
- Output validation.
- PII handling.
- Least privilege.
- Dangerous tool restrictions.
- Approval gates.
- Network egress control.
- File access control.

### 19.2 Prompt Injection Protection

Prompt injection is handled through:

- Trust labeling for system prompts, user input, retrieved content, tool output, and memory.
- Capability filtering before model calls.
- Re-authorization of all model-proposed actions.
- Egress restrictions.
- Output validation.
- Tool call consistency checks.
- Injection eval suites.

Detectors may help but are not sufficient.

### 19.2.1 Trust Labeling Model

Every message block in an LLM request carries trust metadata:

```yaml
MessageBlock:
  role: system | user | assistant | tool
  content: string
  trust_level: trusted | untrusted | verified
  source: system_prompt | user_input | tool_output | memory | retrieval | agent_output
  data_classification: public | internal | confidential | restricted
```

Trust level rules:

| Trust level | Assigned to | Meaning |
|---|---|---|
| `trusted` | System prompts authored by platform or tenant admin. | Platform-controlled content that may grant capabilities. |
| `untrusted` | User input, tool output, memory content, retrieval chunks, agent-generated content. | Content that must not be allowed to escalate privileges or bypass policy. |
| `verified` | Content that passed a platform verification step (hash-checked, admin-reviewed). | Higher confidence but still subject to policy checks. |

LLM gateway must:

- Never promote `untrusted` content to `trusted`.
- Strip or escape injection-like patterns in `untrusted` blocks before assembly.
- Log `trust_level` and `source` in LLM trace events.
- Filter available tools and capabilities based on trust context, not only based on model output.
- Treat model output as `untrusted` for all downstream authorization checks.

### 19.3 Sandboxing

Sandbox levels:

- None for pure operations.
- Process for low-risk local transformations.
- Container for filesystem/network operations.
- MicroVM or external isolated service for untrusted code/high-risk actions.

Sandbox controls:

- CPU/memory/time/process limits.
- Read-only filesystem by default.
- Explicit mounts.
- Default-deny network.
- Destination allowlist.
- No host credentials.
- Ephemeral cleanup.

### 19.4 Secrets

Secrets:

- Stored only in secret provider.
- Referenced by config.
- Issued as scoped credentials.
- Never placed in model context by default.
- Redacted from logs, traces, events, errors, artifacts.
- Audited on access.

---

## 20. Observability Design

### 20.1 Observability Dimensions

| Dimension | Required data |
|---|---|
| Structured logs | Component, event, tenant, user, run, step, trace, safe payload. |
| Metrics | Counts, latency, errors, retries, queue depth, cost, tokens, approvals. |
| Distributed traces | API -> runtime -> model/tool/skill/workflow spans. |
| Agent traces | Agent turn, prompt refs, model route, tool requests. |
| Tool traces | Input schema result, sandbox, effect, output validation. |
| Skill traces | Internal steps and dependency calls. |
| LLM traces | Provider, model, tokens, cost, latency, fallback. |
| Workflow traces | State transitions, checkpoints, retries, waits. |
| Audit events | Security/governance actions. |
| Debug replay | Recorded event sequence and artifact references. |
| Run history | Human-readable timeline. |
| Evaluation dashboards | Eval results, regressions, quality trends. |

### 20.2 Required Identifiers

Telemetry must include where applicable:

- `tenant_id`
- `user_id`
- `session_id`
- `conversation_id`
- `run_id`
- `step_id`
- `attempt_id`
- `trace_id`
- `span_id`
- `correlation_id`
- `resource_id`
- `resource_revision`

### 20.3 Minimum Metrics

- API request count and latency.
- Active runs.
- Run completion/failure/cancellation count.
- Queue depth and wait time.
- Worker lease loss.
- Tool/skill execution count and latency.
- LLM tokens and cost.
- Provider errors.
- Retry counts.
- Approval pending count and latency.
- Budget remaining.
- Policy denies.
- Sandbox violations.
- Tenant usage.

---

## 21. Evaluation and Testing Strategy

### 21.1 Test Matrix

| Area | What to test | Test types | Example cases | Minimum acceptance |
|---|---|---|---|---|
| Core models | Schema, serialization, compatibility. | Unit/contract. | Unknown fields, version migration. | No breaking change without versioning. |
| Agent runtime | Invocation, streaming, limits, delegation. | Unit/integration/eval. | Max turns, child agent limit. | Limits enforced. |
| Orchestration | Plans, graphs, retries, checkpoints. | Integration/golden/failure. | Worker crash mid-step. | Recovery works. |
| Workflow runtime | Long waits, approval, resume. | E2E/failure. | Approval timeout. | Durable terminal state. |
| Tool runtime | Auth, sandbox, schemas, effects. | Unit/integration/security. | Dangerous tool approval. | No bypass. |
| Skill runtime | Dependencies, scenarios, failures. | Integration/eval. | Missing required tool. | Clear failure. |
| LLM gateway | Routing, fallback, tokens, cost. | Unit/integration. | Provider outage. | Policy-safe fallback. |
| Memory | Scope, write policy, deletion. | Unit/integration/security. | Cross-user memory access. | Denied. |
| Retrieval | Access filters, citations, deletion. | Integration/security/eval. | Cross-tenant document. | No leakage. |
| Auth/security | RBAC, ABAC, tenant isolation. | Security/e2e. | Tenant admin reads other tenant. | Denied. |
| API | Schemas, errors, pagination, idempotency. | API/e2e. | Duplicate idempotency key. | Same result. |
| CLI/TUI/Web | API usage, streaming, errors. | Client/e2e. | Reconnect stream. | Correct resume. |
| Observability | Events, traces, redaction. | Integration. | Secret in tool env. | Not logged. |
| Deployment | Containers, health, migrations. | Docker/deployment. | DB unavailable. | Readiness fails. |
| Load | Concurrency, backpressure. | Load. | Tenant noisy neighbor. | Quotas enforced. |

### 21.2 Required Test Types

The platform must include:

- Unit tests.
- Integration tests.
- Contract tests.
- Adapter tests.
- API tests.
- CLI tests.
- TUI tests.
- Web tests.
- Streaming tests.
- Orchestration tests.
- Tool execution tests.
- Skill execution tests.
- Memory tests.
- Retrieval tests.
- Auth tests.
- Security tests.
- Multi-user tests.
- Multi-tenant tests.
- Concurrency tests.
- Failure tests.
- Regression tests.
- Golden trace tests.
- Agent evaluation tests.
- Prompt evaluation tests.
- Load tests.
- Docker/deployment tests.

### 21.3 Golden Trace Tests

Golden trace tests verify expected event order for canonical flows:

- Simple agent response.
- Agent uses tool.
- Tool requires approval.
- Workflow retries.
- Workflow compensates.
- Streaming reconnect.

### 21.4 Release Gates

No production activation unless:

- Contract tests pass.
- Security tests pass for changed area.
- Required eval suites pass.
- Migration tests pass if schema changes.
- Rollback plan exists.
- Observability and audit events are verified.

---

## 22. Deployment Architecture

### 22.1 Local Deployment

Components:

- API process.
- Worker process or thread.
- SQLite/Postgres.
- Local object directory.
- In-process/database queue.
- Echo/local model adapter.
- Mock/local auth.

### 22.2 Docker Compose Production Deployment

Containers:

- `api`
- `worker`
- `web`
- `postgres`
- `redis` or queue service.
- `qdrant` or vector DB.
- `minio` or object storage.
- `otel-collector`
- `prometheus`
- `grafana`
- secret provider or mounted secrets.

### 22.3 Kubernetes Readiness

Kubernetes deployment should include:

- API deployment with readiness/liveness probes.
- Worker deployments by queue/risk class.
- Web deployment.
- ConfigMaps for non-secret config.
- Secrets/secret-store integration.
- Jobs for migrations.
- Horizontal pod autoscaling.
- Pod disruption budgets.
- Network policies.
- Persistent external managed stores or stateful sets where appropriate.

### 22.4 Health Checks

- `/health/live`: process is alive.
- `/health/ready`: dependencies required for traffic are available.
- `/health/deep`: authenticated operational diagnostics.

Readiness must fail when required DB, queue, config, or auth provider is unavailable.

### 22.5 Graceful Shutdown

API:

- Stop accepting new requests.
- Complete short requests or return retryable errors.
- Keep health accurate.

Worker:

- Stop leasing new steps.
- Heartbeat in-flight work.
- Finish or checkpoint.
- Release/expire leases.

### 22.6 Backup and Restore

Backup:

- Database.
- Object storage.
- Secrets metadata and key recovery process.
- Config/release manifests.
- Audit archive.

Restore tests must prove runs, artifacts, policies, audit, and tenant data recover within declared RTO/RPO.

---

## 23. Configuration Management

### 23.1 Config Types

- Environment config.
- Runtime mode config.
- Adapter config.
- Provider config.
- Tenant config.
- User preferences.
- Feature flags.
- Policy config.
- Quota/cost config.
- Deployment config.

### 23.2 Precedence

```text
platform defaults
  -> environment profile
  -> deployment config
  -> tenant config
  -> release manifest
  -> resource revision config
  -> request overrides allowed by policy
```

Later layers may narrow authority, not expand it beyond policy.

### 23.3 Typed Config

All config must be validated against typed schemas before activation.

Invalid production config fails startup or activation.

### 23.4 Secrets

Config contains secret references:

```yaml
api_key_secret_ref: secret://tenant/acme/openai
```

Config must not contain plaintext secrets.

### 23.5 Runtime Reload

Hot reload allowed:

- Routing rules.
- Feature flags.
- Tenant quotas.
- Model provider priority.
- Policy emergency denies.

Restart required:

- Core contract changes.
- Schema migrations.
- Adapter binary changes.
- Sandbox runtime changes.

---

## 24. Error Handling and Failure Recovery

### 24.1 Error Taxonomy

| Category | Examples |
|---|---|
| Validation | Bad schema, unsupported mode, context too large. |
| Authentication | Missing/expired/invalid token. |
| Authorization | Permission denied, tenant mismatch. |
| Adapter | Initialization, compatibility, runtime mapping error. |
| LLM provider | Rate limit, provider timeout, context exceeded, malformed response. |
| Tool execution | Input invalid, sandbox violation, side effect failed. |
| Skill execution | Dependency missing, internal step failed. |
| Workflow | Invalid transition, checkpoint migration failure. |
| Timeout | Request, run, step, model, tool, approval timeout. |
| Rate-limit | Tenant/user/provider limit exceeded. |
| Retrieval | Source unavailable, access denied, index stale. |
| Storage | DB unavailable, object store error, transaction conflict. |
| Partial failure | External effect uncertain, stream interrupted. |
| Internal | Unexpected bug, invariant violation. |

### 24.2 Retryable vs Non-Retryable

Retryable:

- Transient provider timeout.
- Rate limit with retry-after.
- Queue lease loss before effect.
- Network error before confirmed effect.

Non-retryable:

- Auth denied.
- Invalid schema.
- Policy deny.
- Unsupported capability.
- Dangerous action rejected.

Uncertain:

- External side effect may have occurred but confirmation failed.
- Requires verification, reconciliation, or human decision.

### 24.3 Compensation

Compensation is required when:

- Side effect is reversible and later step fails.
- Workflow policy defines rollback.
- User cancels after reversible effects.

Compensation is itself a governed workflow with audit and failure handling.

### 24.4 Dead-Letter Queues

Dead-letter records must include:

- Run/step/attempt.
- Error.
- Last input/output refs.
- Policy decisions.
- Effects.
- Retry history.
- Recommended operator action.

### 24.5 User-Facing Errors

User-facing errors must be safe, actionable, and correlated with trace ID. Internal diagnostics must be redacted and accessible only to authorized operators.

---

## 25. Development Standards

### 25.1 Coding and Typing

- Use explicit types at boundaries.
- Avoid hidden globals.
- Keep functions/classes focused.
- Avoid provider-specific types in core.
- Use structured errors.

### 25.1.1 Engineering Quality Principles

The platform must be implemented as high-quality production software, not as a prototype that happens to pass a demo.

| Principle | Required application |
|---|---|
| SOLID | Keep modules small and cohesive; depend on abstractions; do not force implementers to depend on methods they do not use. |
| Single Responsibility | A class/module has one reason to change. Example: `ToolExecutor` executes governed tools; it does not render UI or manage tenants. |
| Open/Closed | Add providers, frameworks, tools, and stores through adapters, not by modifying core runtime branches. |
| Liskov Substitution | Any adapter satisfying a contract must be replaceable without breaking caller assumptions. Unsupported behavior must be declared, not hidden. |
| Interface Segregation | Split large contracts into focused contracts. A retriever should not implement memory writes unless it is also a memory store. |
| Dependency Inversion | Runtime depends on contracts; concrete adapters are injected by composition/configuration. |
| DRY | Avoid duplicated orchestration, policy, schema, and auth logic across CLI/TUI/Web/API/workers. Shared behavior lives behind services or contracts. |
| KISS | Prefer explicit state machines, schemas, and small services over clever implicit framework magic. |
| YAGNI | Do not build marketplaces, multi-region active/active, or arbitrary code execution before the governed vertical slice works. |
| High cohesion | Keep related behavior together: scheduling in scheduler, authorization in reference monitor, provider calls in adapters. |
| Low coupling | Components communicate through typed contracts and events, not hidden imports, globals, or direct database shortcuts. |
| Fail closed | When policy, config, auth, secrets, sandbox, or tenant resolution is unavailable, protected actions fail closed. |
| Explicit over implicit | State transitions, retries, approval, compensation, and fallback are declared in schemas/config, not hidden in model prompts. |

### 25.1.2 Code Quality Rules

Required:

- Clear names that describe domain intent: `RunService`, `ReferenceMonitor`, `ToolExecutor`, `WorkflowRuntime`.
- Small functions with one level of abstraction where practical.
- No god classes. If a module handles routing, planning, execution, memory, tools, auth, and formatting, split it.
- No copy-pasted policy, schema, or authorization checks. Centralize through contracts.
- No provider-specific objects crossing core boundaries.
- No raw dictionaries at major boundaries unless validated against schema first.
- No hidden background tasks whose failure is ignored.
- No unbounded loops, queues, recursion, fan-out, token usage, retries, or costs.
- No direct secret values in config, tests, logs, traces, or prompts.
- No silent exception handling.
- No hardcoded tenant IDs, model IDs, tool names, role names, or filesystem paths in production code.
- No production behavior that relies on wall-clock sleeps instead of durable timers or scheduler state.

Recommended:

- Prefer immutable value objects for domain records and resource revisions.
- Prefer composition over inheritance for runtime services.
- Prefer dependency injection or explicit composition roots over service locators.
- Prefer table-driven tests for policy, routing, and state transitions.
- Prefer generated clients from OpenAPI/protobuf schemas where feasible.
- Prefer deterministic fixture adapters for tests.

### 25.1.3 Static Analysis and Tooling Gates

Every implementation should configure equivalent tooling for its language:

| Gate | Requirement |
|---|---|
| Formatter | All code formatted by an automated formatter. |
| Linter | CI fails on lint errors, unused code, unsafe patterns, and dependency direction violations. |
| Type checker | Public boundaries and core modules pass static type checks. |
| Import/dependency linter | Enforces core -> no adapters/providers, UI -> no runtime, adapters -> no adapters. |
| Secret scanner | CI fails on committed secrets. |
| Dependency scanner | Known critical vulnerabilities block release unless explicitly waived. |
| Test coverage | Coverage thresholds are set per module risk; security-critical modules require high coverage and negative tests. |
| Schema validation | API, event, config, adapter, and resource schemas are validated in CI. |
| Migration validation | Fresh install and upgrade path are tested. |
| Container scanning | Production images are scanned and use minimal base images. |

### 25.1.4 Code Review Checklist

Every code review must answer:

1. Does this change preserve dependency direction?
2. Does any protected action bypass the reference monitor?
3. Are tenant/user/session/run identifiers propagated correctly?
4. Are inputs and outputs validated at boundaries?
5. Are errors typed, safe for users, and useful for operators?
6. Are retries, timeouts, cancellation, and idempotency explicit?
7. Are secrets redacted and scoped?
8. Are logs/traces/events/audit records emitted at the right boundary?
9. Are tests added at the correct level?
10. Could this change create cross-tenant leakage, unbounded cost, or unobservable side effects?

### 25.2 Interface Rules

- Contracts live in core.
- Implementations live in runtime/adapters/infrastructure.
- Adapters implement contracts and do not define platform semantics.
- Breaking contract changes require version bump.

### 25.3 Dependency Direction Rules

Automated import/dependency linting should enforce:

- Core has no framework/provider dependencies.
- API does not import adapters directly.
- UI does not import runtime.
- Adapters do not import each other.

### 25.4 Error Handling Rules

- No silent `except/pass`.
- All errors classified.
- Retry policy explicit.
- User-facing message safe.
- Internal diagnostics correlated.

### 25.5 Logging Rules

- Structured logs only.
- Include trace/correlation IDs.
- Do not log secrets.
- Avoid full sensitive prompts by default.
- Log policy decisions by reference.

### 25.6 Testing Rules

- Every contract has conformance tests.
- Every adapter runs contract tests.
- Every API route has auth and error tests.
- Every runtime state transition has tests.
- Every security boundary has negative tests.

### 25.7 Documentation Rules

- Public contracts documented.
- Adapter authoring documented.
- Operational runbooks documented.
- Security model documented.
- API documented.

### 25.8 Migration Rules

- Every DB migration has validation.
- Running worker compatibility considered.
- Rollback or no-rollback note required.
- Checkpoint payloads versioned.

Database migration strategy:

- Use a schema migration tool appropriate to the implementation language (e.g., Alembic for Python, Flyway for JVM, golang-migrate for Go).
- Every schema change produces a numbered, forward-only migration file.
- Rollback migrations are optional but must be documented as safe or unsafe.
- Migrations run as a separate step before API and worker startup, never automatically on process start in production.
- Zero-downtime migrations prefer additive changes: add column, then backfill, then add constraint.
- Destructive changes (drop column, rename table) require a multi-step migration across releases.
- Checkpoint schema versions are tracked separately from database schema versions.
- Migration test must verify: fresh install, upgrade from previous version, rollback where documented.

### 25.9 Versioning and Compatibility

- API versioning.
- Contract versioning.
- Adapter versioning.
- Resource revisioning.
- Prompt versioning.
- Workflow versioning.
- Backward compatibility window defined.

### 25.10 Security Review Rules

Security review required for:

- New side-effecting capability.
- New secret access.
- New model provider.
- New sandbox permission.
- New admin permission.
- New tenant isolation boundary.
- New egress destination.

---

## 26. Suggested Implementation Phases

### Phase 1: Foundation

Goal: establish platform language and durable skeleton.

Deliverables:

- Core contracts.
- Domain models.
- Error model.
- Event model.
- Config model.
- Initial database migrations.

Tests:

- Schema tests.
- Contract compile/type tests.
- Migration tests.

Done when: core models are stable enough for runtime implementation.

Risks: over-modeling before first vertical slice.

### Phase 2: Core Runtime

Goal: execute a simple governed run.

Deliverables:

- Run service.
- Scheduler.
- Worker runtime.
- State store.
- Event outbox.
- Reference monitor.
- Echo model adapter.
- Function agent adapter.

Tests:

- Submit/get/cancel.
- Worker lease.
- Event stream.
- Policy allow/deny.

Done when: first vertical slice runs end to end.

### Phase 3: Adapter System

Goal: support replaceable implementations.

Deliverables:

- Adapter registry.
- Manifest schema.
- Compatibility checks.
- Health checks.
- Contract test harness.
- Initial LLM/tool/auth/storage adapters.

Tests:

- Adapter conformance.
- Bad config rejection.
- Health/failure routing.

### Phase 4: API and Clients

Goal: expose stable platform APIs.

Deliverables:

- REST API.
- SSE/WebSocket streaming.
- SDK.
- CLI.
- Optional TUI/Web shell.

Tests:

- API auth.
- Streaming reconnect.
- SDK compatibility.
- Client e2e.

### Phase 5: Memory/RAG

Goal: add governed context.

Deliverables:

- Memory service.
- Knowledge ingestion.
- Retrieval adapter.
- Artifact service.
- Citation support.

Tests:

- Memory isolation.
- Retrieval access control.
- Deletion propagation.

### Phase 6: Security and Multi-User

Goal: production identity and tenant isolation.

Deliverables:

- OIDC/JWT auth.
- RBAC/ABAC.
- Tenant admin.
- Secret provider.
- Sandbox and egress.
- Approval service.

Tests:

- Multi-user.
- Multi-tenant.
- Prompt injection.
- Sandbox.
- Secret redaction.

### Phase 7: Observability and Evaluation

Goal: release with evidence.

Deliverables:

- Metrics.
- Traces.
- Audit.
- Cost ledger.
- Evaluation runner.
- Golden trace suites.

Tests:

- Trace completeness.
- Audit access.
- Eval gates.

### Phase 8: Production Deployment

Goal: reliable Docker/Compose/Kubernetes-ready operation.

Deliverables:

- Dockerfiles.
- Compose.
- K8s manifests.
- Migrations.
- Health/readiness.
- Backup/restore.
- Graceful shutdown.

Tests:

- Deployment smoke.
- Migration.
- Restore.
- Worker crash recovery.
- Load baseline.

### Phase 9: Enterprise Hardening

Goal: high-assurance multi-tenant operations.

Deliverables:

- Advanced tenant isolation.
- Customer-managed keys.
- SSO/SCIM.
- HA queues/stores.
- Advanced policy.
- Incident tooling.
- Compliance evidence.

Tests:

- Disaster recovery.
- Noisy-neighbor.
- Penetration/security.
- Enterprise audit export.

---

## 27. Trade-Offs and Design Decisions

| Trade-off | Recommendation | Rationale |
|---|---|---|
| Framework-agnostic core vs direct framework integration | Framework-agnostic core. | Prevents lock-in and supports many organizations. |
| Monolith vs modular monolith vs microservices | Start modular monolith with API/worker split. | Easier correctness and transactions; extraction remains possible. |
| Local-first vs production-first | Production-shaped with local mode as subset. | Avoids local shortcuts leaking into production. |
| Plugin flexibility vs security | Flexibility through manifests, contracts, sandbox, policy. | Keeps extension without unsafe execution. |
| Generic contracts vs framework-specific power | Generic core plus namespaced extension fields. | Preserves portability while allowing advanced frameworks. |
| Sync vs async execution | Support both; async is default for non-trivial work. | Durable recovery and long-running workflows need async. |
| REST vs WebSocket vs SSE vs gRPC | REST for commands/queries, SSE for streams, WebSocket for interactive bidirectional, gRPC optional internal. | Matches use cases without forcing one protocol. |
| SQL vs document DB vs event store | SQL authoritative store plus event/outbox. | Strong consistency and queryability; eventing without full event-sourcing complexity. |
| Qdrant vs FAISS vs pgvector | Adapter-based; pgvector/local for simple, Qdrant/Weaviate for scale. | Avoids lock-in. |
| Internal workflow engine vs external workflow engine | Lightweight internal runtime plus external adapters. | Good baseline; Temporal/Prefect for advanced durability. |
| Strong contracts vs adapter flexibility | Strong required fields plus namespaced extension fields. | Prevents vague integrations while preserving adapter power. |
| Multi-tenant isolation vs operational complexity | Build tenant context from day one. | Retrofitting tenancy is risky and expensive. |

---

## 28. Anti-Patterns to Avoid

- God orchestrator. **Instead**: Distribute responsibilities across focused runtime components.
- God agent. **Instead**: Create specialized agents coordinated by a workflow or supervisor.
- UI calling tools directly. **Instead**: UI should submit runs; backend platform securely invokes tools.
- CLI/TUI/Web duplicating orchestration logic. **Instead**: Keep orchestration purely in the backend API layer.
- Framework-specific logic in core. **Instead**: Keep core framework-agnostic; isolate specific frameworks in adapters.
- Hardcoded LLM providers. **Instead**: Use LLMGateway and model routing configuration.
- Hardcoded tool schemas. **Instead**: Discover and validate tool schemas dynamically at runtime.
- Auth mixed into business logic. **Instead**: Centralize authorization in the ReferenceMonitor.
- Memory tightly coupled to one vector DB. **Instead**: Use a generic MemoryStore contract.
- No tests for adapters. **Instead**: Write contract conformance tests for all adapters.
- No session isolation. **Instead**: Filter all session and memory access by tenant and user ID.
- No tenant isolation. **Instead**: Make tenant_id a required field in all domain models and queries.
- No audit trail. **Instead**: Emit and store immutable audit events for all protected actions.
- Treating file existence as feature completeness. **Instead**: Test behavior end-to-end to verify contracts.
- Local-only assumptions inside production code. **Instead**: Treat local mode as a constrained instance of the production topology.
- Provider-specific schemas leaking into platform schemas. **Instead**: Map external schemas to canonical Platform envelopes.
- Silent failures. **Instead**: Emit structured error events and fail fast.
- Unbounded tool permissions. **Instead**: Apply least-privilege scopes to all tool adapters.
- Unversioned prompts. **Instead**: Treat prompts as code and version them in registries.
- Unversioned adapters. **Instead**: Include semantic versioning in adapter manifests.
- Unobservable execution. **Instead**: Plumb telemetry contexts through all driver calls.
- Raw secrets in prompts or logs. **Instead**: Use a SecretProvider and redact logs.
- Approval treated as sandboxing. **Instead**: Combine approval with actual process or data sandboxing.
- LLM retries treated as deterministic. **Instead**: Expect varied outputs and validate against schemas on retry.
- Streaming treated as authoritative state. **Instead**: Treat streams as ephemeral and EventBus as authoritative.
- In-memory production run state. **Instead**: Persist run states durably in RunService before dispatching.
- Cross-tenant filtering only in the UI. **Instead**: Enforce tenant filtering in database queries and API boundaries.

---

## 29. Final Recommended Architecture

### 29.1 Architecture Style

Use a modular monolith with separated API and worker deployables, strict internal boundaries, and adapter-based external integrations.

Recommended deployables:

- API service.
- Worker service.
- Web client.
- Evaluation worker.
- Sandbox executor service for high-risk tools.

### 29.2 Final Folder Structure

Use the structure and folder rules defined in Section 11 as the canonical reference. Do not duplicate or deviate from that structure without updating Section 11.

### 29.3 Key Interfaces

Required first interfaces:

- `AgentRuntime`
- `AgentAdapter`
- `WorkflowRuntime`
- `WorkflowAdapter`
- `ToolExecutor`
- `ToolAdapter`
- `SkillExecutor`
- `SkillAdapter`
- `LLMGateway`
- `LLMAdapter`
- `MemoryStore`
- `Retriever`
- `AuthProvider`
- `AuthorizationPolicy`
- `SecretProvider`
- `EventBus`
- `AuditSink`
- `EvaluationRunner`
- `ConfigProvider`
- `TenantResolver`

### 29.4 Key Services

- `RunService`
- `SchedulerService`
- `WorkerRuntime`
- `ReferenceMonitor`
- `AgentRuntimeService`
- `WorkflowRuntimeService`
- `ToolRuntimeService`
- `SkillRuntimeService`
- `LLMGatewayService`
- `MemoryService`
- `KnowledgeService`
- `ArtifactService`
- `ApprovalService`
- `TenantService`
- `AdminService`
- `AuditService`
- `UsageService`
- `EvaluationService`
- `ReleaseService`

### 29.5 Required Infrastructure

| Component | Default recommendation |
|---|---|
| Database | PostgreSQL for production; SQLite/Postgres for local. |
| Queue | Redis Streams, RabbitMQ, NATS, Kafka, or DB-backed queue initially. |
| Cache | Redis in production; in-memory local. |
| Object storage | S3/GCS/Azure Blob/MinIO; local filesystem for dev. |
| Vector store | pgvector for simple, Qdrant/Weaviate for scale. |
| Secret provider | Cloud secret manager, Vault, or Kubernetes secrets with rotation. |
| Observability | OpenTelemetry, Prometheus, Grafana, logs backend. |
| Sandbox | Container first, stronger isolation for untrusted code. |

### 29.6 Required Test Strategy

Minimum release gate:

- Unit tests pass.
- Contract tests pass.
- Adapter tests pass.
- API tests pass.
- Security and tenant isolation tests pass.
- Streaming and failure recovery tests pass.
- Golden trace tests pass.
- Required agent/prompt/tool/skill evals pass.
- Docker/deployment smoke tests pass.

### 29.7 First Implementation Priorities

1. Core contracts, domain models, errors, events.
2. Persistent run service, state store, event outbox.
3. Auth context, tenant resolver, default-deny reference monitor.
4. Scheduler and worker leases.
5. Echo LLM adapter and function agent adapter.
6. Tool runtime with one safe read tool.
7. REST run API and SSE stream.
8. Audit and metrics.
9. Approval service.
10. Artifact service.
11. Memory/RAG.
12. Real model/tool/framework adapters.

### 29.8 Intentionally Deferred

| Deferred item | Reason |
|---|---|
| Full marketplace | Requires supply-chain and governance maturity. |
| Multi-region active/active | Operationally complex; defer until scale requires. |
| Arbitrary untrusted code execution | Requires strong sandbox and security review. |
| Advanced external workflow engines | Add after core durable runtime is proven. |
| Rich desktop client | Backend/API stability comes first. |
| Fully automated memory promotion | Risk of memory poisoning; start with governed writes. |
| Complex ML prompt-injection detector | Structural controls are more important initially. |

### 29.9 Final Recommendation

Build the platform as a production-shaped, API-first, modular monolith with a durable worker runtime, non-bypassable reference monitor, versioned resources, and adapter-based integrations. Keep local mode convenient, but never let local assumptions define production architecture. Treat agents, prompts, tools, skills, workflows, models, policies, and adapters as versioned release artifacts with tests and audit evidence.

This architecture is concrete enough for implementation and flexible enough for organizations with different agentic frameworks, model providers, workflow engines, security systems, and deployment environments.

---

## 30. LLM-Friendly Implementation Playbook

This section exists so an LLM or implementation team can build from the specification without guessing architecture, shortcuts, or ordering.

### 30.1 How an Implementing LLM Must Read This Specification

An implementation LLM must follow these priorities in order:

1. Security, tenant isolation, and reference monitor rules override convenience.
2. Core contracts and domain schemas override framework-specific behavior.
3. Durable state and event records override in-memory convenience.
4. Explicit typed schemas override raw unvalidated dictionaries.
5. Adapter capability declarations override assumptions about provider/framework behavior.
6. Tests and acceptance criteria are part of the feature, not follow-up work.
7. Local mode shortcuts must be labeled P0-only and rejected in production mode.

If any requirement appears ambiguous, choose the safer interpretation:

- Deny rather than allow.
- Persist rather than keep only in memory.
- Validate rather than trust.
- Version rather than mutate in place.
- Emit an event/audit record rather than silently perform work.
- Add a narrow interface rather than widening an existing god interface.

### 30.2 Non-Negotiable Implementation Invariants

The implementation is invalid if any of these are false:

1. A UI client cannot invoke a tool, skill, model, memory store, or retriever directly.
2. A framework adapter cannot bypass the platform tool executor.
3. A model-proposed action is never treated as authorized by itself.
4. Every protected data access has tenant context and policy decision.
5. Every accepted run is durable before the client receives accepted status.
6. Every side-effecting tool call has effect tracking and idempotency/reconciliation semantics.
7. Every adapter declares capabilities, limitations, and compatible contract versions.
8. Every secret is obtained through a secret provider and redacted from telemetry.
9. Every run can be observed by run ID and trace ID.
10. Every production release has tests and evaluation evidence for changed resources.

### 30.3 LLM Implementation Task Template

Every implementation task given to an LLM should use this template:

```text
Task:
  Implement <module/service/adapter>.

Spec references:
  - Sections:
  - Contracts:
  - Domain models:
  - API endpoints:
  - Tests:

Inputs:
  - Existing files/modules:
  - Config:
  - Required schemas:

Requirements:
  - Functional:
  - Security:
  - Observability:
  - Persistence:
  - Error handling:
  - Compatibility:

Do not:
  - Bypass reference monitor.
  - Import adapters into core.
  - Put runtime logic in UI.
  - Store secrets in config/logs.
  - Add unbounded retries/queues/loops.

Acceptance criteria:
  - Tests:
  - Manual verification:
  - Events/audit emitted:
  - Failure behavior:
```

### 30.4 Module Implementation Template

Every module should be specified before implementation:

```yaml
Module:
  name: RunService
  responsibility: Submit and manage durable runs.
  owns:
    - run state transitions
    - idempotency lookup
    - run snapshot queries
  does_not_own:
    - tool execution
    - model provider calls
    - UI rendering
  contracts_used:
    - StateStore
    - EventBus
    - AuthorizationPolicy
    - Scheduler
  inputs:
    - ExecutionRequest
  outputs:
    - RunRecord
    - RuntimeEvent
  errors:
    - ValidationError
    - AuthorizationError
    - IdempotencyConflictError
  persistence:
    - runs
    - events
    - idempotency_records
  observability:
    - run.submitted
    - run.queued
  tests:
    - submit_valid_run
    - duplicate_idempotency_key_returns_same_run
    - cross_tenant_read_denied
```

### 30.5 Required Build Order for LLM Implementation

Build in this order to avoid architecture drift:

1. Domain models, error taxonomy, and event schemas.
2. Database migrations for tenants, principals, runs, steps, attempts, events, audit, idempotency.
3. Config provider and production-mode validation.
4. Auth provider and tenant resolver.
5. Reference monitor with default-deny policy.
6. Run service with durable accepted state.
7. Event bus/outbox.
8. Scheduler with leases, heartbeats, and dead-letter.
9. Worker runtime with graceful shutdown.
10. LLM gateway with echo adapter.
11. Function agent adapter.
12. Tool registry and tool executor with one pure/read tool.
13. SSE streaming with cursor resume.
14. Audit sink and metrics.
15. Approval service.
16. Artifact service.
17. Memory service.
18. Knowledge ingestion and retrieval.
19. Real provider/framework adapters.
20. Sandbox executor for side-effecting tools.
21. Evaluation runner and release manifest.
22. Docker/Compose/Kubernetes deployment.
23. Failure-injection, load, and security hardening.

### 30.6 Generated Code Rejection Rules

Generated code must be rejected if it:

- Introduces direct provider SDK calls outside adapters.
- Adds business logic to API route handlers instead of application/runtime services.
- Uses globals for tenant, user, run, config, or current model state.
- Catches broad exceptions without structured logging and typed error conversion.
- Ignores cancellation, deadlines, or worker lease fencing.
- Adds a tool or skill without schema, permission, test, and audit behavior.
- Stores run state only in memory.
- Uses prompt text to enforce security policy.
- Logs raw prompts, secrets, credentials, or sensitive artifacts by default.
- Has no tests for failure paths.

### 30.7 Ambiguity Removal Rules

When a design choice is unclear:

| Ambiguity | Required default |
|---|---|
| Is this action protected? | Treat it as protected. |
| Is this data tenant-owned? | Treat it as tenant-owned. |
| Is this retry safe? | Treat it as unsafe until idempotency/effect verification exists. |
| Can this adapter stream/cancel? | Treat as unsupported until declared and tested. |
| Can this memory be trusted? | Treat as untrusted context with provenance. |
| Should this go in core or adapter? | Put provider/framework-specific code in adapter. |
| Should UI implement this behavior? | Put behavior in backend/API; UI renders and commands only. |
| Should this be sync or async? | Use async/durable for long-running or side-effecting work. |

---

## 31. Production Acceptance Checklist and Definitions

### 31.1 Definition of Ready

A feature is ready to implement only when:

- Purpose and user/system outcome are clear.
- Owner is identified.
- Relevant contracts and domain models are identified.
- Security and tenant impact are known.
- Persistence and migration impact are known.
- Observability and audit events are listed.
- Failure behavior is defined.
- Tests and acceptance criteria are listed.
- Rollback or disablement path is known.

### 31.2 Definition of Done for Any Module

A module is done only when:

- It has a single clear responsibility.
- Public contracts are typed and documented.
- Inputs and outputs are validated.
- Errors are mapped to platform error taxonomy.
- Security checks happen at the correct boundary.
- Tenant/user/run/trace context is propagated.
- State changes are durable where required.
- Events and audit records are emitted where required.
- Unit tests and relevant integration tests pass.
- Negative tests cover denied access and invalid input.
- Observability is sufficient for production debugging.
- Documentation explains how to use and extend it.

### 31.3 Definition of Done for an Adapter

An adapter is done only when:

- Manifest exists and validates.
- Config schema exists and validates.
- Capabilities and unsupported features are declared.
- Compatible platform and contract versions are declared.
- Health check exists.
- Contract tests pass.
- External errors map to platform errors.
- Timeout, cancellation, retry, and rate-limit behavior are tested.
- Credentials are scoped and redacted.
- Security review is complete for side-effecting behavior.

### 31.4 Definition of Done for an API Endpoint

An API endpoint is done only when:

- Request and response schemas are versioned.
- AuthN and AuthZ are tested.
- Tenant isolation is tested.
- Idempotency behavior is defined for mutating operations.
- Pagination/filtering exists for list endpoints.
- Error responses use the platform error model.
- OpenAPI or equivalent documentation is generated.
- API tests include success, validation failure, auth failure, and internal error redaction.

### 31.5 Definition of Done for Runtime Execution

Runtime execution is done only when:

- Run/step/attempt state transitions are explicit and tested.
- Durable accepted state exists before accepted response.
- Worker lease, heartbeat, and fencing semantics are implemented.
- Cancellation and deadlines propagate.
- Retry policy distinguishes safe, unsafe, and uncertain effects.
- Streaming events are resumable.
- Final state is terminal and queryable.
- Golden trace tests pass.

### 31.6 Definition of Done for Production Release

A production release is done only when:

- Release manifest pins all changed resource revisions.
- Contract, adapter, API, security, and integration tests pass.
- Required prompt/agent/tool/skill/workflow evals pass.
- Migration tests pass.
- Rollback/disable path is documented.
- Observability dashboards and alerts exist.
- Runbooks exist for new failure modes.
- Security review is complete for changed protected behavior.
- Deployment smoke test passes.

### 31.7 Final Completeness Checklist

Before declaring the platform implementation complete, verify:

- No core imports framework/provider packages.
- No UI owns runtime orchestration.
- No protected action bypasses reference monitor.
- No tenant isolation is enforced only in UI.
- No production run state is memory-only.
- No tool/skill lacks schema, permission, version, and tests.
- No adapter lacks manifest, capability declaration, and conformance tests.
- No prompt, agent, workflow, policy, or model route is unversioned.
- No secret is logged or model-visible by default.
- No LLM fallback violates data/model policy.
- No accepted run is unobservable.
- No release can bypass required eval and security gates.


---

## Appendix A: End-to-End Request Walkthrough

This section traces a single user request through every major component, showing the concrete data that flows between them. Use this as a reference implementation guide.

Scenario: User sends "What time is it?" to an agent that has access to a `get_current_time` tool.

### Step 1: API receives request

Client sends:

```json
{
  "method": "POST",
  "path": "/api/v1/runs",
  "headers": {"Authorization": "Bearer <token>", "X-Idempotency-Key": "abc-123"},
  "body": {
    "session_id": "sess-001",
    "input": "What time is it?",
    "agent_ref": {"kind": "agent", "id": "default-agent"}
  }
}
```

### Step 2: Authentication

`AuthProvider.authenticate(token)` returns:

```json
{
  "principal_type": "user",
  "principal_id": "tenant-1:user-42",
  "tenant_id": "tenant-1",
  "roles": ["user"],
  "scopes": ["runs.create", "tools.invoke"]
}
```

### Step 3: RunService creates run

Idempotency check: no match for `(tenant-1, abc-123)`. Creates:

```json
{
  "run_id": "run-7001",
  "status": "submitted",
  "target": {"kind": "agent", "id": "default-agent"},
  "tenant_id": "tenant-1",
  "user_id": "user-42",
  "session_id": "sess-001",
  "created_at": "2025-01-15T10:00:00Z"
}
```

Emits: `run.submitted` event. Returns `202 Accepted` with `RunRecord`.

### Step 4: Scheduler enqueues step

Scheduler creates and enqueues:

```json
{
  "step_id": "step-1",
  "run_id": "run-7001",
  "type": "agent",
  "target_ref": {"kind": "agent", "id": "default-agent"},
  "status": "pending",
  "sequence": 1
}
```

Run transitions: `submitted` -> `queued`. Emits: `run.queued`.

### Step 5: Worker leases step

`Scheduler.lease(worker_id="w-1", queues=["default"], lease_seconds=60)` returns the step. Step transitions: `pending` -> `leased` -> `running`. Emits: `step.started`.

Run transitions: `queued` -> `running`. Emits: `run.started`.

### Step 6: ReferenceMonitor authorizes

`ReferenceMonitor.authorize_action(identity, "agent.execute", agent_ref, context)` returns:

```json
{"allowed": true, "decision_id": "dec-001"}
```

### Step 7: AgentRuntime invokes LLM

`LLMGateway.complete(request, context)` sends the user message plus available tool schemas to the model. Model responds requesting a tool call:

```json
{
  "content": null,
  "tool_calls": [{"id": "tc-1", "name": "get_current_time", "arguments": {}}],
  "usage": {"prompt_tokens": 150, "completion_tokens": 20, "total_tokens": 170}
}
```

Emits: `llm.request_started`, `llm.request_completed`.

### Step 8: ReferenceMonitor authorizes tool call

`ReferenceMonitor.authorize_action(identity, "tool.invoke", tool_ref, context)` checks:
- Tool `get_current_time` has `side_effect_class: read` â no approval required.
- User has scope `tools.invoke` â allowed.

Returns: `{"allowed": true}`.

### Step 9: ToolExecutor runs tool

`ToolExecutor.execute(tool_ref, {}, context)` returns:

```json
{
  "output": {"current_time": "2025-01-15T10:00:05Z", "timezone": "UTC"},
  "status": "success",
  "effects": [],
  "usage": {"duration_ms": 12}
}
```

Emits: `tool.started`, `tool.completed`.

### Step 10: AgentRuntime sends tool result to LLM

Second LLM call with tool result. Model responds:

```json
{
  "content": "The current time is 10:00:05 AM UTC.",
  "tool_calls": [],
  "usage": {"prompt_tokens": 200, "completion_tokens": 15, "total_tokens": 215}
}
```

### Step 11: Run completes

Step transitions: `running` -> `completed`. Emits: `step.completed`.
Run transitions: `running` -> `completed`. Emits: `run.completed`.

Final `RunRecord`:

```json
{
  "run_id": "run-7001",
  "status": "completed",
  "output": "The current time is 10:00:05 AM UTC.",
  "usage": {"prompt_tokens": 350, "completion_tokens": 35, "total_tokens": 385}
}
```

### Step 12: Client receives result

If streaming: Client received `stream.chunk` events during LLM generation, then `stream.terminal` with final status.

If polling: Client calls `GET /api/v1/runs/run-7001` and receives the completed `RunRecord`.

If using idempotency: A duplicate `POST /api/v1/runs` with `X-Idempotency-Key: abc-123` returns the stored `RunRecord` without re-execution.
