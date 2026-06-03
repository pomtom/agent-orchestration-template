---
name: readme-generator
description: Analyze an entire .NET solution and generate a comprehensive README.md — architecture, solution structure, dependencies, design patterns, configuration, deployment, logging, monitoring, security, local dev, and troubleshooting. Use when the user asks to create, generate, update, or improve a repository README, document the solution, or onboard a new repo. Works for Web APIs, Azure/Durable Functions, MVC, Blazor, Worker Services, and Console apps.
---

# README Generator

Generate a comprehensive, accurate `README.md` from the actual solution — never from
assumptions.

## Procedure

1. **Detect project types first.** Follow
   [project-type-detection](../../../docs/standards/project-type-detection.md) and emit
   the detection summary.
2. **Analyze the solution:**
   - Enumerate projects, references, and the project dependency graph.
   - Identify APIs/controllers/Minimal APIs, services, functions, Durable
     orchestrators/activities, workers, hosted services, and workflows.
   - Detect architectural patterns (Clean Architecture, layering) and design patterns
     (CQRS, MediatR, Repository, Options, Factory, Decorator, Mediator, etc.) and where
     each appears.
   - Identify dependencies and integrations (databases, message brokers, Azure
     services, external HTTP APIs) from package references and config.
   - Read configuration sources for required settings (without copying secret values).
3. **Classify the architecture.** From the analysis, assign a one-line architecture
   classification (e.g. *event-driven serverless Azure Functions with Service Bus*,
   *layered Clean Architecture Web API with CQRS/MediatR*, *Durable Functions
   fan-out/fan-in*, *background worker pipeline*). This label drives the diagram choice.
4. **Build the diagrams** per the **Architecture diagram guidelines** in
   [readme-standard](../../../docs/standards/readme-standard.md):
   - A **system architecture diagram** visually encoding the architecture *type*
     (subgraphs for boundaries/layers, `classDef` styling, role-based shapes, labeled
     edges, a legend). Pick the Mermaid type that fits — `architecture-beta`/`C4` for
     cloud/container views, layered `flowchart TD` for Clean Architecture, `flowchart LR`
     for serverless/event flows.
   - A **flow/sequence diagram** tracing a representative request or message end-to-end.
   - For Functions, a **trigger/endpoint table** (trigger type, inputs, outputs/bindings).
   - **Validate & render via a connector when available**: call the Mermaid Chart
     connector (`validate_and_render_mermaid_diagram`) to confirm syntax and preview;
     optionally use the draw.io connector (`create_diagram`, Mermaid or XML) for an
     editable/interactive visual. Embed the **validated** Mermaid source in fenced
     ```mermaid blocks so GitHub renders it natively. If no connector is available,
     still produce syntactically valid, self-checked Mermaid.
5. **Generate the README** with every section and rule defined in
   [readme-standard](../../../docs/standards/readme-standard.md), placing the
   classification, system diagram, flow diagram, and trigger table in the
   **Architecture & Flow** section.
6. **Write output** to `README.md` at the repo root. If one exists, present the proposed
   content plus a summary of changes for review instead of silently overwriting.

## Guardrails

- Do **not** create projects or business code; this skill only produces documentation.
- Never embed secrets, real connection strings, or credentials — use placeholders.
- Cross-reference the configuration, logging, and security standards for those sections.
