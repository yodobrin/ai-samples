# Migration Strategy: Python to .NET for the AdaptIQ Agent

This document outlines a high-level approach to migrate the AdaptIQ Agent codebase from Python to .NET. It focuses on concepts, planning artifacts (epics, user stories, features), and guidance—rather than providing full .NET code. It also evaluates SDK options (low-level vs. frameworks like Semantic Kernel).

---

## 1. Goals and Scope

- Maintain existing functionality: multi-agent orchestration, research iterations, and report synthesis.
- Leverage .NET ecosystem for performance, type safety, and enterprise support.
- Evaluate trade-offs between using Azure OpenAI low-level SDK and higher-level frameworks.

## 2. Migration Epics and User Stories

### Epic 1: Core Orchestration Infrastructure
- **User Story 1.1**: As a developer, I want to port the `ResearchOrchestrator` logic into .NET so that I can orchestrate agent workflows using C#.
- **User Story 1.2**: As a tech lead, I need the async processing pipeline and tool-call machinery re-implemented in .NET Task-based model.

### Epic 2: Agent Abstractions and Tools
- **User Story 2.1**: As a developer, I want to define a base `Agent` interface in .NET with pluggable tool methods.
- **User Story 2.2**: As an AI engineer, I want each specialized agent (SupportCaseSearch, APRLRecommendation, MSLearnRecommendation, WebScraper) implemented as a .NET class, exposing SDK or HTTP clients.

### Epic 3: Integration with Azure OpenAI
- **User Story 3.1**: As a developer, I want to call Azure OpenAI completions through the official .NET SDK so that I can replace Python client usage.
- **User Story 3.2**: As a researcher, I want the ability to swap between low-level `OpenAIClient` and higher-level frameworks (Semantic Kernel) with minimal code changes.

### Epic 4: Notebook and CLI Workflows
- **User Story 4.1**: As a user, I want interactive notebooks replaced or complemented by .NET interactive notebooks (C# in Jupyter/VS Code) or CLI tools.

### Features Breakdown
- **Feature 1**: BaseAgent abstraction in C# with tool registration and invocation.
- **Feature 2**: Research iteration loop utilizing `async/await` and cancellation tokens.
- **Feature 3**: Report planning and writing components, generating Markdown output.
- **Feature 4**: Clients for external services (ZebraAI, APRL JSON, MS Learn) using HttpClient and typed models.
- **Feature 5**: Configuration management via `IConfiguration` and `Options` pattern.

---

## 3. Key .NET Concepts and Patterns

| Python Pattern               | .NET Equivalent                            |
|------------------------------|--------------------------------------------|
| `async def` + `await`        | `async Task` + `await`                     |
| Pydantic models              | C# record or `class` with System.Text.Json |
| `@tool` decorator           | Attribute-based metadata + Reflection      |
| OpenAI Python client         | Azure.AI.OpenAI (`OpenAIClient`)           |
| Notebook entrypoint (IPython)| .NET Interactive, LINQPad, or CLI app      |

### 3.1 Agent and Tool Registration
- Create an `IAgent` interface with a `ProcessQueryAsync` method.
- Use custom attributes (`[Tool(FunctionName)]`) to mark tool methods.
- Reflect over agent classes at startup to build a tool registry.

### 3.2 Orchestration Loop
- Implement a `ResearchOrchestrator` class that:
  - Tracks iterations using a `Stopwatch` and `int` counters.
  - Calls `GenerateThoughtsAsync`, `EvaluateGapsAsync`, `SelectAgentsAsync`, `RunAgentTasksAsync`.
  - Uses `Parallel.ForEachAsync` or `Task.WhenAll` for concurrent agent tasks.

### 3.3 Data Models and Serialization
- Define C# models for:
  - `ReportPlan`, `ReportDraft`, `ResearchIteration`, etc.
- Use `System.Text.Json` or `Newtonsoft.Json` for JSON handling.

### 3.4 Dependency Injection
- Leverage `Microsoft.Extensions.DependencyInjection` to register agents, HTTP clients, and `OpenAIClient`.
- Scoping: `Singleton` for clients, `Transient` for per-operation services.

---

## 4. Azure OpenAI SDK vs. Semantic Kernel

### 4.1 Low-Level SDK (`Azure.AI.OpenAI`)
- **Pros**: 
  - Fine-grained control over request/response.
  - Fewer hidden abstractions; easy to map Python code one-to-one.
- **Cons**: 
  - You must implement pipeline logic (tool calls, retries).
  - Less built-in templating or memory management.

### 4.2 Semantic Kernel
- **Pros**:
  - Built-in skill and prompt management.
  - Supports planner, memory, chaining out-of-the-box.
- **Cons**:
  - Opinionated binding and life cycle.
  - Rapidly evolving API; potential breaking changes.

#### Recommendation
- Start with the low-level SDK for core orchestration and tool invocation. 
- Evaluate adding Semantic Kernel for higher-level scenarios (e.g., memory or chaining), wrapping it behind an abstraction interface so it can be swapped out.

---

## 5. Example: Translating a Python Async Call

**Python**:
```python
thoughts = await orchestrator._async_generate_thoughts(
    iteration_idx=1,
    query="Topic",
    previous_observations=None,
    background_context="Context"
)
```

**C# (.NET)**:
```csharp
var thoughts = await orchestrator.GenerateThoughtsAsync(
    iterationIndex: 1,
    query: "Topic",
    previousObservations: null,
    backgroundContext: "Context",
    cancellationToken: ct
);
```
- Use `Task<string>` instead of `Coroutine`.
- Pass `CancellationToken` for time-limit enforcement.

---

## 6. Migration Approach and Phases

1. **Proof of Concept (PoC)**
   - Port `BaseAgent` and one agent (e.g. SupportCaseSearchAgent) to .NET.
   - Wire up Azure OpenAI low-level calls.
2. **Core Orchestrator**
   - Implement `ResearchOrchestrator` and iteration loop.
3. **Complete Agent Suite**
   - Port APRL, MSLearn, WebScraper agents.
4. **Testing and Validation**
   - Unit tests with xUnit and mocks for HTTP and OpenAI clients.
5. **Documentation and Samples**
   - Provide C# interactive notebook examples.
6. **Framework Evaluation**
   - Integrate Semantic Kernel behind abstraction and compare capabilities.

---

## 7. Conclusion

Migrating to .NET involves translating async patterns, data models, and orchestration logic into C# idioms. By organizing the work into epics, user stories, and features, you can plan incremental development. Start with the low-level Azure OpenAI SDK for stability, then optionally adopt Semantic Kernel for advanced orchestration. The proposed approach provides structure while maintaining flexibility to swap frameworks as they evolve.

