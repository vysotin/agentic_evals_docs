# Appendix F: Glossary of Terms

> **Appendices**
> **Reference Document**

## Overview

This glossary provides definitions for key terms used throughout this guide. Terms are organized alphabetically with cross-references to related concepts.

---

## A

**A/B Testing**
A method of comparing two versions of an agent or prompt by randomly assigning users to each version and measuring performance differences.

**Agent**
An AI system that can perceive its environment, make decisions, and take actions autonomously to achieve specified goals. In the context of LLMs, agents typically combine language models with tools, memory, and planning capabilities.

**Agent Flow**
A Galileo AI metric measuring how well an agent navigates through multi-step workflows, including smoothness of transitions and absence of backtracking.

**AgentBench**
A comprehensive benchmark for evaluating LLM agents across multiple environments including operating systems, databases, and web interactions.

**AgentDojo**
A benchmark developed by ETH Zurich for evaluating agent robustness against prompt injection attacks, featuring 97 tasks and 629 security test cases.

**AgentHarm**
A benchmark for evaluating AI agent resistance to malicious use, measuring both harm prevention and task completion.

**Agentic AI**
AI systems capable of autonomous decision-making, planning, and action execution with minimal human intervention.

**AI Agent**
See *Agent*.

**AI RMF (AI Risk Management Framework)**
NIST's framework for managing AI risks through four functions: GOVERN, MAP, MEASURE, and MANAGE.

---

## B

**Backtracking**
When an agent returns to a previous step in its workflow, often indicating poor planning or execution failures.

**Benchmark**
A standardized test or set of tests used to evaluate AI system performance against established criteria.

**Bias**
Systematic errors in AI outputs that unfairly favor or disfavor certain groups, inputs, or outcomes.

---

## C

**Chain-of-Thought (CoT)**
A prompting technique where the model is encouraged to show its reasoning steps, improving performance on complex tasks.

**Completion Rate**
The percentage of started interactions that reach a defined endpoint.

**Containment Rate**
The percentage of customer interactions resolved entirely by an AI agent without human escalation. Key metric for customer service ROI.

**Context Precision**
RAG metric measuring the proportion of retrieved context that is relevant to answering the query.

**Context Recall**
RAG metric measuring the proportion of relevant information from the ground truth that appears in the retrieved context.

**Context Window**
The maximum amount of text (measured in tokens) that an LLM can process in a single request.

---

## D

**DeepEval**
An open-source evaluation framework specifically designed for AI agent evaluation, featuring agent-specific metrics like TaskCompletionMetric and ToolCorrectnessMetric.

**Distribution Drift**
Changes in the statistical distribution of input data over time, which can degrade model performance.

**Distribution Mismatch**
The gap between the distribution of data used in evaluation and the distribution encountered in production.

---

## E

**Embedding**
A numerical representation of text (or other data) in a high-dimensional vector space, used for semantic similarity comparison.

**Error Rate**
The percentage of agent executions that result in errors, failures, or exceptions.

**EU AI Act**
The European Union's comprehensive regulatory framework for AI, with enforcement beginning in August 2026 for high-risk systems.

**Evaluation-Driven Development (EDD)**
A development methodology that prioritizes evaluation throughout the agent development lifecycle, similar to Test-Driven Development.

---

## F

**Faithfulness**
RAG metric measuring how well a generated response is grounded in the provided context, without hallucination.

**Few-Shot Learning**
Providing a model with a small number of examples to guide its behavior on a task.

**First Contact Resolution (FCR)**
The percentage of customer issues resolved in a single interaction without follow-ups or escalations.

**Forensic Loop**
A continuous improvement process: production failures → trace capture → root cause analysis → new test cases → fixes → validation.

---

## G

**GAIA**
A benchmark for General AI Assistants, structured in three difficulty levels testing progressive complexity in tool use and reasoning.

**Galileo AI**
A platform providing specialized agent metrics including Agent Flow, Agent Efficiency, Conversation Quality, and Intent Change.

**Golden Dataset**
A curated set of high-quality test cases with verified ground truth, used as the standard for evaluation.

**GPAI (General Purpose AI)**
Under the EU AI Act, AI models trained with large amounts of data using self-supervision and capable of a wide range of tasks.

**Groundedness**
See *Faithfulness*.

**Guardrails**
Safety mechanisms that constrain agent behavior within acceptable boundaries, preventing harmful or unintended actions.

---

## H

**Hallucination**
When an AI model generates information that is not grounded in its training data or provided context, producing false or fabricated content.

**HarmScore**
A metric measuring the severity and frequency of harmful content in agent outputs.

**Human-in-the-Loop (HITL)**
A design pattern where human oversight is integrated into AI workflows for validation, correction, or decision-making.

---

## I

**Indirect Prompt Injection**
An attack where malicious instructions are embedded in content that the agent retrieves or processes, rather than directly in user input.

**Intent Change**
A Galileo AI metric detecting when users shift their goals mid-conversation, often indicating agent failure to address initial needs.

**Intent Fulfillment**
Whether the agent addressed the user's underlying intent, beyond just completing literal steps.

---

## J

**Jailbreak**
An attempt to bypass an AI model's safety constraints through specially crafted prompts.

---

## K

**KL Divergence (Kullback-Leibler Divergence)**
A statistical measure of how one probability distribution differs from another, used to detect distribution drift.

---

## L

**LangChain**
A popular framework for building LLM applications, providing tools for prompt management, chains, and agent orchestration.

**Langfuse**
An open-source LLM observability platform providing tracing, evaluation, and prompt management capabilities.

