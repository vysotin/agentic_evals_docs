# Section 9: Full Trace Observability

> **Part V**: Observability & Tracing
> **Estimated Reading Time**: 15 minutes

## Overview

Full trace observability represents the foundational shift from traditional system monitoring to behavioral telemetry for AI agents. Unlike conventional applications where tracking CPU usage and error rates suffices, AI agents fail silently—producing plausible but incorrect outputs while all system metrics appear healthy. This section explores the architecture of agent traces, the emerging OpenTelemetry standards for GenAI observability, instrumentation approaches, and practical implementation patterns that enable teams to understand, debug, and improve agentic systems at scale.

## Table of Contents

- [9.1 From Traditional Monitoring to Agent Observability](#91-from-traditional-monitoring-to-agent-observability)
- [9.2 The Architecture of an Agent Trace](#92-the-architecture-of-an-agent-trace)
- [9.3 OpenTelemetry and OpenLLMetry Standards](#93-opentelemetry-and-openllmetry-standards)
- [9.4 Trace Collection and Instrumentation](#94-trace-collection-and-instrumentation)
- [Key Takeaways](#key-takeaways)

---

## 9.1 From Traditional Monitoring to Agent Observability

Traditional software monitoring focuses on infrastructure health metrics: CPU utilization, memory consumption, request latency, and error rates. These metrics work well for deterministic systems where failures manifest as exceptions, timeouts, or resource exhaustion. But they catastrophically fail for AI agents.

### The Silent Failure Problem

An AI agent can fail completely while every traditional metric remains green [1, 2]. Consider a customer service agent that:

- Receives a query about order cancellation
- Successfully calls the database API (✓ no errors)
- Returns a response within 200ms (✓ sub-second latency)
- Uses 2,000 tokens (✓ within budget)
- Generates perfectly formatted JSON output (✓ valid schema)

Yet the agent **failed** because it misunderstood the query, retrieved the wrong order, and provided instructions for the incorrect product. Traditional monitoring captures none of this. The failure is *logical*, not *technical*—the agent did something wrong, not something that broke.

This is the fundamental challenge Nitika Bhatia identifies in her framework: agents produce *incorrect outputs* that look correct to traditional observability systems [3]. You need to observe what the agent is *thinking* and *deciding*, not just what systems it's calling and how fast they respond.

### Behavioral Telemetry: Observing Agent Cognition

Industry leaders describe the shift as moving from "system telemetry" to "behavioral telemetry" [4]. You must instrument the agent's internal decision-making process:

- **Reasoning steps**: What thoughts led to each action?
- **Tool selection**: Why did the agent choose this particular tool?
- **Context utilization**: What information did it use from memory or retrieval?
- **Goal tracking**: Is the agent still pursuing the original objective, or has it drifted?
- **Verification**: Did the agent validate its own outputs before returning them?

As one industry report notes: "Observability in 2026 becomes behavioral telemetry, including step-level traces (reason → tool call → result → decision) and outcome evaluation" [5]. This represents a paradigm shift as profound as the move from manual testing to automated testing in traditional software.

### The Adoption Reality in 2026

The industry has embraced this shift rapidly. According to the 2026 State of AI Agents report, **89% of organizations have implemented some form of observability for their agents**, and **62% have detailed tracing that allows them to inspect individual agent steps and tool calls** [6]. This near-universal adoption reflects a hard-learned lesson: without observability, you cannot debug, improve, or trust production agents.

Yet adoption doesn't equal mastery. Many organizations implement basic tracing but lack the systematic frameworks to extract insights from trace data. The challenge has moved from "should we trace?" to "how do we make traces actionable?"

---

## 9.2 The Architecture of an Agent Trace

An agent trace is a hierarchical, structured record of everything that happened during an agent's execution. Unlike flat log files, traces capture the *relationships* between events—which tool calls resulted from which reasoning steps, how information flowed between components, and where the agent's execution branched or backtracked.

### Core Components of a Trace

Every agent trace contains these fundamental elements [7, 8]:

#### 1. Root Span: The User Request

The root span represents the entire agent session from initial trigger to final response. It captures:

- **Input**: The user's query, request, or triggering event
- **Session context**: User ID, conversation history, relevant metadata
- **Start timestamp**: When the agent began processing
- **End timestamp**: When the final response was returned
- **Status**: Success, failure, or partial completion

Example structure:

```json
{
  "trace_id": "trace-abc-123",
  "root_span": {
    "span_id": "span-root",
    "name": "agent.run",
    "start_time": "2026-01-13T10:15:00Z",
    "end_time": "2026-01-13T10:15:03.2Z",
    "attributes": {
      "user_input": "Cancel my order for product SKU-12345",
      "user_id": "user-456",
      "session_id": "session-789",
      "agent_type": "customer_service_agent"
    },
    "status": "SUCCESS"
  }
}
```

#### 2. Reasoning Spans: Agent Thoughts

These spans capture the agent's chain-of-thought reasoning—the internal deliberation that occurs between observations and actions. In agentic architectures, these are often the most valuable spans for understanding agent behavior.

```json
{
  "span_id": "span-thought-1",
  "parent_id": "span-root",
  "name": "agent.reasoning",
  "attributes": {
    "thought": "The user wants to cancel an order. I need to verify the order exists and is cancellable before proceeding. I should call the getOrderStatus tool first.",
    "reasoning_type": "planning"
  }
}
```

Reasoning spans answer the critical question: *Why did the agent decide to take this action?* Without these spans, traces show what happened but not why, making root cause analysis nearly impossible [9].

#### 3. Tool Call Spans: External Interactions

Tool spans record every interaction with external systems—API calls, database queries, web searches, calculator invocations, or any other tool the agent can use.

```json
{
  "span_id": "span-tool-1",
  "parent_id": "span-root",
  "name": "tool.getOrderStatus",
  "attributes": {
    "tool_name": "getOrderStatus",
    "tool_input": {
      "order_id": "ORD-789",
      "user_id": "user-456"
    },
    "tool_output": {
      "status": "SHIPPED",
      "cancellable": false,
      "shipped_date": "2026-01-10"
    },
    "latency_ms": 145,
    "cost_usd": 0.0001
  }
}
```

Tool spans are the bridge between the agent's intentions (reasoning spans) and the real world. They're critical for identifying:

- **Tool selection errors**: Did the agent call the right tool?
- **Parameter errors**: Were the inputs correct?
- **Performance bottlenecks**: Which tools are slow?
- **Cost optimization**: Where is token/API spending concentrated?

#### 4. Memory Spans: Context Management

Modern agents maintain memory systems—vector stores, conversation history, persistent knowledge bases. Memory spans track reads and writes to these systems.

```json
{
  "span_id": "span-memory-1",
  "parent_id": "span-root",
  "name": "memory.retrieve",
  "attributes": {
    "memory_type": "vector_store",
    "query": "user preferences for order cancellations",
    "results_count": 3,
    "relevance_scores": [0.89, 0.76, 0.65],
    "retrieved_content": ["User prefers email confirmation...", ...]
  }
}
```

Memory spans reveal whether the agent is effectively using context. Common failure modes they expose include:

- **Forgetting**: Failing to retrieve relevant prior context
- **Hallucination**: Inventing information instead of checking memory
- **Context overload**: Retrieving too much irrelevant information

#### 5. LLM Call Spans: Model Invocations

Every call to a language model gets its own span, capturing the full prompt, response, token usage, and model parameters.

```json
{
  "span_id": "span-llm-1",
  "parent_id": "span-thought-1",
  "name": "llm.generate",
  "attributes": {
    "model": "claude-sonnet-4-5",
    "prompt_tokens": 450,
    "completion_tokens": 85,
    "total_tokens": 535,
    "temperature": 0.7,
    "max_tokens": 1000,
    "prompt": "Based on the order status...",
    "response": "I need to inform the user that the order has already shipped...",
    "latency_ms": 892,
    "cost_usd": 0.0027
  }
}
```

These spans are essential for:

- **Cost tracking**: Identifying expensive prompts
- **Performance optimization**: Finding slow model calls
- **Quality analysis**: Correlating model outputs with success/failure
- **Debugging**: Understanding what information the model received

### Hierarchical Relationships and Trace Trees

The power of structured traces lies in their hierarchical nature. A complete trace forms a tree:

```
agent.run (root)
├── agent.reasoning (thought 1)
│   └── llm.generate
├── tool.getOrderStatus
├── agent.reasoning (thought 2)
│   └── llm.generate
├── memory.retrieve
├── agent.reasoning (thought 3)
│   └── llm.generate
└── agent.output
```

This structure enables powerful analysis:

- **Path analysis**: What sequence of steps led to success vs. failure?
- **Branching identification**: Where did the agent consider multiple options?
- **Depth metrics**: How many reasoning cycles did the agent use?
- **Parallelism detection**: Did the agent execute steps concurrently?

### Correlation IDs and Distributed Tracing

In multi-agent systems, a single user request may trigger multiple agents working cooperatively. Correlation IDs link all related spans across agent boundaries, creating a unified trace of the entire workflow [10].

This is particularly critical for architectures where:

- A **coordinator agent** delegates sub-tasks to **specialist agents**
- Multiple agents collaborate on solving a complex problem
- Agents invoke other agents as tools

Without correlation IDs, debugging multi-agent coordination failures becomes nearly impossible.

---

## 9.3 OpenTelemetry and OpenLLMetry Standards

As the AI agent ecosystem matured, the industry recognized the need for standardized observability. The result: OpenTelemetry extensions for Generative AI, with OpenLLMetry emerging as the leading implementation [11, 12].

### Why Standards Matter

Before standardization, every observability platform and agent framework used different trace formats. This created vendor lock-in and made it impossible to:

- Switch observability backends without reinstrumenting
- Compare agent performance across frameworks
- Share evaluation tools and methodologies
- Build ecosystem-wide tooling

OpenTelemetry solves this by providing a **vendor-neutral standard** for telemetry data [13]. The same instrumentation can export traces to Langfuse, Arize Phoenix, Datadog, or any OpenTelemetry-compatible backend.

### OpenTelemetry GenAI Semantic Conventions

The OpenTelemetry community established **semantic conventions for generative AI systems**—a standard schema for tracking prompts, model responses, token usage, tool calls, and agent actions [14, 15]. These conventions define:

#### Standard Attribute Names

Instead of each platform inventing its own names for common concepts, the conventions specify:

- `gen_ai.system`: The LLM provider (e.g., "openai", "anthropic")
- `gen_ai.request.model`: The specific model used
- `gen_ai.usage.prompt_tokens`: Tokens in the prompt
- `gen_ai.usage.completion_tokens`: Tokens in the response
- `gen_ai.prompt`: The full prompt text
- `gen_ai.completion`: The full response text

#### Span Types for AI Operations

The conventions define specific span types for AI workloads:

- **`gen_ai.client.request`**: A call to an LLM API
- **`gen_ai.agent.run`**: An agent execution session
- **`gen_ai.tool.call`**: A tool invocation by an agent
- **`gen_ai.retrieval`**: A retrieval or search operation

These standardized span types enable cross-platform tools to interpret traces correctly without custom logic for each framework [16].

### The OpenLLMetry Project

OpenLLMetry, created by Traceloop, extends OpenTelemetry with specific instrumentation for LLM applications and agents [17, 18]. What began as pragmatic gap-filling quickly revealed deeper needs: GenAI workloads introduce concepts OpenTelemetry didn't originally anticipate.

Key innovations include:

#### Session-Level Grouping

Traditional distributed tracing tracks individual requests. But AI agents often span multiple requests in a conversation or workflow. OpenLLMetry introduces **session spans** that group related traces into logical units [18].

```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow

Traceloop.init()

@workflow(name="customer_support_session")
def handle_customer_conversation(user_id, session_id):
    # All agent interactions within this function
    # are grouped into a single session trace
    pass
```

#### Large Payload Handling

LLM prompts and responses can be massive—tens of thousands of tokens. OpenLLMetry provides strategies for handling large payloads without overwhelming trace storage:

- Truncation with configurable limits
- Compression of repetitive content
- Reference-based storage (storing large payloads separately and linking them)

#### Annotation Primitives

OpenLLMetry provides decorators and context managers for marking AI-specific entry points:

```python
from traceloop.sdk.decorators import task, agent

@agent(name="research_agent")
def research_agent(query):
    # Agent execution automatically traced
    pass

@task(name="web_search")
def search_web(query):
    # Tool call automatically traced
    pass
```

These annotations produce properly structured traces without manual span management [19].

### Industry Support and Convergence

By 2026, OpenTelemetry GenAI semantic conventions have achieved remarkable adoption. Contributors include Amazon, Elastic, Google, IBM, Microsoft, Datadog, and leading observability startups [20]. Major platforms now natively support the standards:

- **Datadog LLM Observability** natively supports OpenTelemetry GenAI conventions (v1.37+) [21]
- **Langfuse** operates as an OpenTelemetry backend, accepting traces via OTLP endpoint [22]
- **Arize Phoenix** accepts and visualizes OpenTelemetry-formatted traces [23]
- **Azure AI Foundry** has native integrations with Microsoft Agent Framework using OpenTelemetry [24]

This convergence means you can instrument once and export everywhere—a massive improvement over the fragmented landscape of 2023-2024.

### OTLP: The Protocol

The **OpenTelemetry Protocol (OTLP)** is the wire format for sending trace data. It's a gRPC- or HTTP-based protocol that all OpenTelemetry-compatible systems understand [25].

Instrumented agents send traces to an OTLP endpoint:

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Configure OTLP exporter
otlp_exporter = OTLPSpanExporter(
    endpoint="https://your-observability-platform.com/v1/traces",
    headers={"Authorization": "Bearer YOUR_API_KEY"}
)

# Set up tracer
provider = TracerProvider()
processor = BatchSpanProcessor(otlp_exporter)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
```

The beauty of OTLP: change the endpoint URL, and your traces go to a different platform. No code changes needed.

---

## 9.4 Trace Collection and Instrumentation

With standards defined, the practical question becomes: how do you actually instrument your agents? The industry has converged on three approaches, often used in combination [26, 27].

### Auto-Instrumentation: Zero-Code Observability

Auto-instrumentation libraries automatically trace supported frameworks with minimal setup. This is the fastest path to basic observability.

#### How It Works

You install an auto-instrumentation package, initialize it, and it monkey-patches common libraries to emit traces automatically:

```python
# Auto-instrument OpenAI, LangChain, CrewAI, and more
from traceloop.sdk import Traceloop

Traceloop.init(app_name="my-agent", api_key="your-key")

# Now all LLM calls and agent operations are automatically traced
from openai import OpenAI
client = OpenAI()

# This call is automatically traced with full prompt/response
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

No manual span creation. No explicit context propagation. Everything just works [28].

#### Supported Frameworks

Leading auto-instrumentation libraries support the major agent frameworks:

- **LLM APIs**: OpenAI, Anthropic, Google Gemini, Cohere, AWS Bedrock
- **Agent frameworks**: LangChain, LlamaIndex, CrewAI, AutoGen, Semantic Kernel
- **Vector databases**: Pinecone, Weaviate, Chroma, Qdrant
- **Workflow engines**: LangGraph, Haystack, DSPy

This covers the vast majority of production agent architectures [29, 30].

#### Advantages and Limitations

**Advantages**:
- **Fast to implement**: Minutes to add, not days
- **Low maintenance**: Updates to frameworks are handled by library maintainers
- **Comprehensive**: Captures all instrumented operations automatically

**Limitations**:
- **Black box**: Limited control over what's traced and how
- **Generic**: May not capture domain-specific information you care about
- **Dependency on support**: If your framework isn't supported, you're stuck

Auto-instrumentation is ideal for **getting started quickly** and for **standardized workflows**. It's the recommended first step for any team.

### Manual Instrumentation: Surgical Precision

When auto-instrumentation doesn't capture everything you need, manual instrumentation provides complete control. You explicitly create spans for operations you want to track.

#### Basic Pattern

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

def my_custom_operation(input_data):
    # Create a span for this operation
    with tracer.start_as_current_span("custom.operation") as span:
        # Add attributes to the span
        span.set_attribute("input.size", len(input_data))
        span.set_attribute("operation.type", "data_processing")

        # Do your work
        result = process_data(input_data)

        # Record results
        span.set_attribute("output.size", len(result))

        return result
```

This creates a span named `custom.operation` with custom attributes describing the operation [31].

#### Advanced Patterns

**Nested spans** for complex workflows:

```python
def agent_task(query):
    with tracer.start_as_current_span("agent.task") as parent_span:
        parent_span.set_attribute("query", query)

        # Step 1: Plan
        with tracer.start_as_current_span("agent.plan") as plan_span:
            plan = create_plan(query)
            plan_span.set_attribute("plan.steps", len(plan))

        # Step 2: Execute
        with tracer.start_as_current_span("agent.execute") as exec_span:
            result = execute_plan(plan)
            exec_span.set_attribute("result.status", result.status)

        return result
```

This produces a hierarchical trace showing the planning and execution phases clearly.

**Error tracking**:

```python
def risky_operation():
    with tracer.start_as_current_span("risky.operation") as span:
        try:
            result = might_fail()
            span.set_status(Status(StatusCode.OK))
            return result
        except Exception as e:
            span.set_status(Status(StatusCode.ERROR, str(e)))
            span.record_exception(e)
            raise
```

Exceptions are captured in the span, making it easy to filter for failed operations in your observability dashboard.

### Hybrid Approach: Best of Both Worlds

Most production systems use a hybrid strategy:

1. **Auto-instrument** common operations (LLM calls, vector searches)
2. **Manually instrument** business-logic-specific operations (domain-specific reasoning, custom tools, proprietary workflows)

This maximizes coverage while maintaining flexibility [32].

Example:

```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import task, agent
from opentelemetry import trace

# Auto-instrumentation for standard libraries
Traceloop.init(app_name="hybrid-agent")

# Manual instrumentation for custom logic
tracer = trace.get_tracer(__name__)

@agent(name="customer_agent")  # Auto-traced by OpenLLMetry
def customer_support_agent(query, user_id):
    # Manually trace custom business logic
    with tracer.start_as_current_span("check.user.tier") as span:
        tier = get_user_tier(user_id)
        span.set_attribute("user.tier", tier)

    # Auto-traced LLM call
    response = generate_response(query, tier)

    return response

@task(name="user.tier.lookup")  # Custom decorator for semantic clarity
def get_user_tier(user_id):
    # Database call (manually traced if not auto-instrumented)
    with tracer.start_as_current_span("db.query.user_tier"):
        return db.execute("SELECT tier FROM users WHERE id = ?", user_id)
```

This approach gives you comprehensive traces with minimal boilerplate.

### Framework-Native Instrumentation

Some AI frameworks provide built-in observability without external libraries:

#### Microsoft Foundry
Native tracing for Semantic Kernel and Microsoft Agent Framework [33]:

```python
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.tracing import enable_tracing

# Enable built-in tracing
enable_tracing()

# All operations now produce OpenTelemetry traces
client = ChatCompletionsClient(endpoint="...", credential="...")
response = client.complete(messages=[...])
```

#### OpenAI Agents SDK
Built-in tracing support [34]:

```python
from openai_agents import Agent, trace_agent

agent = Agent(...)

# Traces are automatically generated
with trace_agent(agent):
    result = agent.run("Complete this task")
```

Framework-native instrumentation is ideal when available, as it's optimized for that framework's specific patterns and requires no additional dependencies.

### Performance Considerations

Tracing adds overhead. Best practices for production:

#### Asynchronous Exporting

Never block agent execution to send traces. Use batched, asynchronous export:

```python
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Batch spans and export asynchronously
processor = BatchSpanProcessor(
    otlp_exporter,
    max_queue_size=2048,
    schedule_delay_millis=5000,  # Export every 5 seconds
    max_export_batch_size=512
)
```

This ensures tracing never adds user-visible latency [35].

#### Sampling for High-Volume Systems

For systems processing millions of requests, trace 100% during development but sample in production:

```python
from opentelemetry.sdk.trace.sampling import TraceIdRatioBased

# Trace 10% of requests
sampler = TraceIdRatioBased(0.1)

provider = TracerProvider(sampler=sampler)
```

Combine with **head-based sampling** (sample entire traces) and **tail-based sampling** (sample based on duration or errors) for intelligent coverage.

#### Payload Truncation

Limit prompt/response sizes to prevent trace bloat:

```python
# Configure OpenLLMetry to truncate large payloads
Traceloop.init(
    app_name="my-agent",
    disable_batch=False,
    max_attribute_length=8000  # 8KB limit per attribute
)
```

---

## Key Takeaways

Full trace observability is the foundation of reliable AI agents in production. The key lessons from 2026:

1. **Traditional monitoring fails for AI agents**: Silent logical failures require observing agent reasoning, not just system health. 89% of organizations have implemented agent observability [6].

2. **Traces must be hierarchical and structured**: Capture reasoning spans, tool calls, memory access, and LLM invocations in a tree structure with parent-child relationships. This enables root cause analysis.

3. **Standards enable the ecosystem**: OpenTelemetry and OpenLLMetry provide vendor-neutral instrumentation. Instrument once, export everywhere. Major platforms (Datadog, Langfuse, Arize, Microsoft) have converged on these standards [20, 21].

4. **Start with auto-instrumentation**: Frameworks like OpenLLMetry, OpenLIT, and platform-native solutions (Azure AI Foundry, OpenAI Agents SDK) provide zero-code tracing for common frameworks [28, 33]. This gives immediate visibility.

5. **Augment with manual instrumentation**: Add custom spans for business-logic-specific operations that auto-instrumentation misses. The hybrid approach maximizes coverage with minimal effort [32].

6. **Performance matters**: Use asynchronous export, intelligent sampling, and payload truncation to ensure tracing doesn't impact user experience. Observability should be invisible to end users [35].

7. **Traces are the new code**: In traditional software, code documents the app. In AI systems, traces document behavior. They're not just debugging tools—they're the source of truth for understanding what your agent actually does [36].

The industry has spoken: comprehensive tracing is non-negotiable for production AI agents. The question is no longer *whether* to instrument, but *how deeply* and *how effectively*. The next section explores how to operationalize these traces in production monitoring and continuous evaluation.

---

## References

1. Horovits, D. (2025). *OpenTelemetry for GenAI and the OpenLLMetry project*. Medium. https://horovits.medium.com/opentelemetry-for-genai-and-the-openllmetry-project-81b9cea6a771

2. VictoriaMetrics Team. (2025). *AI Agents Observability with OpenTelemetry and the VictoriaMetrics Stack*. https://victoriametrics.com/blog/ai-agents-observability/

3. Bhatia, N. (2025). *Part 5 — Evaluating Agentic AI Systems: Solving Root Cause Diagnosis*. Medium. https://medium.com/@nitikab23/part-5-evaluating-agentic-ai-systems-the-complete-framework-solving-root-cause-diagnosis-24da2980203e

4. N-iX Team. (2026). *AI agent observability: The new standard for enterprise AI in 2026*. https://www.n-ix.com/ai-agent-observability/

5. Medium. (2025). *2026: The Year Agentic Architecture gets the operational lift*. https://medium.com/@aiforhuman/2026-the-year-agentic-architecture-gets-the-operational-lift-23faabadb5b7

6. LangChain. (2025). *State of AI Agents*. https://www.langchain.com/state-of-agent-engineering

7. OpenTelemetry. (2025). *AI Agent Observability - Evolving Standards and Best Practices*. https://opentelemetry.io/blog/2025/ai-agent-observability/

8. Portkey. (2025). *The complete guide to LLM observability for 2026*. https://portkey.ai/blog/the-complete-guide-to-llm-observability/

9. Maxim AI. (2026). *The 5 Best Agent Debugging Platforms in 2026*. https://www.getmaxim.ai/articles/the-5-best-agent-debugging-platforms-in-2026/

10. OpenTelemetry. (2024). *OpenTelemetry for Generative AI*. https://opentelemetry.io/blog/2024/otel-generative-ai/

11. GitHub - Traceloop. (2025). *OpenLLMetry: Open-source observability for your GenAI or LLM application*. https://github.com/traceloop/openllmetry

12. Agenta. (2025). *The AI Engineer's Guide to LLM Observability with OpenTelemetry*. https://agenta.ai/blog/the-ai-engineer-s-guide-to-llm-observability-with-opentelemetry

13. OpenTelemetry. (2025). *OpenTelemetry Official Documentation*. https://opentelemetry.io/

14. OpenTelemetry. (2025). *Semantic conventions for generative AI systems*. https://opentelemetry.io/docs/specs/semconv/gen-ai/

15. Datadog. (2024). *Datadog LLM Observability natively supports OpenTelemetry GenAI Semantic Conventions*. https://www.datadoghq.com/blog/llm-otel-semantic-convention/

16. SigNoz. (2026). *OpenTelemetry Agents - The Complete Beginner's Guide (2026)*. https://signoz.io/blog/opentelemetry-agent/

17. Horovits, D. (2025). *OpenTelemetry for GenAI and the OpenLLMetry project*. Medium. https://horovits.medium.com/opentelemetry-for-genai-and-the-openllmetry-project-81b9cea6a771

18. GitHub - Traceloop. (2025). *OpenLLMetry SDK and Documentation*. https://github.com/traceloop/openllmetry

19. liteLLM. (2025). *OpenTelemetry - Tracing LLMs with any observability tool*. https://docs.litellm.ai/docs/observability/opentelemetry_integration

20. OpenTelemetry. (2025). *AI Agent Observability - Evolving Standards and Best Practices*. https://opentelemetry.io/blog/2025/ai-agent-observability/

21. Datadog. (2024). *Datadog LLM Observability natively supports OpenTelemetry GenAI Semantic Conventions*. https://www.datadoghq.com/blog/llm-otel-semantic-convention/

22. Langfuse. (2025). *OpenTelemetry Integration - Langfuse Documentation*. https://langfuse.com/docs/integrations/opentelemetry

23. Arize AI. (2025). *Phoenix: Open-source AI Observability*. https://phoenix.arize.com/

24. Microsoft Learn. (2026). *Trace and Observe AI Agents in Microsoft Foundry*. https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/trace-agents-sdk

25. OpenTelemetry. (2025). *OTLP Protocol Specification*. https://opentelemetry.io/docs/specs/otlp/

26. AIMultiple. (2026). *Top 10+ Agentic Orchestration Frameworks & Tools in 2026*. https://research.aimultiple.com/agentic-orchestration/

27. Arize AI. (2025). *Agent Observability and Tracing*. https://arize.com/ai-agents/agent-observability/

28. Datadog. (2025). *Automatic Instrumentation for LLM Observability*. https://docs.datadoghq.com/llm_observability/instrumentation/auto_instrumentation/

29. AIMultiple. (2026). *Top 10+ Agentic Orchestration Frameworks & Tools in 2026*. https://research.aimultiple.com/agentic-orchestration/

30. Arize AI. (2025). *Agent Observability and Tracing*. https://arize.com/ai-agents/agent-observability/

31. OpenTelemetry. (2025). *Manual Instrumentation Guide*. https://opentelemetry.io/docs/instrumentation/manual/

32. OpenTelemetry. (2025). *AI Agent Observability - Evolving Standards and Best Practices*. https://opentelemetry.io/blog/2025/ai-agent-observability/

33. Microsoft Learn. (2026). *Trace and Observe AI Agents in Microsoft Foundry*. https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/trace-agents-sdk

34. OpenAI. (2025). *Tracing - OpenAI Agents SDK*. https://openai.github.io/openai-agents-python/tracing/

35. Maxim AI. (2026). *Best Tools to Monitor and Debug AI Agents in Production*. https://www.getmaxim.ai/articles/best-tools-to-monitor-and-debug-ai-agents-in-production/

36. LangChain. (2025). *In software, the code documents the app. In AI, the traces do*. https://blog.langchain.com/in-software-the-code-documents-the-app-in-ai-the-traces-do/

---

**Navigation:**
← [Section 8: Defining Custom Metrics](08_defining_custom_metrics.md) | [Table of Contents](../00_table_of_contents.md) | [Section 10: Production Monitoring and Observability](10_production_monitoring_and_observability.md) →
