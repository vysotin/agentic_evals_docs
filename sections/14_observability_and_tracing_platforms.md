# Section 14: Observability and Tracing Platforms

> **Part VII**: Tools & Platforms
> **Estimated Reading Time**: 18 minutes

## Overview

The AI agent observability landscape has matured rapidly, with specialized platforms emerging to address the unique challenges of monitoring autonomous, multi-step systems. Unlike traditional application performance monitoring (APM), agent observability requires capturing hierarchical traces of reasoning steps, tool invocations, memory access, and decision pathways. This section provides a comprehensive analysis of leading open-source and commercial observability platforms, their OpenTelemetry compatibility, and guidance for selecting the right solution for your needs.

## Table of Contents

- [14.1 Open-Source Solutions](#141-open-source-solutions)
- [14.2 Commercial Platforms](#142-commercial-platforms)
- [14.3 Platform Comparison Matrix](#143-platform-comparison-matrix)

---

## 14.1 Open-Source Solutions

Open-source observability platforms offer transparency, customization flexibility, and freedom from vendor lock-in. The emergence of OpenTelemetry as the industry standard has enabled a rich ecosystem of interoperable tools.

### 14.1.1 Langfuse (20.3k GitHub Stars)

**Langfuse** is an open-source LLM engineering platform that has become one of the most popular choices for AI agent observability. With over 20,000 GitHub stars, it provides comprehensive tracing, evaluation, prompt management, and analytics capabilities.

**Key Features:**
- **Full Trace Observability**: Captures hierarchical traces including all LLM calls, tool invocations, retrieval operations, and agent reasoning steps
- **Agent Graph Visualization**: LLM agents can be visualized as graphs to illustrate the flow of complex agentic workflows
- **Prompt Management**: Centralized versioning and management of prompts with strong caching for zero-latency deployments
- **Evaluations**: Supports LLM-as-a-judge, user feedback collection, manual labeling, and custom evaluation pipelines
- **Cost & Latency Tracking**: Real-time monitoring of token usage and associated costs across different providers
- **LLM Playground**: Interactive environment for testing and iterating on prompts with direct links from trace inspection

**OpenTelemetry Compatibility:**
Langfuse provides native OpenTelemetry support through its OTEL-native SDK v3, which automatically converts all emitted spans into rich Langfuse observations. It can operate as an OpenTelemetry Backend, receiving traces on the `/api/public/otel` (OTLP) endpoint using HTTP/protobuf.

**Deployment Options:**
- Self-hosted (Docker, Kubernetes)
- Cloud (SaaS hosted by Langfuse)

**Storage Backend:** PostgreSQL, ClickHouse, Redis/Valkey, S3/Blob Store

**Best For:** Teams seeking a comprehensive, fully open-source LLM engineering platform with strong community support and extensive integration options.

```python
# Example: Basic Langfuse integration
from langfuse import Langfuse

langfuse = Langfuse()

# Create a trace for an agent session
trace = langfuse.trace(
    name="customer-support-agent",
    user_id="user-123",
    metadata={"session_type": "support"}
)

# Log a reasoning step
span = trace.span(
    name="tool-selection",
    input={"query": "What is my order status?"},
    output={"selected_tool": "order_lookup"}
)
```

---

### 14.1.2 Arize Phoenix (8.2k GitHub Stars)

**Arize Phoenix** is an open-source AI observability and evaluation platform built on OpenTelemetry. It provides a comprehensive workflow for debugging and iterating on AI applications.

**Key Features:**
- **Tracing**: Full OpenTelemetry-based instrumentation with auto-instrumentation for popular frameworks
- **Evaluation**: Built-in evaluators for hallucination detection, relevance, toxicity, and custom metrics
- **Datasets & Experiments**: Create versioned datasets for experimentation, evaluation, and fine-tuning
- **Prompt Engineering**: Optimize prompts, compare models, adjust parameters, and replay traced LLM calls
- **Prompt Management**: Systematic version control, tagging, and experimentation for prompts

**OpenTelemetry Compatibility:**
Phoenix is built on top of OpenTelemetry and accepts traces over OTLP. It provides auto-instrumentation for popular frameworks including LlamaIndex, LangChain, DSPy, Mastra, and Vercel AI SDK.

**Deployment Options:**
- Self-hosted (Docker, Kubernetes)
- Cloud (Arize-hosted at app.phoenix.arize.com)

**Storage Backend:** SQLite, PostgreSQL

**Integration Highlight:** Amazon Bedrock Agents observability integration provides comprehensive traceability, systematic evaluation, and data-driven optimization capabilities for AWS users.

**Best For:** Teams wanting a free, self-hostable solution with a clear upgrade path to enterprise-grade Arize AX.

---

### 14.1.3 Langtrace (1.1k GitHub Stars)

**Langtrace** is an OpenTelemetry-native, end-to-end observability tool focused on real-time tracing and the three pillars of LLM observability: Usage, Accuracy, and Performance.

**Key Features:**
- **Real-Time Tracing**: Live visibility into agent execution with detailed span information
- **Evaluations**: Built-in accuracy evaluation capabilities
- **Prompt & Dataset Management**: Centralized management for prompts and evaluation datasets
- **Token & Cost Monitoring**: Detailed tracking of token usage and associated costs
- **Latency & Success Rate Monitoring**: Performance metrics with alerting capabilities

**OpenTelemetry Compatibility:**
Built on OpenTelemetry standards, Langtrace supports OTLP exporters and can integrate with existing observability stacks like Grafana, Datadog, and Honeycomb.

**Unique Differentiator:** Can be used with existing observability infrastructure without requiring a Langtrace API key, making it highly flexible for teams with established monitoring setups.

**Storage Backend:** PostgreSQL, ClickHouse

---

### 14.1.4 TruLens (3k GitHub Stars)

**TruLens** is an open-source evaluation and tracking tool for LLM experiments and AI agents, now shepherded by Snowflake.

**Key Features:**
- **Feedback Functions**: Programmatic evaluation using customizable feedback functions
- **RAG Triad**: Specialized evaluation framework for RAG applications measuring context relevance, groundedness, and answer relevance
- **Honest, Harmless, Helpful Evals**: Built-in evaluators for safety and quality assessment
- **OpenTelemetry Interoperability**: Can use spans emitted by non-TruLens code and emit spans for other systems

**OpenTelemetry Compatibility:**
TruLens is OpenTelemetry compatible and uses OTLP for trace export. It can interoperate with other OpenTelemetry-instrumented systems.

**Storage Backend:** SQLite, PostgreSQL

**Best For:** Teams focused on programmatic evaluation with strong emphasis on RAG system assessment.

---

### 14.1.5 Evidently AI (7k GitHub Stars)

**Evidently AI** is an open-source platform for evaluating, testing, and monitoring both ML and LLM systems, from experiments to production.

**Key Features:**
- **100+ Built-in Metrics**: Comprehensive metric library for diverse evaluation needs
- **LLM Evaluation**: Specialized metrics for language model assessment
- **Data Drift Detection**: Monitor for distribution shifts in inputs and outputs
- **Test Suites**: Structured testing framework for quality assurance
- **Monitoring Dashboard**: Production monitoring with alerting capabilities
- **Synthetic Data Generation**: Generate test data for evaluation

**OpenTelemetry Compatibility:**
Tracing is based on OpenTelemetry using the open-source Tracely library. Can receive and process OpenTelemetry traces.

**Storage Backend:** File system, SQLite, PostgreSQL, S3-compatible storage

**Best For:** Teams needing comprehensive ML and LLM evaluation with rich built-in metrics and drift detection.

---

### 14.1.6 MLflow Tracing (23.6k GitHub Stars)

**MLflow Tracing** is a fully OpenTelemetry-compatible LLM observability solution from Databricks, representing the evolution of the popular MLflow platform into the GenAI era.

**Key Features:**
- **One-Line Auto Tracing**: Simple instrumentation for 20+ GenAI libraries
- **Full OpenTelemetry Support**: OTLP endpoint at `/v1/traces` for receiving traces from any language
- **Multi-threaded & Async Support**: Production-ready performance characteristics
- **PII Redaction**: Built-in privacy protection
- **Sampling**: Configurable sampling for high-volume production environments
- **Lightweight SDK**: 95% smaller footprint for production deployments
- **TypeScript Support**: New tracing SDK for JavaScript/TypeScript environments
- **Feedback Tracking**: Native support for human feedback, ground truth, and LLM judges

**OpenTelemetry Compatibility:**
MLflow 3.6.0 (November 2025) brought full OpenTelemetry support to the open-source MLflow server. Key capabilities include:
- OTLP endpoint for receiving traces
- Dual export to MLflow and other OpenTelemetry-compatible backends
- Span-level statistics exported as OpenTelemetry metrics
- Support for traces from any OpenTelemetry-compatible language (Java, Go, Rust, etc.)

**Framework Integrations:**
- Python: OpenAI, Anthropic, LangChain, LlamaIndex, DSPy, AutoGen, CrewAI, Pydantic AI
- TypeScript: Vercel AI SDK, LangChain.js, Mastra, Anthropic SDK, Gemini SDK

**Storage Backend:** SQLite, PostgreSQL, MySQL, MSSQL

**Cost Advantage:** 100% free and open-source with no SaaS costs. All trace data remains on your infrastructure.

**Best For:** Teams already using MLflow or wanting a framework-agnostic, fully open-source solution with strong OpenTelemetry integration.

```python
# Example: MLflow Tracing with auto-instrumentation
import mlflow
from openai import OpenAI

# Enable auto-tracing for OpenAI
mlflow.openai.autolog()

client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "What is AI agent evaluation?"}]
)

# Traces are automatically captured and sent to MLflow
```

---

### 14.1.7 Helicone (4.9k GitHub Stars)

**Helicone** is an open-source AI Gateway and LLM observability platform providing unified access to over 100 AI models.

**Key Features:**
- **AI Gateway**: Single API for 100+ LLM providers with intelligent routing and automatic fallbacks
- **Quick Integration**: Minimal code changes required for full observability
- **Playground**: Interactive environment for prompt testing
- **Prompt Management**: Version control and management for prompts
- **Fine-tuning Support**: Tools for model customization
- **Enterprise Ready**: SOC 2 and GDPR compliant

**OpenTelemetry Compatibility:**
Supports asynchronous logging for multiple LLM platforms through OpenLLMetry integration.

**Storage Backend:** Supabase (PostgreSQL), ClickHouse, Minio

**Best For:** Teams needing a unified gateway for multiple LLM providers with built-in observability.

---

### 14.1.8 Lunary

**Lunary** is an open-source observability platform focused on enterprise-grade features including security and compliance.

**Key Features:**
- **LLM Tracing**: Comprehensive trace capture for LLM applications
- **Prompt Management**: Centralized prompt versioning and deployment
- **Real-time Analytics**: Live performance monitoring
- **Error Detection**: Automated identification of failures and anomalies
- **PII Masking**: Built-in privacy protection
- **Role-Based Access Control (RBAC)**: Enterprise security features
- **Single Sign-On (SSO)**: Enterprise authentication integration

**OpenTelemetry Compatibility:**
First-class support for OpenTelemetry with a dedicated OTLP endpoint. Compatible with OpenLLMetry, OpenLIT, and Arize OpenInference.

**Best For:** Enterprise teams requiring strong security and compliance features alongside observability.

---

## 14.2 Commercial Platforms

Commercial platforms offer managed infrastructure, enterprise support, and often more polished user experiences in exchange for subscription costs.

### 14.2.1 LangSmith (LangChain)

**LangSmith** is the commercial observability and evaluation platform from LangChain, providing deep integration with the LangChain ecosystem.

**Key Features:**
- **Complete Visibility**: Step-by-step tracing of agent behavior with detailed span information
- **Live Dashboards**: Real-time monitoring of costs, latency, and response quality
- **Alerts**: Configurable alerting for quality degradation and anomalies
- **Offline & Online Evaluation**: Support for both benchmark testing and production traffic evaluation
- **Evaluation Methods**: Human review queues, heuristic checks, LLM-as-judge scoring, pairwise comparisons
- **OpenTelemetry Support**: OTel integration for unified observability across services

**Deployment Options:**
- Cloud SaaS (free tier: 5K traces monthly)
- Self-hosted (Enterprise plan only)

**Best For:** Teams heavily invested in the LangChain/LangGraph ecosystem seeking seamless integration.

---

### 14.2.2 Braintrust

**Braintrust** is an enterprise-grade AI evaluation and observability platform focused on quality-driven development.

**Key Features:**
- **Production Monitoring**: Real-time tracking with quality alerts and anomaly detection
- **Online Scoring**: Continuous evaluation of production traffic using the same metrics as pre-deployment testing
- **CI/CD Integration**: GitHub Actions integration with automatic eval execution on pull requests
- **Experiment Tracking**: Version-controlled experiments with side-by-side comparisons
- **A/B Testing**: Production traffic splitting for model comparison
- **Brainstore**: Purpose-built database for AI application logs and traces (80x faster queries)

**OpenTelemetry Compatibility:**
Features an OpenTelemetry compatibility mode for seamless tracing and natively supports OpenTelemetry.

**Reported Results:** Customers consistently report accuracy improvements of 30% or more within weeks of adoption.

**Best For:** Enterprise teams prioritizing quality gates and evaluation-driven development workflows.

---

### 14.2.3 Maxim AI

**Maxim AI** is an end-to-end platform unifying simulation, evaluation, and observability across the AI agent lifecycle.

**Key Features:**
- **Three-Layer Evaluation Framework**:
  - System Efficiency Layer: Latency, token usage, tool call counts
  - Session-Level Outcomes: End-to-end task success, trajectory quality
  - Node-Level Precision: Tool selection accuracy, step-level metrics
- **User Simulator**: Test agent performance with simulated user interactions
- **Comprehensive Metrics**: Extensive metric library for diverse evaluation needs

**OpenTelemetry Compatibility:**
Built on OpenTelemetry standards, offering vendor-agnostic and framework-independent observability.

**Best For:** Teams needing comprehensive simulation, evaluation, and observability in a unified platform.

---

### 14.2.4 Openlayer

**Openlayer** provides integrated development testing, production monitoring, and security guardrails.

**Key Features:**
- **Development Testing**: Pre-deployment validation workflows
- **Production Monitoring**: Real-time quality tracking
- **Security Guardrails**: Built-in safety checks and policy enforcement

**OpenTelemetry Compatibility:**
Any OpenTelemetry-compatible instrumentation can be used to export traces to Openlayer.

---

### 14.2.5 WhyLabs

**WhyLabs** is an AI observability platform emphasizing data quality, model monitoring, and explainability.

**Key Features:**
- **Data Quality Monitoring**: Track input and output data characteristics
- **Model Monitoring**: Performance tracking over time
- **Explainability**: Tools for understanding model decisions
- **Statistical Analysis**: Deep analysis of model inputs and outputs

**OpenTelemetry Compatibility:**
Medium compatibility—focused more on statistical analysis. Data from OTel-instrumented systems can be integrated.

---

### 14.2.6 Monte Carlo

**Monte Carlo** is a data and AI observability leader providing AI-powered anomaly detection and root-cause analysis.

**Key Features:**
- **AI-Powered Anomaly Detection**: Intelligent identification of quality issues
- **Root-Cause Analysis**: Automated diagnosis of failures
- **Trace Telemetry Consolidation**: Unified view of AI agent traces

**OpenTelemetry Compatibility:**
Leverages the OpenTelemetry framework for deployment, consolidating and visualizing AI agent trace telemetry.

---

### 14.2.7 LangWatch

**LangWatch** is a privacy-first LLM evaluation platform with a focus on agent simulations and unified evaluations.

**Key Features:**
- **Privacy-First Design**: Data privacy as a core principle
- **Agent Simulations**: Test agents in simulated environments
- **Unified Evaluations**: Consistent evaluation framework across use cases

**OpenTelemetry Compatibility:**
Fully compatible with OpenTelemetry, allowing use of any OpenTelemetry-compatible instrumentation library.

---

## 14.3 Platform Comparison Matrix

### 14.3.1 Feature Comparison

| Platform | License | Tracing | Evaluation | Prompt Mgmt | Cost Tracking | Self-Host |
|----------|---------|---------|------------|-------------|---------------|-----------|
| Langfuse | MIT | ✅ | ✅ | ✅ | ✅ | ✅ |
| Arize Phoenix | Elastic 2.0 | ✅ | ✅ | ✅ | ✅ | ✅ |
| Langtrace | AGPL-3.0 | ✅ | ✅ | ✅ | ✅ | ✅ |
| TruLens | MIT | ✅ | ✅ | ❌ | ❌ | ✅ |
| Evidently AI | Apache 2.0 | ✅ | ✅ | ❌ | ❌ | ✅ |
| MLflow | Apache-2.0 | ✅ | ✅ | ❌ | ✅ | ✅ |
| Helicone | Apache 2.0 | ✅ | ✅ | ✅ | ✅ | ✅ |
| Lunary | MIT | ✅ | ✅ | ✅ | ✅ | ✅ |
| LangSmith | Commercial | ✅ | ✅ | ✅ | ✅ | Enterprise |
| Braintrust | Commercial | ✅ | ✅ | ✅ | ✅ | ❌ |
| Maxim AI | Commercial | ✅ | ✅ | ✅ | ✅ | ❌ |

### 14.3.2 OpenTelemetry Compatibility

| Platform | OTLP Support | OpenLLMetry | Native OTel SDK | Export to OTel Backends |
|----------|--------------|-------------|-----------------|------------------------|
| Langfuse | ✅ HTTP/protobuf | ✅ | ✅ (v3) | ✅ |
| Arize Phoenix | ✅ | ✅ | ✅ | ✅ |
| Langtrace | ✅ | ✅ | ✅ | ✅ |
| TruLens | ✅ | ✅ | ✅ | ✅ |
| Evidently AI | ✅ | ✅ | Via Tracely | ✅ |
| MLflow | ✅ Full | ✅ | ✅ | ✅ (Dual export) |
| Helicone | Partial | ✅ | ❌ | ❌ |
| Lunary | ✅ | ✅ | ✅ | ✅ |
| LangSmith | ✅ | Partial | ❌ | ✅ |
| Braintrust | ✅ | ✅ | ✅ | ✅ |

### 14.3.3 Storage Backends

| Platform | Primary Storage | Alternatives |
|----------|-----------------|--------------|
| Langfuse | PostgreSQL | ClickHouse, Redis, S3 |
| Arize Phoenix | SQLite | PostgreSQL |
| Langtrace | PostgreSQL | ClickHouse |
| TruLens | SQLite | PostgreSQL |
| Evidently AI | File system | SQLite, PostgreSQL, S3 |
| MLflow | SQLite | PostgreSQL, MySQL, MSSQL |
| Helicone | Supabase (PostgreSQL) | ClickHouse, Minio |
| Lunary | Not specified | - |

### 14.3.4 Deployment Options

| Platform | Docker | Kubernetes | Cloud SaaS | Air-Gapped |
|----------|--------|------------|------------|------------|
| Langfuse | ✅ | ✅ | ✅ | ✅ |
| Arize Phoenix | ✅ | ✅ | ✅ | ✅ |
| Langtrace | ✅ | ✅ | ✅ | ✅ |
| TruLens | ✅ | ❌ | ❌ | ✅ |
| Evidently AI | ✅ | ✅ | ✅ | ✅ |
| MLflow | ✅ | ✅ | ✅ | ✅ |
| Helicone | ✅ | ✅ (Helm) | ✅ | ✅ |
| Lunary | ✅ | ✅ | ✅ | ✅ |
| LangSmith | ❌ | Enterprise | ✅ | Enterprise |
| Braintrust | ❌ | ❌ | ✅ | ❌ |

---

## Key Takeaways

1. **OpenTelemetry is the standard**: Nearly all modern platforms are built on or compatible with OpenTelemetry, enabling interoperability and preventing vendor lock-in

2. **Open-source options are mature**: Platforms like Langfuse, Phoenix, and MLflow provide production-ready capabilities at zero cost

3. **MLflow's evolution is significant**: Full OpenTelemetry support in MLflow 3.6.0 makes it a compelling choice for teams already using the MLflow ecosystem

4. **Choose based on existing stack**: Teams using LangChain benefit from LangSmith's tight integration; MLflow users should consider MLflow Tracing; framework-agnostic teams have many options

5. **Consider deployment requirements**: Self-hosting capabilities vary significantly—ensure your chosen platform meets compliance and data residency requirements

6. **Evaluation integration matters**: The best observability platforms integrate evaluation capabilities, enabling the feedback loops essential for continuous improvement

---

## References

1. Langfuse. (2025). *Langfuse: Open Source LLM Engineering Platform*. [https://langfuse.com/](https://langfuse.com/)

2. Langfuse. (2025). *Open Source LLM Observability via OpenTelemetry*. [https://langfuse.com/integrations/native/opentelemetry](https://langfuse.com/integrations/native/opentelemetry)

3. Arize AI. (2025). *Phoenix: AI Observability & Evaluation*. GitHub. [https://github.com/Arize-ai/phoenix](https://github.com/Arize-ai/phoenix)

4. MLflow. (2025). *Full OpenTelemetry Support in MLflow Tracing*. [https://mlflow.org/blog/opentelemetry-tracing-support](https://mlflow.org/blog/opentelemetry-tracing-support)

5. MLflow. (2025). *MLflow Tracing for LLM Observability*. [https://mlflow.org/docs/latest/genai/tracing/](https://mlflow.org/docs/latest/genai/tracing/)

6. LangChain. (2025). *LangSmith Observability*. [https://www.langchain.com/langsmith/observability](https://www.langchain.com/langsmith/observability)

7. Braintrust. (2026). *The 4 Best LLM Monitoring Tools in 2026*. [https://www.braintrust.dev/articles/best-llm-monitoring-tools-2026](https://www.braintrust.dev/articles/best-llm-monitoring-tools-2026)

8. O-mega. (2026). *Top 5 AI Agent Observability Platforms 2026 Guide*. [https://o-mega.ai/articles/top-5-ai-agent-observability-platforms-the-ultimate-2026-guide](https://o-mega.ai/articles/top-5-ai-agent-observability-platforms-the-ultimate-2026-guide)

9. AWS. (2025). *Amazon Bedrock Agents Observability Using Arize AI*. [https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agents-observability-using-arize-ai/](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agents-observability-using-arize-ai/)

10. Traceloop. (2025). *OpenLLMetry: Open-Source Observability for GenAI*. GitHub. [https://github.com/traceloop/openllmetry](https://github.com/traceloop/openllmetry)

11. Statsig. (2025). *Arize Phoenix Overview: Open-Source AI Observability*. [https://www.statsig.com/perspectives/arize-phoenix-ai-observability](https://www.statsig.com/perspectives/arize-phoenix-ai-observability)

12. LakeFS. (2026). *LLM Observability Tools: 2026 Comparison*. [https://lakefs.io/blog/llm-observability-tools/](https://lakefs.io/blog/llm-observability-tools/)

---

**Navigation:**
← [Previous Section: Evaluation-Driven Development](13_evaluation_driven_development.md) | [Table of Contents](../README.md) | [Next Section: Evaluation Frameworks and Libraries](15_evaluation_frameworks_and_libraries.md) →