**LangGraph**
LangChain's framework for building stateful, multi-actor applications with LLMs.

**LangSmith**
LangChain's commercial platform for LLM application development, debugging, and evaluation.

**Latency**
The time delay between a request and its response, typically measured in milliseconds.

**LLM (Large Language Model)**
A neural network model trained on large amounts of text data, capable of generating and understanding natural language.

**LLM-as-Judge**
Using an LLM to evaluate the outputs of another AI system, providing scalable quality assessment.

---

## M

**MCP (Model Context Protocol)**
Anthropic's protocol standardizing how AI agents connect to external tools, databases, and APIs.

**Mechanistic Interpretability**
Research techniques that peer directly into neural network internals to understand how models process information.

**Metric**
A quantifiable measure used to evaluate agent performance on a specific dimension.

**MLflow**
An open-source platform for the complete machine learning lifecycle, including experimentation, deployment, and model registry.

**Multi-Agent System**
An architecture where multiple specialized agents collaborate to complete complex tasks.

---

## N

**NIST (National Institute of Standards and Technology)**
U.S. government agency that develops standards and guidelines, including the AI Risk Management Framework.

---

## O

**Observability**
The ability to understand the internal state of a system from its external outputs, typically through traces, metrics, and logs.

**Online Evaluation**
Evaluation performed on live production traffic, as opposed to offline evaluation on test datasets.

**OpenLLMetry**
A set of OpenTelemetry extensions specifically designed for LLM application tracing and monitoring.

**OpenTelemetry**
A CNCF project providing vendor-neutral APIs, SDKs, and tools for generating and collecting telemetry data.

**OTLP (OpenTelemetry Protocol)**
The standard protocol for transmitting telemetry data to OpenTelemetry backends.

---

## P

**P50, P95, P99**
Percentile metrics for latency or other measurements. P95 means 95% of requests are faster than this value.

**PII (Personally Identifiable Information)**
Information that can identify an individual, such as names, addresses, phone numbers, and social security numbers.

**Plan Adherence**
Metric measuring how faithfully an agent follows its generated plan during execution.

**Plan Quality**
Metric assessing whether an agent's plan is logical, complete, and efficient for the given task.

**Prompt Injection**
An attack where malicious instructions are inserted into prompts to manipulate AI behavior.

**PSI (Population Stability Index)**
A metric for measuring changes in population distributions over time, used for drift detection.

---

## Q

**Quality Gate**
A checkpoint in development where code must pass defined quality criteria before proceeding.

---

## R

**RAG (Retrieval-Augmented Generation)**
An architecture that combines information retrieval with text generation, grounding LLM outputs in retrieved context.

**RAGAS**
An open-source framework specifically designed for evaluating RAG systems.

**Red-Teaming**
Adversarial testing where evaluators attempt to find vulnerabilities or failure modes in AI systems.

**RefusalRate**
The percentage of harmful or inappropriate requests that an agent correctly refuses to fulfill.

**Regression Testing**
Testing to ensure that new changes haven't broken previously working functionality.

---

## S

**Safety Envelope**
The boundaries within which agent behavior is considered safe and acceptable.

**Semantic Conventions**
Standardized attribute names and values used in telemetry data, enabling interoperability.

**Span**
In distributed tracing, a unit of work with a start time, duration, and associated metadata.

**SWE-Bench**
A benchmark for evaluating AI coding assistants on real-world software engineering tasks.

**Systemic Risk**
Under the EU AI Act, risks posed by GPAI models with particularly high capabilities or widespread deployment.

---

## T

**Task Completion Rate**
See *Task Success Rate*.

**Task Success Rate**
The percentage of user requests that an agent successfully completes end-to-end.

**Token**
The basic unit of text processing for LLMs, typically representing a word or word part.

**Tool**
An external capability that an agent can invoke, such as web search, database queries, or API calls.

**Tool Correctness**
Metric measuring whether an agent selected and executed tools correctly with proper parameters.

**Trace**
A record of the execution path through a system, capturing spans, events, and metadata.

**TTFT (Time to First Token)**
The latency from request submission to receiving the first response token.

**TruLens**
An open-source framework for evaluating and tracking LLM applications, known for its "RAG Triad" evaluation.

---

## U

**Unit Evaluation**
Testing individual components (prompts, tools) in isolation before integration testing.

---

## V

**Vector Database**
A database optimized for storing and querying high-dimensional embeddings, commonly used in RAG systems.

**Verbosity Bias**
The tendency for LLM judges to prefer longer responses regardless of quality.

---

## W

**Workflow**
A sequence of steps or actions that an agent executes to complete a task.

---

## X

**XAI (Explainable AI)**
AI systems designed to provide human-understandable explanations for their decisions and outputs.

---

## Z

**Zero-Shot**
Performing a task without any task-specific examples, relying only on the model's general capabilities.

---

## Cross-Reference Index

| Term | Related Terms |
|------|---------------|
| Agent | Agentic AI, Multi-Agent System, Tool |
| Evaluation | Benchmark, Metric, LLM-as-Judge |
| Observability | Trace, Span, OpenTelemetry |
| RAG | Faithfulness, Context Precision, Vector Database |
| Safety | Guardrails, Red-Teaming, Jailbreak |
| Security | Prompt Injection, PII, Adversarial |

---

**Navigation:**
← [Previous Appendix: Code Examples](appendix_e_code_examples.md) | [Table of Contents](../../README.md) | [Next Appendix: Additional Resources and References](appendix_g_additional_resources.md) →
