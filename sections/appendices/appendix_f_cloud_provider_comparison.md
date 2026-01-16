# Appendix F: Cloud Provider Feature Comparison Matrix

> **Appendices**
> **Reference Document**

## Overview

This appendix provides detailed feature comparison matrices for AI agent evaluation capabilities across major cloud providers: Microsoft Azure AI Foundry, Google Cloud Vertex AI, and Amazon Web Services Bedrock. Use these matrices to select the right platform for your evaluation needs.

---

## Table of Contents

- [F.1 Overall Comparison](#f1-overall-comparison)
- [F.2 Evaluation Capabilities](#f2-evaluation-capabilities)
- [F.3 Observability Features](#f3-observability-features)
- [F.4 Safety and Guardrails](#f4-safety-and-guardrails)
- [F.5 Pricing Comparison](#f5-pricing-comparison)
- [F.6 Selection Guide](#f6-selection-guide)

---

## F.1 Overall Comparison

### F.1.1 Platform Overview

| Feature | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|---------|-----------------|------------------|-------------|
| **Primary Focus** | Enterprise AI development | ML platform + GenAI | Foundation model access |
| **Agent Framework** | Azure AI Agent Service | Vertex AI Agent Builder | Bedrock Agents |
| **Evaluation Tool** | Azure AI Evaluation SDK | Rapid Eval / AutoSxS | Model Evaluation Jobs |
| **Observability** | Azure Monitor + App Insights | Cloud Logging + Trace | CloudWatch |
| **Safety Tools** | Azure AI Content Safety | Responsible AI Toolkit | Bedrock Guardrails |
| **Pricing Model** | Pay-per-use | Pay-per-use | Pay-per-use |
| **Free Tier** | Limited | Limited | Limited |

### F.1.2 Supported Models

| Model Provider | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|----------------|-----------------|------------------|-------------|
| **OpenAI** | GPT-4o, GPT-4, GPT-3.5 | Via Model Garden | No |
| **Anthropic** | No | Claude 3.5, Claude 3 | Claude 3.5, Claude 3 |
| **Google** | No | Gemini 2.0, PaLM | No |
| **Meta** | Llama 3.1, Llama 3 | Llama 3.1 | Llama 3.1, Llama 3 |
| **Mistral** | Mistral models | Mistral models | Mistral models |
| **Amazon** | No | No | Titan models |
| **Cohere** | Cohere models | Cohere models | Cohere models |

---

## F.2 Evaluation Capabilities

### F.2.1 Built-in Evaluators

| Evaluator | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|-----------|-----------------|------------------|-------------|
| **Coherence** | Native | Native (AutoSxS) | Via custom |
| **Relevance** | Native | Native | Via custom |
| **Fluency** | Native | Native | Via custom |
| **Groundedness** | Native | Native | Via custom |
| **Similarity** | Native | Native | Via custom |
| **Faithfulness** | Native | Native | Via custom |
| **Safety** | Native | Native | Native |
| **Toxicity** | Native | Native | Native |
| **Custom Rubrics** | Supported | Supported | Supported |
| **Code Quality** | Limited | Limited | Limited |

### F.2.2 Evaluation Methods

| Method | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|--------|-----------------|------------------|-------------|
| **LLM-as-Judge** | Yes (GPT-4) | Yes (Gemini/Claude) | Yes (Claude) |
| **Pairwise Comparison** | Manual | Native (AutoSxS) | Manual |
| **Human Evaluation** | Supported | Supported | Supported |
| **Automated Metrics** | Yes | Yes | Yes |
| **A/B Testing** | Via experiments | Native | Via experiments |
| **Batch Evaluation** | Yes | Yes | Yes |
| **Real-time Evaluation** | Yes | Yes | Limited |

### F.2.3 Agent-Specific Evaluation

| Capability | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|------------|-----------------|------------------|-------------|
| **Task Completion** | Custom metrics | Custom metrics | Custom metrics |
| **Tool Use Accuracy** | Via tracing | Via tracing | Via tracing |
| **Plan Quality** | Custom | Custom | Custom |
| **Multi-turn Eval** | Supported | Supported | Supported |
| **Workflow Testing** | Supported | Supported | Supported |
| **Trace Analysis** | Native | Native | Native |

### F.2.4 RAG-Specific Evaluation

| Metric | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|--------|-----------------|------------------|-------------|
| **Context Precision** | Custom | Custom | Custom |
| **Context Recall** | Custom | Custom | Custom |
| **Faithfulness** | Native | Native | Custom |
| **Answer Relevancy** | Native | Native | Custom |
| **Retrieval Quality** | Via integration | Native | Via integration |

---

## F.3 Observability Features

### F.3.1 Tracing Capabilities

| Feature | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|---------|-----------------|------------------|-------------|
| **Auto Instrumentation** | Yes | Yes | Yes |
| **Distributed Tracing** | Azure Monitor | Cloud Trace | X-Ray |
| **LLM Call Tracing** | Native | Native | Native |
| **Tool Call Tracing** | Native | Native | Native |
| **Custom Spans** | Supported | Supported | Supported |
| **Token Tracking** | Yes | Yes | Yes |
| **Cost Attribution** | Yes | Yes | Yes |
| **Latency Analysis** | Yes | Yes | Yes |

### F.3.2 Logging and Monitoring

| Feature | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|---------|-----------------|------------------|-------------|
| **Prompt Logging** | Optional | Optional | Optional |
| **Response Logging** | Optional | Optional | Optional |
| **Error Logging** | Native | Native | Native |
| **Metrics Dashboard** | Azure Portal | Cloud Console | CloudWatch |
| **Custom Dashboards** | Supported | Supported | Supported |
| **Alerting** | Native | Native | Native |
| **Anomaly Detection** | Limited | Limited | Limited |

### F.3.3 Integration Options

| Integration | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|-------------|-----------------|------------------|-------------|
| **OpenTelemetry** | Yes | Yes | Yes |
| **Langfuse** | Via OTLP | Via OTLP | Via OTLP |
| **LangSmith** | SDK | SDK | SDK |
| **Datadog** | Native | Native | Native |
| **Grafana** | Via Azure Monitor | Via export | Via CloudWatch |
| **Custom Exporters** | Supported | Supported | Supported |

---

## F.4 Safety and Guardrails

### F.4.1 Content Safety Features

| Feature | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|---------|-----------------|------------------|-------------|
| **Hate Speech Detection** | Native (4 severity) | Native | Native |
| **Violence Detection** | Native (4 severity) | Native | Native |
| **Sexual Content** | Native (4 severity) | Native | Native |
| **Self-Harm Content** | Native (4 severity) | Native | Native |
| **Custom Categories** | Blocklists | Limited | Topic policies |
| **Severity Levels** | 4 levels | 3 levels | 3 levels |
| **Real-time Filtering** | Yes | Yes | Yes |
| **Batch Analysis** | Yes | Yes | Yes |

### F.4.2 Guardrails Configuration

| Capability | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|------------|-----------------|------------------|-------------|
| **Input Filtering** | Yes | Yes | Yes |
| **Output Filtering** | Yes | Yes | Yes |
| **PII Detection** | Native | Via DLP | Native |
| **Custom Policies** | Yes | Yes | Yes |
| **Prompt Injection Defense** | Limited | Limited | Yes (88% block rate) |
| **Jailbreak Prevention** | Yes | Yes | Yes |
| **Topic Restrictions** | Blocklists | Limited | Topic filters |
| **Word Filters** | Yes | Yes | Yes |

### F.4.3 Compliance Features

| Feature | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|---------|-----------------|------------------|-------------|
| **GDPR Support** | Yes | Yes | Yes |
| **HIPAA Support** | Yes (BAA) | Yes (BAA) | Yes (BAA) |
| **SOC 2** | Yes | Yes | Yes |
| **FedRAMP** | High | Moderate | High |
| **Data Residency** | Regional | Regional | Regional |
| **Encryption** | At rest + transit | At rest + transit | At rest + transit |
| **Audit Logging** | Yes | Yes | Yes |

---

## F.5 Pricing Comparison

### F.5.1 Model Pricing (as of January 2026)

*Note: Prices are approximate and subject to change. Check provider documentation for current pricing.*

| Model | Azure (per 1M tokens) | Vertex AI (per 1M tokens) | Bedrock (per 1M tokens) |
|-------|----------------------|---------------------------|-------------------------|
| **GPT-4o (input)** | $2.50 | N/A | N/A |
| **GPT-4o (output)** | $10.00 | N/A | N/A |
| **Claude 3.5 Sonnet (input)** | N/A | $3.00 | $3.00 |
| **Claude 3.5 Sonnet (output)** | N/A | $15.00 | $15.00 |
| **Gemini 1.5 Pro (input)** | N/A | $3.50 | N/A |
| **Gemini 1.5 Pro (output)** | N/A | $10.50 | N/A |
| **Llama 3.1 70B (input)** | $0.27 | $0.27 | $0.27 |
| **Llama 3.1 70B (output)** | $0.35 | $0.35 | $0.35 |

### F.5.2 Evaluation Service Pricing

| Service | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|---------|-----------------|------------------|-------------|
| **LLM-as-Judge Evals** | Token cost only | Token cost only | Token cost only |
| **Built-in Evaluators** | Included | Included | Included |
| **Batch Evaluation** | Token cost | Token cost | Token cost |
| **Human Evaluation** | N/A | N/A | N/A |

### F.5.3 Observability Pricing

| Service | Azure | Google Cloud | AWS |
|---------|-------|--------------|-----|
| **Logging** | Azure Monitor pricing | Cloud Logging pricing | CloudWatch pricing |
| **Tracing** | Application Insights pricing | Cloud Trace pricing | X-Ray pricing |
| **Alerting** | Included | Included | Included |

---

## F.6 Selection Guide

### F.6.1 Decision Matrix

| If You Need... | Best Choice | Reason |
|----------------|-------------|--------|
| OpenAI models + enterprise | Azure AI Foundry | Native OpenAI integration |
| Google models (Gemini) | Vertex AI | Native Google models |
| Claude models + simplicity | AWS Bedrock | Strong Claude support, easy setup |
| Multi-provider flexibility | AWS Bedrock | Most model providers |
| Best pairwise comparison | Vertex AI | Native AutoSxS |
| Strongest content safety | Azure AI Foundry | Most granular safety controls |
| Best guardrails | AWS Bedrock | 88% prompt injection block rate |
| Existing AWS infrastructure | AWS Bedrock | Native integration |
| Existing Azure infrastructure | Azure AI Foundry | Native integration |
| Existing GCP infrastructure | Vertex AI | Native integration |

### F.6.2 Use Case Recommendations

| Use Case | Recommended Platform | Alternative |
|----------|---------------------|-------------|
| **Enterprise Customer Support** | Azure AI Foundry | Vertex AI |
| **Research/Analysis Agents** | Vertex AI (AutoSxS) | Azure |
| **Code Generation** | AWS Bedrock (Claude) | Azure (GPT-4) |
| **Healthcare/HIPAA** | Any (all have BAA) | - |
| **Multi-language Support** | Vertex AI | Azure |
| **Cost-Sensitive** | AWS Bedrock | Vertex AI |
| **Maximum Safety** | Azure AI Foundry | AWS Bedrock |

### F.6.3 Migration Considerations

| From | To | Complexity | Key Considerations |
|------|-----|------------|-------------------|
| Azure → AWS | Medium | Model availability, SDK changes |
| Azure → GCP | Medium | Evaluation method differences |
| AWS → Azure | Medium | Model migration, safety config |
| AWS → GCP | Low-Medium | Similar SDK patterns |
| GCP → Azure | Medium | AutoSxS alternative needed |
| GCP → AWS | Low-Medium | Similar architecture |

---

## Summary Comparison

| Category | Azure AI Foundry | Google Vertex AI | AWS Bedrock |
|----------|-----------------|------------------|-------------|
| **Overall Rating** | Enterprise-focused | ML-platform integrated | Model-access focused |
| **Evaluation Strength** | Comprehensive SDK | AutoSxS pairwise | Simple job-based |
| **Observability** | Strong | Strong | Good |
| **Safety** | Excellent | Good | Very Good |
| **Ease of Use** | Medium | Medium | Easy |
| **Model Selection** | OpenAI-focused | Google-focused | Multi-provider |
| **Best For** | Azure shops | GCP shops | AWS shops |

---

**Navigation:**
← [Previous Appendix: Additional Resources and References](appendix_e_additional_resources.md) | [Table of Contents](../../README.md) | [Next Appendix: Security Benchmark Comparison](appendix_g_security_benchmark_comparison.md) →
