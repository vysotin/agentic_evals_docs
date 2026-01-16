# Appendix E: Additional Resources and References

> **Appendices**
> **Reference Document**

## Overview

This appendix consolidates additional resources for AI agent evaluation, including academic papers, industry reports, official documentation, and community resources not fully covered in the main sections.

---

## Table of Contents

- [E.1 Academic Papers](#e1-academic-papers)
- [E.2 Industry Reports](#e2-industry-reports)
- [E.3 Official Documentation](#e3-official-documentation)
- [E.4 GitHub Repositories](#e4-github-repositories)
- [E.5 Online Courses and Tutorials](#e5-online-courses-and-tutorials)
- [E.6 Community Resources](#e6-community-resources)
- [E.7 Blogs and Publications](#e7-blogs-and-publications)

---

## E.1 Academic Papers

### E.1.1 Foundational Papers

| Paper | Authors | Year | Link |
|-------|---------|------|------|
| *AgentBench: Evaluating LLMs as Agents* | Liu et al. | 2023 | [arXiv](https://arxiv.org/abs/2308.03688) |
| *GAIA: A Benchmark for General AI Assistants* | Mialon et al. | 2023 | [arXiv](https://arxiv.org/abs/2311.12983) |
| *RAGAS: Automated Evaluation of Retrieval Augmented Generation* | Es et al. | 2023 | [arXiv](https://arxiv.org/abs/2309.15217) |
| *ReAct: Synergizing Reasoning and Acting in Language Models* | Yao et al. | 2022 | [arXiv](https://arxiv.org/abs/2210.03629) |

### E.1.2 Evaluation Frameworks (2024-2025)

| Paper | Authors | Year | Focus |
|-------|---------|------|-------|
| *Beyond Task Completion: An Assessment Framework for Evaluating Agentic AI Systems* | Akshathala et al. | 2025 | Four-pillar assessment |
| *Evaluating Agentic AI Systems: A Balanced Framework* | Shukla | 2025 | Five-axis evaluation |
| *AgentAuditor: LLM-as-Judge for Agent Safety* | Various | 2025 | Safety evaluation |
| *Evaluation and Benchmarking of LLM Agents: A Survey* | KDD 2025 | 2025 | Comprehensive taxonomy |

### E.1.3 Security Research

| Paper | Authors | Year | Focus |
|-------|---------|------|-------|
| *AgentDojo: A Benchmark for Prompt Injection Robustness* | ETH Zurich | 2024 | Prompt injection |
| *RAS-Eval: Real-World Agent Security Evaluation* | Various | 2025 | Tool execution security |
| *AgentHarm: Evaluating Harmful Use Prevention* | Various | 2025 | Malicious use prevention |
| *Not What You've Signed Up For: Compromising Real-World LLM-Integrated Applications* | Greshake et al. | 2023 | Indirect injection |

### E.1.4 LLM-as-Judge Research

| Paper | Authors | Year | Focus |
|-------|---------|------|-------|
| *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena* | Zheng et al. | 2023 | Judge evaluation |
| *GPT-4 Is a Good Judge, But a Biased One* | Jung et al. | 2024 | Bias analysis |
| *Can Large Language Models Be an Alternative to Human Evaluations?* | Chen et al. | 2023 | Human comparison |

---

## E.2 Industry Reports

### E.2.1 State of the Industry

| Report | Publisher | Year | Link |
|--------|-----------|------|------|
| *State of AI Agents* | LangChain | 2026 | [langchain.com](https://www.langchain.com/state-of-agent-engineering) |
| *AI Agent Trends 2026* | Google Cloud | 2026 | [cloud.google.com](https://cloud.google.com/resources/content/ai-agent-trends-2026) |
| *Enterprise AI Agents Report* | G2 | 2026 | [learn.g2.com](https://learn.g2.com/enterprise-ai-agents-report) |
| *Agentic AI Strategy* | Deloitte Tech Trends | 2026 | [deloitte.com](https://www.deloitte.com/insights) |
| *7 Agentic AI Trends to Watch* | Machine Learning Mastery | 2026 | [machinelearningmastery.com](https://machinelearningmastery.com/7-agentic-ai-trends-to-watch-in-2026/) |

### E.2.2 Analyst Predictions

| Report | Publisher | Year | Key Prediction |
|--------|-----------|------|----------------|
| *Gartner Predicts 2026* | Gartner | 2025 | 40% of enterprise apps with AI agents |
| *Strategic Predictions for 2026* | Gartner | 2025 | $58B market shake-up |
| *Forrester AI Predictions* | Forrester | 2026 | Evaluation as competitive advantage |
| *IDC AI Market Forecast* | IDC | 2026 | Global AI investment trends |

---

## E.3 Official Documentation

### E.3.1 Evaluation Frameworks

| Tool | Documentation | Key Resources |
|------|---------------|---------------|
| **DeepEval** | [docs.confident-ai.com](https://docs.confident-ai.com) | Agent metrics, custom metrics |
| **RAGAS** | [docs.ragas.io](https://docs.ragas.io) | RAG evaluation guide |
| **OpenAI Evals** | [github.com/openai/evals](https://github.com/openai/evals) | Eval templates |
| **TruLens** | [trulens.org/docs](https://www.trulens.org/docs) | RAG Triad |
| **Evidently** | [docs.evidentlyai.com](https://docs.evidentlyai.com) | ML monitoring |

### E.3.2 Observability Platforms

| Platform | Documentation | Key Resources |
|----------|---------------|---------------|
| **Langfuse** | [langfuse.com/docs](https://langfuse.com/docs) | Tracing, evaluation |
| **Arize Phoenix** | [docs.arize.com/phoenix](https://docs.arize.com/phoenix) | LLM observability |
| **OpenLLMetry** | [github.com/traceloop/openllmetry](https://github.com/traceloop/openllmetry) | OTel instrumentation |
| **OpenTelemetry GenAI** | [opentelemetry.io/docs/specs/semconv/gen-ai](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | Semantic conventions |

### E.3.3 Cloud Providers

| Provider | Documentation | Key Resources |
|----------|---------------|---------------|
| **Azure AI Foundry** | [learn.microsoft.com/azure/ai-studio](https://learn.microsoft.com/azure/ai-studio) | Agent evaluation |
| **Google Vertex AI** | [cloud.google.com/vertex-ai/docs](https://cloud.google.com/vertex-ai/docs) | Rapid Eval |
| **AWS Bedrock** | [docs.aws.amazon.com/bedrock](https://docs.aws.amazon.com/bedrock) | Model evaluation |

### E.3.4 Regulatory Frameworks

| Framework | Documentation | Key Resources |
|-----------|---------------|---------------|
| **EU AI Act** | [artificialintelligenceact.eu](https://artificialintelligenceact.eu) | Full text, compliance |
| **NIST AI RMF** | [nist.gov/itl/ai-risk-management-framework](https://www.nist.gov/itl/ai-risk-management-framework) | Risk management |
| **ISO/IEC 42001** | [iso.org](https://www.iso.org) | AI management system |

---

## E.4 GitHub Repositories

### E.4.1 Evaluation Frameworks

| Repository | Stars | Description |
|------------|-------|-------------|
| [openai/evals](https://github.com/openai/evals) | 15K+ | OpenAI's evaluation framework |
| [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | 5K+ | Agent evaluation |
| [explodinggradients/ragas](https://github.com/explodinggradients/ragas) | 7K+ | RAG evaluation |
| [truera/trulens](https://github.com/truera/trulens) | 3K+ | Feedback functions |
| [evidentlyai/evidently](https://github.com/evidentlyai/evidently) | 7K+ | ML monitoring |

### E.4.2 Observability

| Repository | Stars | Description |
|------------|-------|-------------|
| [langfuse/langfuse](https://github.com/langfuse/langfuse) | 20K+ | Open-source LLM observability |
| [Arize-ai/phoenix](https://github.com/Arize-ai/phoenix) | 8K+ | AI observability |
| [traceloop/openllmetry](https://github.com/traceloop/openllmetry) | 6K+ | OTel for LLMs |
| [mlflow/mlflow](https://github.com/mlflow/mlflow) | 23K+ | ML lifecycle |

### E.4.3 Agent Frameworks

| Repository | Stars | Description |
|------------|-------|-------------|
| [langchain-ai/langchain](https://github.com/langchain-ai/langchain) | 85K+ | LLM framework |
| [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | 5K+ | Stateful agents |
| [microsoft/autogen](https://github.com/microsoft/autogen) | 30K+ | Multi-agent |
| [run-llama/llama_index](https://github.com/run-llama/llama_index) | 30K+ | Data framework |

### E.4.4 Benchmarks

| Repository | Stars | Description |
|------------|-------|-------------|
| [THUDM/AgentBench](https://github.com/THUDM/AgentBench) | 2K+ | Agent benchmark |
| [ethz-spylab/agentdojo](https://github.com/ethz-spylab/agentdojo) | 500+ | Security benchmark |
| [princeton-nlp/SWE-bench](https://github.com/princeton-nlp/SWE-bench) | 2K+ | Coding benchmark |

---

## E.5 Online Courses and Tutorials

### E.5.1 University Courses

| Course | Institution | Platform | Focus |
|--------|-------------|----------|-------|
| *Agentic AI and AI Agents* | UC Berkeley | edX | Agent foundations |
| *Full Stack LLM Bootcamp* | FSDL | Self-hosted | LLM applications |
| *Deep Learning Specialization* | Deeplearning.AI | Coursera | ML fundamentals |

### E.5.2 Vendor Training

| Course | Provider | Focus |
|--------|----------|-------|
| Azure AI Fundamentals | Microsoft Learn | Azure AI services |
| Google Cloud AI | Google Cloud Skills | Vertex AI |
| AWS AI/ML Training | AWS Training | Bedrock, SageMaker |

### E.5.3 Tutorials and Guides

| Resource | Provider | Topic |
|----------|----------|-------|
| DeepEval Getting Started | Confident AI | Agent evaluation |
| RAGAS Tutorials | Explodinggradients | RAG evaluation |
| Langfuse Cookbook | Langfuse | Observability recipes |
| OpenTelemetry GenAI Guide | CNCF | Instrumentation |

---

## E.6 Community Resources

### E.6.1 Slack/Discord Communities

| Community | Platform | Focus | Size |
|-----------|----------|-------|------|
| LangChain | Slack | Agent development | 50K+ |
| MLOps Community | Slack | ML operations | 20K+ |
| Hugging Face | Discord | ML/AI | 100K+ |
| Weights & Biases | Discord | Experiment tracking | 15K+ |

### E.6.2 Forums and Q&A

| Forum | URL | Best For |
|-------|-----|----------|
| LangChain Forum | langchain.dev | LangChain questions |
| Stack Overflow | stackoverflow.com | Technical issues |
| Reddit r/MachineLearning | reddit.com | Research discussion |
| Reddit r/LocalLLaMA | reddit.com | Open-source LLMs |

### E.6.3 Conferences

| Conference | Focus | When |
|------------|-------|------|
| NeurIPS | ML research | December |
| ICML | ML theory | Summer |
| ACL/EMNLP | NLP | Varies |
| Interrupt | Agent development | Annual |
| ODSC | Applied data science | Multiple |

---

## E.7 Blogs and Publications

### E.7.1 Vendor Blogs

| Blog | Organization | Key Topics |
|------|--------------|------------|
| [Galileo AI Blog](https://galileo.ai/blog) | Galileo | Agent metrics |
| [Arize Blog](https://arize.com/blog) | Arize AI | ML observability |
| [Langfuse Blog](https://langfuse.com/blog) | Langfuse | Open-source tracing |
| [LangChain Blog](https://blog.langchain.com) | LangChain | Agent development |
| [Anthropic Research](https://anthropic.com/research) | Anthropic | Safety research |
| [OpenAI Research](https://openai.com/research) | OpenAI | Model capabilities |

### E.7.2 Independent Publications

| Publication | Focus |
|-------------|-------|
| The Gradient | ML research summaries |
| Import AI | Weekly AI newsletter |
| AI Snake Oil | Critical AI analysis |
| Simon Willison's Weblog | LLM developments |
| Machine Learning Mastery | Tutorials |

### E.7.3 Newsletters

| Newsletter | Author/Org | Frequency |
|------------|------------|-----------|
| The Batch | deeplearning.ai | Weekly |
| Import AI | Jack Clark | Weekly |
| AI Weekly | Best | Weekly |
| Last Week in AI | Various | Weekly |

---

## Quick Reference Links

### Essential Documentation

- **Evaluation**: [DeepEval](https://docs.confident-ai.com) | [RAGAS](https://docs.ragas.io) | [Langfuse](https://langfuse.com/docs)
- **Observability**: [OpenTelemetry GenAI](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | [OpenLLMetry](https://github.com/traceloop/openllmetry)
- **Cloud**: [Azure AI](https://learn.microsoft.com/azure/ai-studio) | [Vertex AI](https://cloud.google.com/vertex-ai/docs) | [Bedrock](https://docs.aws.amazon.com/bedrock)

### Key Research Papers

- **Benchmarks**: [AgentBench](https://arxiv.org/abs/2308.03688) | [GAIA](https://arxiv.org/abs/2311.12983)
- **Security**: [AgentDojo](https://arxiv.org/abs/2406.13352) | [Indirect Injection](https://arxiv.org/abs/2302.12173)
- **Evaluation**: [LLM-as-Judge](https://arxiv.org/abs/2306.05685) | [RAGAS](https://arxiv.org/abs/2309.15217)

### Industry Reports

- [State of AI Agents (LangChain)](https://www.langchain.com/state-of-agent-engineering)
- [Gartner AI Predictions](https://www.gartner.com/en/articles/strategic-predictions-for-2026)
- [EU AI Act](https://artificialintelligenceact.eu)

---

**Navigation:**
← [Previous Appendix: Glossary of Terms](appendix_d_glossary.md) | [Table of Contents](../../README.md) | [Next Appendix: Cloud Provider Feature Comparison Matrix](appendix_f_cloud_provider_comparison.md) →
