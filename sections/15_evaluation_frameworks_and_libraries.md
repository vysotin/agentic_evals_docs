# Section 15: Evaluation Frameworks and Libraries

> **Part VII**: Tools & Platforms
> **Estimated Reading Time**: 16 minutes

## Overview

Evaluation frameworks provide the building blocks for systematically assessing AI agent performance. Unlike observability platforms that focus on tracing and monitoring, evaluation frameworks specialize in defining metrics, running test suites, and scoring outputs. This section covers general-purpose evaluation frameworks, instrumentation libraries, and OpenTelemetry backends that form the foundation of modern AI agent evaluation infrastructure.

## Table of Contents

- [15.1 General-Purpose Frameworks](#151-general-purpose-frameworks)
- [15.2 Instrumentation Libraries](#152-instrumentation-libraries)
- [15.3 General-Purpose OpenTelemetry Backends](#153-general-purpose-opentelemetry-backends)

---

## 15.1 General-Purpose Frameworks

General-purpose evaluation frameworks provide structured approaches to testing and scoring LLM and agent outputs across diverse use cases.

### 15.1.1 OpenAI Evals

**OpenAI Evals** is an open-source framework and registry for evaluating LLMs and LLM systems, developed by OpenAI and widely adopted across the industry.

**Key Features:**
- **Evaluation Types**:
  - **Basic (Ground-Truth) Evals**: Compare model outputs to known correct answers using deterministic checks
  - **Model-Graded Evals**: Use another AI model to evaluate whether outputs meet desired goals
- **Registry of Evals**: Pre-built tests covering question answering, logic puzzles, code generation, and content compliance
- **Custom Eval Support**: Create custom evaluations using proprietary data and domain-specific criteria
- **API and Dashboard**: Both programmatic and visual interfaces for running evaluations
- **Graders**: Automated scoring with the Evals API for eval-driven development
- **Prompt Optimizer**: Tools for improving prompts based on evaluation results

**2025-2026 Developments:**
OpenAI introduced the Evals API for eval-driven development, enabling a tighter "measure → improve → ship" loop. Graders and the Prompt optimizer help teams iterate more effectively.

**Use Cases:**
- Model selection through objective comparison
- Continuous quality assurance with regression detection
- Fine-tuning validation
- Pre-deployment benchmarking

**Best Practices:**
By 2026, the most effective evaluation stacks prioritize "traceability"—the ability to link a specific evaluation score back to the exact version of the prompt, model, and dataset that produced it.

```python
# Example: Custom OpenAI Eval
from evals.elsuite import BasicEval

class CustomAccuracyEval(BasicEval):
    def eval_sample(self, sample, *args):
        prompt = sample["input"]
        expected = sample["ideal"]

        # Get model response
        response = self.completion_fn(prompt)

        # Custom scoring logic
        score = self.calculate_accuracy(response, expected)
        return {"score": score}
```

**GitHub:** [github.com/openai/evals](https://github.com/openai/evals)

---

### 15.1.2 DeepEval (Confident AI)

**DeepEval** is an open-source LLM evaluation framework designed to work like Pytest for LLM outputs, providing 50+ plug-and-use metrics with research backing.

**Key Features:**
- **50+ Built-in Metrics**: Comprehensive library including G-Eval, hallucination detection, faithfulness, relevancy, and toxicity
- **Self-Explaining Metrics**: Each metric provides detailed insights into why a score falls short and how it can be improved
- **Agentic Metrics**: Six dedicated metrics for evaluating agent execution flows:
  - `PlanQualityMetric`: Evaluates the quality of agent planning
  - `PlanAdherenceMetric`: Measures how well agents follow their plans
  - `ToolCorrectnessMetric`: Assesses tool selection and usage accuracy
  - `StepEfficiencyMetric`: Evaluates the efficiency of reasoning steps
- **Custom Metrics**: Easy-to-build custom metrics using the `BaseMetric` class
- **Red Teaming**: Test for 40+ safety vulnerabilities with built-in red team capabilities (see also: DeepTeam)
- **Synthetic Data Generation**: State-of-the-art evolution techniques for test data creation
- **CI/CD Integration**: Seamless integration with any CI/CD environment

**Supported Evaluation Areas:**
- RAG systems
- AI agents
- Chatbots
- Code generation
- Custom use cases

**Model Provider Integrations:**
Ollama, Azure OpenAI, Anthropic, Gemini, and more.

```python
# Example: DeepEval agent evaluation
from deepeval import evaluate
from deepeval.metrics import ToolCorrectnessMetric, PlanAdherenceMetric
from deepeval.test_case import LLMTestCase

# Define test case with agent trace
test_case = LLMTestCase(
    input="What's the weather in Tokyo?",
    actual_output="The weather in Tokyo is 22°C and sunny.",
    retrieval_context=["Tokyo weather data: 22°C, sunny, humidity 65%"],
    tools_called=["weather_api"],
    expected_tools=["weather_api"]
)

# Run evaluation with multiple metrics
tool_metric = ToolCorrectnessMetric(threshold=0.8)
plan_metric = PlanAdherenceMetric(threshold=0.7)

evaluate([test_case], [tool_metric, plan_metric])
```

**Best For:** Teams wanting pytest-like testing experience for LLMs with comprehensive agentic metrics.

**GitHub:** [github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval)

---

### 15.1.3 RAGAS

**RAGAS** (Retrieval Augmented Generation Assessment) is a framework for reference-free evaluation of RAG pipelines, pioneering the LLM-as-judge approach for RAG systems.

**Key Features:**
- **Reference-Free Evaluation**: Assess RAG quality without requiring ground truth labels
- **Component-Level Metrics**:
  - **Retrieval Component**: `context_relevancy`, `context_recall`, `context_precision`
  - **Generation Component**: `faithfulness`, `answer_relevancy`
- **Custom Metrics**: Create tailored metrics with simple decorators
- **Synthetic Data Generation**: Generate test queries and stress-test retrieval pipelines
- **Framework Integrations**: Built-in support for LangChain, LlamaIndex, and more

**Core Metrics Explained:**
- **Context Precision**: Measures signal-to-noise ratio of retrieved context
- **Context Recall**: Measures if all relevant information was retrieved
- **Faithfulness**: Measures factual accuracy of generated answers against context
- **Answer Relevancy**: Measures how well answers address the original question

**2026 Position:**
RAGAS is recognized as one of the top 5 RAG evaluation platforms, particularly suited for research teams building custom evaluation infrastructure or organizations requiring transparent, modifiable metrics.

```python
# Example: RAGAS evaluation
from ragas import evaluate
from ragas.metrics import faithfulness, context_precision, answer_relevancy
from datasets import Dataset

# Prepare evaluation dataset
data = {
    "question": ["What is the capital of France?"],
    "answer": ["Paris is the capital of France."],
    "contexts": [["Paris is the capital and largest city of France."]],
    "ground_truth": ["Paris"]
}
dataset = Dataset.from_dict(data)

# Run evaluation
results = evaluate(
    dataset,
    metrics=[faithfulness, context_precision, answer_relevancy]
)
print(results)
```

**Best For:** Teams focused specifically on RAG system evaluation with need for transparent, reference-free metrics.

**Documentation:** [docs.ragas.io](https://docs.ragas.io/en/stable/)

---

### 15.1.4 PromptFoo

**PromptFoo** is an open-source tool for quick A/B testing of prompts and agents with LLM-judged evaluations.

**Key Features:**
- **Rapid Testing**: Quick comparison of different prompts and model configurations
- **LLM-as-Judge**: Automated evaluation using AI judges
- **Custom Assertions**: Define specific criteria for pass/fail determinations
- **Output Comparisons**: Side-by-side analysis of different approaches
- **CI/CD Ready**: Integration with development pipelines

**Best For:** Teams needing rapid iteration on prompts with minimal setup.

---

## 15.2 Instrumentation Libraries

Instrumentation libraries provide the foundational layer for capturing telemetry data from AI applications. These libraries focus on generating traces that can be sent to any compatible backend.

### 15.2.1 OpenLLMetry (6.7k GitHub Stars)

**OpenLLMetry** by Traceloop is the leading open-source observability toolkit built on OpenTelemetry, specifically designed for LLM applications.

**Key Features:**
- **Auto-Instrumentation**: Automatic tracing for LLMs, vector databases, and frameworks
- **Multi-Language Support**: SDKs for Python, TypeScript, Go, and Ruby
- **Extensible**: Workflow and task annotations for custom instrumentation
- **Backend-Agnostic**: Works with any OpenTelemetry-compatible backend
- **Semantic Conventions**: Extends OpenTelemetry conventions for LLM-specific data

**Supported LLM Providers:**
OpenAI, Anthropic, Cohere, HuggingFace, Replicate, AWS Bedrock, AWS SageMaker, Vertex AI, Aleph Alpha, Gemini, Groq, IBM Watsonx, Mistral AI, Ollama, Together AI, WRITER

**Supported Frameworks:**
LangChain, LlamaIndex, Haystack, CrewAI, LangGraph, LiteLLM, OpenAI Agents, Agno, AWS Strands, Langflow

**Deployment Options:**
- Self-hosted with OpenTelemetry Collector
- Cloud via various observability platforms
- Docker and Kubernetes

```python
# Example: OpenLLMetry setup (2 lines!)
from traceloop.sdk import Traceloop

Traceloop.init(app_name="my-agent-app")

# All LLM calls are now automatically traced
```

**Unique Value:** OpenLLMetry's tight integration with OpenTelemetry enables seamless use with existing observability stacks while actively contributing to the standardization of LLM observability within the OpenTelemetry project.

**GitHub:** [github.com/traceloop/openllmetry](https://github.com/traceloop/openllmetry)

---

### 15.2.2 OpenLIT (2.1k GitHub Stars)

**OpenLIT** is an OpenTelemetry-native AI engineering platform providing comprehensive observability with a focus on the complete development lifecycle.

**Key Features:**
- **Analytics Dashboard**: Visual insights into LLM application performance
- **Cost Tracking**: Support for custom and fine-tuned models
- **Exceptions Monitoring**: Dedicated dashboard for error tracking
- **Prompt Management**: Centralized prompt versioning and management
- **API Keys Management**: Secure handling of provider credentials
- **Fleet Hub**: OpAMP management for distributed deployments
- **Zero-Code Kubernetes Observability**: Easy deployment in containerized environments

**Supported Providers:**
OpenAI, Ollama, Anthropic, Deepseek, Cohere, Mistral, Azure OpenAI, HuggingFace, Amazon Bedrock, Vertex AI, Groq, NVIDIA NIM, and many more.

**Supported Frameworks:**
LangChain, OpenAI Agents, LiteLLM, CrewAI, LlamaIndex, Browser Use, Pydantic, DSPy, AG2, Haystack, Mem0, Guardrails AI, and more.

**Storage Backend:** ClickHouse, SQLite

**GitHub:** [github.com/openlit/openlit](https://github.com/openlit/openlit)

---

### 15.2.3 OpenInference (798 GitHub Stars)

**OpenInference** by Arize AI is a set of conventions and plugins complementary to OpenTelemetry, designed to enable tracing of AI applications with focus on standardization.

**Key Features:**
- **Specification-Driven**: Clear conventions for LLM observability data
- **Transport-Agnostic**: Works with any OTEL-compatible collector
- **Community-Driven**: Open development with broad input
- **Multi-Language**: Support for Python, TypeScript, and Java
- **Framework Coverage**: LlamaIndex, DSPy, LangChain, Guardrails, CrewAI, Haystack, and more

**Supported Providers:**
OpenAI, AWS Bedrock, MistralAI, VertexAI, Anthropic, Google GenAI, Groq

**Unique Value:** OpenInference is not a standalone product but a specification that provides standardized conventions for LLM observability, complementing OpenTelemetry.

**GitHub:** [github.com/Arize-ai/openinference](https://github.com/Arize-ai/openinference)

---

### 15.2.4 OpenLIT SDK

The **OpenLIT SDK** is a monitoring framework built on OpenTelemetry providing comprehensive observability for the entire AI stack.

**Key Features:**
- **Full Stack Coverage**: Traces, metrics, and logs for AI applications
- **OpenTelemetry Native**: Built on and fully compatible with OTel standards
- **SDK Options**: Available for multiple languages

---

### 15.2.5 MLflow Tracing SDK

**MLflow Tracing SDK** provides both Python and TypeScript SDKs for comprehensive LLM observability.

**Key Features:**
- **One-Line Auto Tracing**: Simple instrumentation for 20+ GenAI libraries
- **Manual Tracing SDK**: Fine-grained control when needed
- **Multi-Threaded & Async Support**: Production-ready concurrency handling
- **PII Redaction**: Built-in privacy protection
- **Lightweight Production SDK**: 95% smaller footprint (mlflow-tracing package)

**TypeScript Enhancements (2025-2026):**
Auto-tracing support added for Vercel AI SDK, LangChain.js, Mastra, Anthropic SDK, and Gemini SDK.

**Documentation:** [mlflow.org/docs/latest/genai/tracing/](https://mlflow.org/docs/latest/genai/tracing/)

---

### 15.2.6 MLflow Lightweight SDK (mlflow-tracing)

For production environments where footprint matters, the **mlflow-tracing** package provides a minimal SDK focused purely on trace generation.

**Key Features:**
- **95% Smaller Footprint**: Minimal dependencies
- **Production Optimized**: Designed for high-throughput environments
- **Full Compatibility**: Same trace format as the full MLflow SDK

---

## 15.3 General-Purpose OpenTelemetry Backends

While LLM-specific platforms provide specialized features, general-purpose OpenTelemetry backends offer powerful, scalable infrastructure for teams building custom observability solutions.

### 15.3.1 Jaeger (22.3k GitHub Stars)

**Jaeger** is a CNCF-graduated distributed tracing system, originally created by Uber Technologies.

**Key Features:**
- **Distributed Tracing**: End-to-end transaction monitoring
- **Service Dependency Analysis**: Visualize relationships between services
- **Performance Monitoring**: Latency and throughput tracking
- **Root Cause Analysis**: Drill down into failure causes
- **Adaptive Sampling**: Intelligent trace sampling for high-volume environments
- **Post-Collection Processing**: Transform and enrich trace data

**OpenTelemetry Compatibility:**
Jaeger v2 is built on the OpenTelemetry Collector framework, receiving trace data via OTLP and supporting various OTel components (receivers, processors, exporters).

**Storage Backends:** Elasticsearch, OpenSearch, Cassandra, Badger, Kafka, Memory

**Deployment Options:**
- Self-hosted (Docker, Kubernetes)
- All-in-one binary for development

**GitHub:** [github.com/jaegertracing/jaeger](https://github.com/jaegertracing/jaeger)

---

### 15.3.2 SigNoz (25.2k GitHub Stars)

**SigNoz** is an open-source observability platform providing unified logs, traces, and metrics in a single application—positioned as an open-source alternative to Datadog and New Relic.

**Key Features:**
- **Application Performance Monitoring (APM)**: Full-stack performance visibility
- **Distributed Tracing**: Complete request path tracking
- **Log Management**: Centralized log aggregation and search
- **Infrastructure Monitoring**: Host and container metrics
- **Exception Monitoring**: Error tracking and analysis
- **Alerting**: Configurable alert rules

**OpenTelemetry Compatibility:**
OpenTelemetry-native with native OTLP support. Provides an OpenTelemetry collector to receive data in OTLP format.

**LLM-Specific Support:**
Support for OpenAI, Anthropic, Amazon Bedrock, Azure OpenAI, Google Gemini, DeepSeek, Grok with framework integrations including LangChain, LlamaIndex, AutoGen, CrewAI, LiteLLM, Semantic Kernel, and Vercel AI SDK.

**Storage Backend:** ClickHouse

**Unique Value:** Unified platform for logs, metrics, and traces eliminates the need for multiple tools while being OpenTelemetry-native to prevent vendor lock-in.

**GitHub:** [github.com/SigNoz/signoz](https://github.com/SigNoz/signoz)

---

### 15.3.3 Grafana Tempo (5k GitHub Stars)

**Grafana Tempo** is a high-scale distributed tracing backend designed for cost-efficiency through object storage.

**Key Features:**
- **Cost-Effective at Scale**: Uses object storage instead of indexing
- **Deep Grafana Integration**: Seamless use with Prometheus and Loki
- **TraceQL**: Powerful query language for trace analysis
- **Traces Drilldown UI**: Intuitive analysis interface
- **Multi-Protocol Support**: Jaeger, Zipkin, Kafka, OpenCensus, OpenTelemetry

**OpenTelemetry Compatibility:**
Receiver layer, wire format, and storage format are all based directly on OpenTelemetry standards.

**Storage Backend:** Azure, GCS, S3, local disk

**LLM Support:**
Integrations with OpenAI, Anthropic, Bedrock, Cohere, Watsonx, Gemini, Mistral AI, together.ai, Ollama via OpenLLMetry.

**Deployment Options:**
- Self-hosted
- Grafana Cloud
- Grafana Enterprise Traces
- Docker, Helm, Jsonnet

**Unique Value:** Cost-effectiveness at scale—by not indexing traces and relying on object storage, Tempo can store massive volumes of trace data at a fraction of traditional costs.

**GitHub:** [github.com/grafana/tempo](https://github.com/grafana/tempo)

---

### 15.3.4 Uptrace (4k GitHub Stars)

**Uptrace** is an open-source APM and observability platform unifying traces, metrics, and logs with a focus on cost savings.

**Key Features:**
- **Service Graph**: Visual representation of service dependencies
- **RED Metrics**: Rate, Errors, Duration tracking
- **Latency Percentiles**: P50, P95, P99 monitoring
- **Error Pattern Analysis**: Intelligent error grouping
- **Log Pattern Analysis**: Automated log categorization
- **Alerting**: Configurable notifications

**OpenTelemetry Compatibility:**
OpenTelemetry-native with support for ingestion from OpenTelemetry, Prometheus, Vector, FluentBit, and CloudWatch. Can be used as a Tempo/Prometheus datasource in Grafana.

**Storage Backend:** PostgreSQL, ClickHouse

**Cost Claim:** Up to 80% cost savings compared to Datadog with predictable pricing model.

**GitHub:** [github.com/uptrace/uptrace](https://github.com/uptrace/uptrace)

---

### 15.3.5 MLflow as OpenTelemetry Backend

**MLflow** can serve as a full-featured OpenTelemetry backend since version 3.6.0, making it a compelling choice for teams already using MLflow.

**OTLP Endpoint Support:**
- Exposes OTLP endpoint at `/v1/traces`
- Accepts traces from any OTel-compatible language (Java, Go, Rust, etc.)
- Supports gRPC and HTTP/protobuf

**OpenTelemetry Metrics Export:**
- Exports span-level statistics as OpenTelemetry metrics
- Compatible with Prometheus, Datadog, Grafana, and other metrics backends

**Span-Level Statistics:**
- Detailed performance metrics per span type
- Token usage tracking
- Latency distribution analysis

**Full OTel Compatibility:**
- Dual export to MLflow and other backends simultaneously
- Unified traces combining MLflow SDK and third-party OTel instrumentation
- Vendor-neutral approach

---

## Key Takeaways

1. **Choose frameworks based on focus area**: DeepEval excels for agent-specific metrics, RAGAS for RAG systems, OpenAI Evals for general LLM testing

2. **Instrumentation standardization**: OpenLLMetry has emerged as the de facto standard for LLM-specific instrumentation on top of OpenTelemetry

3. **Backend flexibility matters**: General-purpose OTel backends like SigNoz and Jaeger provide scalability and cost control for teams with custom needs

4. **MLflow evolution**: Full OpenTelemetry support transforms MLflow from an ML lifecycle tool to a comprehensive GenAI observability platform

5. **Framework-specific vs. general**: LLM-specific platforms add valuable features (cost tracking, prompt management) but general backends offer more flexibility

6. **Red teaming integration**: DeepEval's built-in red teaming capabilities (40+ vulnerabilities) make it valuable for security-conscious teams

---

## References

1. OpenAI. (2025). *How Evals Drive the Next Chapter in AI for Businesses*. [https://openai.com/index/evals-drive-next-chapter-of-ai/](https://openai.com/index/evals-drive-next-chapter-of-ai/)

2. OpenAI. (2025). *Working with Evals*. [https://platform.openai.com/docs/guides/evals](https://platform.openai.com/docs/guides/evals)

3. Confident AI. (2025). *DeepEval: The LLM Evaluation Framework*. [https://deepeval.com/docs/getting-started](https://deepeval.com/docs/getting-started)

4. Confident AI. (2025). *AI Agent Evaluation Metrics*. [https://deepeval.com/guides/guides-ai-agent-evaluation-metrics](https://deepeval.com/guides/guides-ai-agent-evaluation-metrics)

5. RAGAS. (2025). *Ragas: Automated Evaluation of Retrieval Augmented Generation*. [https://docs.ragas.io/en/stable/](https://docs.ragas.io/en/stable/)

6. Maxim AI. (2026). *Top 5 RAG Evaluation Platforms in 2026*. [https://www.getmaxim.ai/articles/top-5-rag-evaluation-platforms-in-2026/](https://www.getmaxim.ai/articles/top-5-rag-evaluation-platforms-in-2026/)

7. Traceloop. (2025). *OpenLLMetry: Open-Source Observability for GenAI*. [https://github.com/traceloop/openllmetry](https://github.com/traceloop/openllmetry)

8. OpenLIT. (2025). *OpenLIT: OpenTelemetry-native GenAI Observability*. [https://github.com/openlit/openlit](https://github.com/openlit/openlit)

9. Arize AI. (2025). *OpenInference*. [https://github.com/Arize-ai/openinference](https://github.com/Arize-ai/openinference)

10. Jaeger. (2025). *Jaeger: Open Source Distributed Tracing*. [https://www.jaegertracing.io/](https://www.jaegertracing.io/)

11. SigNoz. (2025). *SigNoz: Open Source Observability Platform*. [https://signoz.io/](https://signoz.io/)

12. Grafana. (2025). *Grafana Tempo*. [https://grafana.com/oss/tempo/](https://grafana.com/oss/tempo/)

13. Davies, D. (2026). *The Best LLM Evaluation Tools of 2026*. Medium. [https://medium.com/online-inference/the-best-llm-evaluation-tools-of-2026-40fd9b654dce](https://medium.com/online-inference/the-best-llm-evaluation-tools-of-2026-40fd9b654dce)

---

**Navigation:**
← [Previous Section: Observability and Tracing Platforms](14_observability_and_tracing_platforms.md) | [Table of Contents](../README.md) | [Next Section: Cloud Provider Evaluation Platforms](16_cloud_provider_evaluation_platforms.md) →
