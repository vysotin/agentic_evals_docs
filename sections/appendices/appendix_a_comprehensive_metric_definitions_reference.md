# Appendix A: Comprehensive Metric Definitions Reference

> **Appendices**
> **Reference Document**

## Overview

This appendix provides a comprehensive reference for all 80+ evaluation metrics covered in this guide. Each metric includes a concise definition, measurement formula (where applicable), and typical industry benchmarks from 2026 deployments. Use this as a quick-reference guide when designing evaluation portfolios or implementing specific metrics.

---

## Table of Contents

- [A.1 Task Completion and Success Metrics](#a1-task-completion-and-success-metrics)
- [A.2 Process Quality Metrics](#a2-process-quality-metrics)
- [A.3 Tool and Action Metrics](#a3-tool-and-action-metrics)
- [A.4 Outcome Quality Metrics](#a4-outcome-quality-metrics)
- [A.5 Performance and Efficiency Metrics](#a5-performance-and-efficiency-metrics)
- [A.6 Content Safety Metrics](#a6-content-safety-metrics)
- [A.7 Security Metrics](#a7-security-metrics)
- [A.8 Advanced Agent Metrics](#a8-advanced-agent-metrics)
- [A.9 Reasoning and Planning Metrics](#a9-reasoning-and-planning-metrics)
- [A.10 Interaction Quality Metrics](#a10-interaction-quality-metrics)
- [A.11 Robustness Metrics](#a11-robustness-metrics)
- [A.12 Business Metrics](#a12-business-metrics)
- [A.13 Drift and Distribution Metrics](#a13-drift-and-distribution-metrics)
- [A.14 Domain-Specific Metrics](#a14-domain-specific-metrics)

---

## A.1 Task Completion and Success Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Task Success Rate** | Percentage of user requests successfully completed end-to-end | `(Successful Tasks / Total Tasks) × 100` | 75-90% depending on domain |
| **Task Resolution** | Whether agent fully resolved user's underlying need | LLM-judge assessment | 85-90% for high performers |
| **Error Rate** | Percentage of runs resulting in errors or failures | `(Failed Executions / Total Executions) × 100` | <5% production; <1% critical systems |
| **Containment Rate** | Interactions resolved without human escalation | `(Automated Resolutions / Total Interactions) × 100` | 70-90% (top KPI for ROI) |
| **Completion Rate** | Started interactions that reach a defined endpoint | `(Completed Sessions / Started Sessions) × 100` | 75-90% |
| **First Contact Resolution (FCR)** | Issues resolved in single interaction | Track re-contacts within 7 days | 75-90% |

---

## A.2 Process Quality Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Plan Quality** | Logical, complete, and efficient planning | LLM-judge score (0-1 scale) | >0.7 threshold |
| **Plan Coherence** | Logical flow without off-topic steps | LLM analysis of reasoning trace | >4.0 avg (1-5 scale) |
| **Plan Adherence** | Faithfulness to intended steps | `(Planned Steps Executed / Total Planned) × 100` | >85% |
| **Step-wise Contribution** | Value added by each reasoning step | Identify redundant/dead-end steps | <10% wasted steps |
| **Step-Level Accuracy** | Correctness of individual steps | Per-step verification | >95% |
| **Instruction Adherence** | Following system prompts and constraints | Programmatic constraint checking | >98% |

---

## A.3 Tool and Action Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Tool Use Accuracy** | Correct tool selection and execution | `(Correct Tool Uses / Total Tool Calls) × 100` | >98% |
| **Tool Correctness** | Proper parameters and successful execution | Binary per-call + aggregation | >95% |
| **Tool Selection Accuracy** | Choosing optimal tool for task | Compare selected vs. optimal | >90% |
| **Tool Call Frequency** | Number of tool calls per task | Count per task type | Task-dependent |
| **Tool Usage Patterns** | Distribution of tool usage | Analyze call sequences | No specific threshold |

---

## A.4 Outcome Quality Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Factual Correctness** | Accuracy of facts in responses | Verification against ground truth | >99% for critical apps |
| **Intent Fulfillment** | Addressing user's true underlying intent | LLM-judge semantic assessment | >90% |
| **Answer Completeness** | Addressing all aspects of query | Rubric-based or LLM assessment | >85% |
| **Contextual Relevancy** | Relevance to user query and context | Similarity scoring | >0.8 (0-1 scale) |
| **Groundedness** | Responses grounded in provided context | Citation/attribution analysis | >95% |
| **Response Quality** | Overall quality of generated output | Multi-criteria LLM judge | >4.0 (1-5 scale) |

---

## A.5 Performance and Efficiency Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **End-to-End Latency** | Total time from request to response | Timestamp difference | <5s typical; <1s voice |
| **Time-to-First-Token (TTFT)** | Time until first response token | Timestamp of first token | <500ms |
| **Response Time** | Average latency across requests | Mean of latencies | <2s typical |
| **P50 Latency** | Median latency | 50th percentile | <500ms for queries |
| **P95 Latency** | 95th percentile latency | 95th percentile | <1000ms for queries |
| **P99 Latency** | 99th percentile latency | 99th percentile | <2000ms |
| **Execution Time per Agent** | Time spent in each agent component | Per-component timing | Varies by architecture |
| **Token Usage** | Total tokens consumed | Sum of input + output tokens | Task-dependent |
| **Token Cost per Task** | Dollar cost of tokens per task | `Tokens × Price per Token` | Monitor trends |
| **Tokens per Second** | Processing throughput | `Tokens / Time` | Model-dependent |
| **Context Utilization** | Effective use of context window | `Used Context / Available Context` | 60-80% optimal |
| **Tool Call Count** | Number of tools invoked | Count per session | Minimize redundant calls |
| **Agent Efficiency Score** | Composite efficiency measure | `Optimal Resources / Actual Resources` | >0.65 acceptable |
| **Step Utility** | Value contribution per step | LLM assessment per step | >80% useful steps |
| **Cost per Interaction** | Total cost per user interaction | Sum all costs | Track and optimize |
| **Cost per Successful Completion** | Cost for successful outcomes only | `Total Cost / Successful Tasks` | Key ROI metric |

---

## A.6 Content Safety Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **PII Detection Rate** | Identification of personal information | `(True Positives / Total PII) × 100` | >99.9% |
| **Harmful Content Rate** | Responses containing harmful content | `(Harmful Outputs / Total Outputs) × 100` | <0.01% |
| **Toxicity Detection Rate** | Identification of toxic language | Detection precision/recall | >95% detection |
| **Bias Detection Score** | Identification of biased outputs | Disparity analysis across groups | <5% disparity |
| **Policy Compliance Rate** | Adherence to organizational policies | `(Compliant Outputs / Total) × 100` | >99% |
| **Content Filter Effectiveness** | Success of content filtering | Blocked/total harmful attempts | >88% (Bedrock benchmark) |

---

## A.7 Security Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Prompt Injection Resistance** | Defense against prompt injection attacks | Success rate of defense | <15% attack success |
| **Jailbreak Detection Rate** | Identification of jailbreak attempts | Detection accuracy | >95% |
| **Adversarial Robustness Score** | Performance under adversarial inputs | Performance drop under attack | <10% degradation |
| **Code Vulnerability Rate** | Security issues in generated code | Vulnerabilities per 1000 lines | <1 high-severity |
| **Data Poisoning Detection** | Identification of poisoned training data | Detection accuracy | Research frontier |
| **Security Violation Rate** | Unauthorized access or actions | `(Violations / Total Actions) × 100` | <0.1% |
| **Attack Success Rate** | Percentage of successful attacks | `(Successful Attacks / Total Attacks) × 100` | Target: <15% |
| **Task Completion Under Attack** | Performance during active attacks | Success rate under adversarial conditions | >65% |

---

## A.8 Advanced Agent Metrics

### Galileo AI Metrics (2025-2026)

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Agent Flow** | Navigation through multi-step workflows | Backtracking/loop analysis | >0.85 |
| **Agent Efficiency** | Resource usage relative to optimal | `Optimal / Actual Resources` | >0.65 |
| **Conversation Quality** | Holistic interaction quality | Multi-criteria scoring (1-5) | >4.0 |
| **Intent Change** | Detection of goal shifts | Intent classification per turn | <10% frustration pivots |

---

## A.9 Reasoning and Planning Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Reasoning Quality** | Soundness of logical reasoning | LLM-judge assessment | >4.0 (1-5 scale) |
| **Logical Consistency** | Absence of contradictions | Contradiction detection | >95% consistent |
| **Decision Quality** | Appropriateness of decisions | Outcome-based evaluation | Context-dependent |
| **Goal-Drift Score** | Deviation from original objective | Semantic similarity to goal | <0.2 drift |

---

## A.10 Interaction Quality Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Conversation Quality Score** | Overall interaction quality | Multi-criteria LLM assessment | >4.0 (1-5 scale) |
| **Avg Messages per Conversation** | Interaction length | Count messages per session | Task-dependent |
| **Turn-Taking Quality** | Natural conversation flow (voice) | Interrupt/overlap analysis | <5% inappropriate interrupts |
| **Response Tone** | Appropriateness of tone | Sentiment/style analysis | Match expected tone |
| **Response Clarity** | Ease of understanding | Readability + LLM assessment | >4.0 (1-5 scale) |
| **User Satisfaction (CSAT)** | Direct user feedback | Survey score (1-5) | >4.0 |

---

## A.11 Robustness Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Error Recovery Rate** | Ability to recover from failures | `(Recovered / Total Errors) × 100` | >80% |
| **Hallucination Rate** | Generation of false information | `(Hallucinated / Total Facts) × 100` | <5% |
| **Autonomy Score** | Steps without human intervention | `(Autonomous Steps / Total) × 100` | Context-dependent |
| **Stress Test Performance** | Performance under load | Metrics under peak load | <20% degradation |
| **Failure Mode Distribution** | Types of failures encountered | Categorize and analyze failures | No catastrophic modes |
| **Resilience to Perturbations** | Stability under input variations | Performance with perturbed inputs | <10% degradation |

---

## A.12 Business Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **Conversion Rate Impact** | Effect on user conversions | A/B test comparison | Positive delta |
| **Agent-Assisted Conversion Rate** | Conversions with agent help | `(Agent-Assisted Conversions / Total) × 100` | >baseline |
| **Return on Investment (ROI)** | Financial return from agent deployment | `(Value Generated - Cost) / Cost` | >100% |
| **Outcome-Aligned Economics** | Cost relative to successful outcomes | `Cost / Successful Outcome` | Optimize over time |

---

## A.13 Drift and Distribution Metrics

| Metric | Definition | Formula | Benchmark |
|--------|------------|---------|-----------|
| **KL Divergence** | Distribution difference measurement | Statistical measure | Alert at >0.3 |
| **Population Stability Index (PSI)** | Population distribution change | Statistical measure | Alert at >0.25 |
| **Data Drift Score** | Input distribution change | Various statistical tests | <0.1 for stability |
| **Distribution Health Score** | Overall distribution alignment | Composite of drift metrics | Monitor trends |

---

## A.14 Domain-Specific Metrics

### RAG System Metrics

| Metric | Definition | Benchmark |
|--------|------------|-----------|
| **Context Precision** | Relevance of retrieved chunks | >0.8 |
| **Context Recall** | Coverage of needed information | >0.8 |
| **Faithfulness** | Grounding in retrieved context | >0.95 |
| **Answer Relevancy** | Response relevance to query | >0.85 |

### Code Generation Metrics

| Metric | Definition | Benchmark |
|--------|------------|-----------|
| **Code Quality Score** | Overall code quality | Linting score >90% |
| **Security Vulnerabilities** | Identified security issues | 0 high-severity |
| **Functional Correctness** | Code passes tests | >95% test pass rate |
| **Code Efficiency** | Performance of generated code | Within 2x optimal |

### Customer Support Metrics

| Metric | Definition | Benchmark |
|--------|------------|-----------|
| **Resolution Quality** | Quality of issue resolution | >4.0 (1-5 scale) |
| **Escalation Rate** | Hand-offs to human agents | <30% |
| **Customer Satisfaction** | Post-interaction CSAT | >4.0 |
| **Average Handle Time** | Time to resolve issues | <5 minutes typical |

### Research Assistant Metrics

| Metric | Definition | Benchmark |
|--------|------------|-----------|
| **Source Quality** | Credibility of sources used | >90% credible sources |
| **Citation Accuracy** | Correctness of citations | >98% |
| **Research Depth** | Comprehensiveness of research | Task-dependent |
| **Synthesis Quality** | Quality of information synthesis | >4.0 (1-5 scale) |

### Healthcare Metrics

| Metric | Definition | Benchmark |
|--------|------------|-----------|
| **Clinical Accuracy** | Correctness of medical info | >99% |
| **Policy Adherence** | Compliance with medical policies | 100% |
| **Error Rate** | Medical information errors | <0.1% |
| **Disclaimer Compliance** | Appropriate medical disclaimers | 100% |

### Voice/Conversational Agent Metrics

| Metric | Definition | Benchmark |
|--------|------------|-----------|
| **Latency** | Response time | <1000ms (sub-500ms ideal) |
| **Turn-Taking Quality** | Natural conversation flow | <5% interruption errors |
| **Speech Recognition Accuracy** | Correct transcription | >95% |
| **Natural Language Understanding** | Intent recognition accuracy | >90% |

---

## Safety Benchmark Metrics Summary

| Benchmark | Primary Metrics | Focus |
|-----------|-----------------|-------|
| **AgentAuditor** | Human-level accuracy for LLM-as-Judge | Safety evaluation calibration |
| **RAS-Eval** | Task completion under attack (36.78% reduction) | Real-world security |
| **AgentDojo** | 97 tasks, 629 security cases | Prompt injection robustness |
| **AgentHarm** | HarmScore, RefusalRate | Malicious use prevention |

---

## Metric Selection Guide

### By Use Case

| Use Case | Primary Metrics |
|----------|-----------------|
| **Customer Support** | FCR, Containment Rate, CSAT, Escalation Rate |
| **Code Generation** | Functional Correctness, Security Vulnerabilities, Code Quality |
| **Research/Analysis** | Source Quality, Citation Accuracy, Factual Correctness |
| **Healthcare** | Clinical Accuracy, Policy Adherence, Error Rate |
| **Voice Agents** | Latency, Turn-Taking Quality, Speech Recognition Accuracy |
| **RAG Systems** | Context Precision, Context Recall, Faithfulness |

### By Evaluation Phase

| Phase | Priority Metrics |
|-------|------------------|
| **Development** | Task Success, Plan Quality, Tool Correctness |
| **Pre-Production** | Safety Metrics, Security Metrics, Robustness |
| **Production** | Latency, Cost, CSAT, Error Rate, Drift Metrics |
| **Continuous Monitoring** | All drift metrics, Business Metrics, Real-time Safety |

---

## References

1. Sections 5-8 of this guide for detailed metric implementations
2. Galileo AI. (2025). *Four New Agent Evaluation Metrics*
3. DeepEval Documentation. (2025). *AI Agent Evaluation Metrics*
4. Azure AI Foundry. (2025). *Agent Evaluation Documentation*
5. LangChain. (2026). *State of AI Agents Survey*

---

**Navigation:**
← [Previous Section: Conclusion and Recommendations](../31_conclusion_and_recommendations.md) | [Table of Contents](../../README.md) | [Next Appendix: Tool and Platform Comparison Tables](appendix_b_tool_platform_comparison_tables.md) →
