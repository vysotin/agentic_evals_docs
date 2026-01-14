# Section 24: Use Case-Specific Evaluation

> **Part IX**: Industry Insights & Case Studies
> **Estimated Reading Time**: 20 minutes

## Overview

Different AI agent applications require fundamentally different evaluation approaches. A customer support agent optimizes for resolution and satisfaction; a code generation agent optimizes for correctness and security; a healthcare agent prioritizes safety above all else. This section provides domain-specific evaluation frameworks, metrics, and success criteria for six major agent use cases, enabling you to tailor your evaluation strategy to your specific application.

## Table of Contents

- [24.1 Customer Support Agents](#241-customer-support-agents)
- [24.2 Research and Analysis Agents](#242-research-and-analysis-agents)
- [24.3 Code Generation Agents](#243-code-generation-agents)
- [24.4 Healthcare Agents](#244-healthcare-agents)
- [24.5 Financial Services Agents](#245-financial-services-agents)
- [24.6 Voice and Conversational Agents](#246-voice-and-conversational-agents)

---

## 24.1 Customer Support Agents

Customer support represents one of the most mature and widely deployed applications of AI agents, with clear metrics and established benchmarks.

### Key Metrics

| Metric | Description | Industry Benchmark |
|--------|-------------|-------------------|
| **First Contact Resolution (FCR)** | Issues resolved in first interaction | 70-79% good; 80%+ world-class (only 5% achieve this) |
| **Customer Satisfaction (CSAT)** | Post-interaction satisfaction score | 75-84% good; 85%+ world-class |
| **Containment Rate** | Tickets resolved by AI without human handoff | 20-40% early pilots; 60-80% at maturity |
| **Average Handle Time (AHT)** | Time to resolve an issue | 20-30% reduction target |
| **Net Promoter Score (NPS)** | Customer loyalty indicator | Industry-specific benchmarks |

**The FCR Imperative**: First Contact Resolution is the north star metric for customer support agents. Each 1% improvement in FCR correlates with approximately 1% reduction in operational costs [1]. World-class FCR (80%+) is achieved by only 5% of support organizations, making it a significant competitive differentiator.

### AI-Specific Metrics (2026)

Beyond traditional support metrics, AI agents require specialized evaluation:

| Metric | Description | Purpose |
|--------|-------------|---------|
| **Sentiment Shift Score** | How customer emotion changes during interaction | Measures de-escalation effectiveness |
| **Agent Assist Utilization** | How often human agents rely on AI recommendations | Tracks AI value in assisted mode |
| **AI Deflection Rate** | Tickets resolved without human involvement | Core automation metric |
| **Human-AI Handoff Quality** | Quality of transitions to human agents | Prevents customer frustration |
| **Escalation Appropriateness** | Whether escalations were truly necessary | Balances containment vs. quality |

> "AI agents should be measured using the same scorecard as human agents." — Quidget [1]

### Test Scenarios

Comprehensive evaluation requires testing across multiple scenario types:

| Scenario Category | Examples |
|-------------------|----------|
| **Simple inquiries** | Order status, password reset, FAQ questions |
| **Complex multi-turn** | Billing disputes, product troubleshooting |
| **Emotional customers** | Complaints, frustration, anger management |
| **Edge cases** | Unusual requests, policy exceptions |
| **Escalation triggers** | Situations requiring human intervention |
| **Multi-intent queries** | Customers with multiple issues in one message |

### Success Criteria

A production-ready customer support agent should demonstrate:

| Criterion | Target |
|-----------|--------|
| AI containment rate | 60-80% at maturity |
| CSAT on AI-handled tickets | ≥90% (matching human performance) |
| Escalation appropriateness | >95% accurate escalation decisions |
| Response relevance | >95% relevant to customer query |
| Policy compliance | 100% adherence to company policies |
| Handoff quality | Seamless context transfer to humans |

**Cost Impact**: Gartner predicts conversational AI could save **$80 billion in contact center labor costs by 2026** [2]. Organizations achieving 80% containment rates report the highest ROI.

---

## 24.2 Research and Analysis Agents

Research agents face unique challenges around source quality, citation accuracy, and factual grounding—areas where current models struggle significantly.

### Source Quality Metrics

| Metric | Description | Benchmark |
|--------|-------------|-----------|
| **Citation Accuracy** | Percentage of citations correctly supporting claims | GPT-4o with Web Search: ~70% statements supported |
| **Source Authority Score** | Quality ranking of cited sources | Academic/institutional sources preferred |
| **Faithfulness** | Whether responses contain hallucinations | Target: >90% faithful statements |
| **Source Diversity** | Variety of perspectives represented | Context-dependent |

### Critical Findings from 2025 Research

The SourceCheckup framework evaluated 7 popular LLMs on 800 questions and 58,000 statement-source pairs, revealing alarming results [3]:

| Finding | Value |
|---------|-------|
| LLM responses not fully supported by cited sources | 50-90% |
| GPT-4o with Web Search: statements unsupported | ~30% |
| GPT-4o responses not fully supported | ~50% |

**Implication**: Even with web search enabled, approximately half of AI-generated research contains unsupported claims. This makes citation verification essential for research applications.

### Retrieval Relevance Metrics

For RAG-based research agents, retrieval quality directly impacts output quality [4]:

| Metric | Target | Purpose |
|--------|--------|---------|
| **Precision@5** | ≥0.7 (specialized domains) | Top-k document relevance |
| **Recall@20** | ≥0.8 (broader datasets) | Retrieval completeness |
| **NDCG@10** | >0.8 | Ranking quality with position weighting |
| **MRR (Mean Reciprocal Rank)** | Track trends | First relevant result position |
| **Context Precision** | >0.85 | Relevant chunks in retrieved context |
| **Context Recall** | >0.80 | Important information retrieved |

### Citation Verification Approaches

| Tool/Approach | Description |
|---------------|-------------|
| **Scite** | 1.4B+ indexed citations with Smart Citations showing support/contradiction |
| **Elicit** | Sentence-level citations backing all AI-generated claims |
| **SourceCheckup** | Framework for evaluating citation accuracy |
| **Manual spot-checking** | Human verification of random samples |

**Model Performance**: Research from arXiv shows that a 4B fine-tuned model achieved 83.64% weighted accuracy for citation classification, suggesting specialized models can improve verification [5].

### Success Criteria

| Criterion | Target |
|-----------|--------|
| Citation accuracy | >85% statements verifiably supported |
| Source authority | >90% from authoritative sources |
| Factual correctness | >95% factually accurate statements |
| Comprehensive coverage | Addresses all aspects of the query |
| Bias disclosure | Identifies when sources have conflicting views |

---

## 24.3 Code Generation Agents

Code generation agents require evaluation across functional correctness, code quality, and critically—security. The relationship between these dimensions isn't straightforward: high functional accuracy doesn't imply security.

### Code Quality Metrics

| Metric | Description | 2025 Top Performance |
|--------|-------------|---------------------|
| **Pass@1** | First attempt functional correctness | Claude Sonnet 4: 95.1%, O1 models: 96.3% |
| **Pass@5** | Chance of working solution in 5 attempts | Higher than Pass@1 by 5-15% |
| **Cyclomatic Complexity** | Code complexity measurement | Lower is better |
| **Maintainability Index** | Long-term code quality | Higher is better |
| **Code Coverage** | Test coverage of generated code | >80% target |

### Security Vulnerability Detection

Security represents a critical and often overlooked dimension. Veracode's 2025 GenAI Code Security Report reveals concerning findings [6]:

| Finding | Statistic |
|---------|-----------|
| Code samples failing security tests | 45% |
| Java security failure rate | 72% (highest risk) |
| Cross-Site Scripting (CWE-80) defense failure | 86% |

**Critical Insight**: No correlation was found between functional performance (Pass@1) and security quality. Models improved at writing functional code but not more secure code. **High Pass@1 scores do not indicate security**.

### Security-Specific Evaluation

| Vulnerability Category | Test Approach |
|------------------------|---------------|
| **SQL Injection (CWE-89)** | Input validation testing |
| **Cross-Site Scripting (CWE-79/80)** | Output encoding verification |
| **Hardcoded Credentials (CWE-798)** | Secret scanning |
| **Path Traversal (CWE-22)** | File path validation |
| **Command Injection (CWE-78)** | Shell input sanitization |
| **Insecure Dependencies** | SCA scanning |

### Evaluation Approaches

| Approach | Tools | Purpose |
|----------|-------|---------|
| **Static Analysis** | Snyk DeepCode AI, SonarQube | Pattern-based vulnerability detection |
| **Dynamic Testing** | OWASP ZAP, Burp Suite | Runtime security testing |
| **Software Composition Analysis** | Dependabot, Snyk | Third-party dependency vulnerabilities |
| **Automated Remediation** | Google CodeMender | AI-powered security fixes |

**Google CodeMender**: Google DeepMind's CodeMender has upstreamed 72 security fixes to open source projects, demonstrating AI can both create and fix security vulnerabilities [7].

### Benchmarks

| Benchmark | Description | Notes |
|-----------|-------------|-------|
| **HumanEval** | 164 Python programming challenges | Contemporary models approach 90%+ Pass@1 |
| **HumanEval_T** | Translation variants | Shows 14 percentage point drops, indicating contamination |
| **SWE-Bench** | Real-world software issues | More realistic than HumanEval |
| **MBPP** | Mostly Basic Python Problems | Easier than HumanEval |

### Success Criteria

| Criterion | Target |
|-----------|--------|
| Functional correctness (Pass@1) | >90% |
| Security scan pass rate | >85% (zero critical vulnerabilities) |
| Code style compliance | >95% linter pass rate |
| Test coverage | >80% for generated code |
| Documentation quality | Appropriate comments and docstrings |
| Build success rate | >95% first-time compilation |

---

## 24.4 Healthcare Agents

Healthcare represents the highest-stakes domain for AI agents, where evaluation failures can directly impact patient safety. Regulatory compliance and human oversight are non-negotiable.

### Safety and Compliance Requirements

| Requirement | Framework | Status (2025) |
|-------------|-----------|---------------|
| **FDA SaMD Compliance** | AI-enabled Device Software Functions | Draft guidance issued January 2025 |
| **EU AI Act** | AI literacy mandated | Effective February 2025 |
| **HIPAA** | Data privacy | Ongoing requirement |
| **Performance Monitoring** | Post-market surveillance | Required with drift detection |

### FDA 2025 Requirements

The FDA's January 2025 guidance establishes comprehensive requirements [8]:

| Requirement | Description |
|-------------|-------------|
| **Total Product Lifecycle (TPLC)** | Continuous evaluation throughout product life |
| **Performance Validation** | Independent datasets with subgroup analyses |
| **Human-AI Team Evaluation** | Reader studies assessing combined performance |
| **Post-market Monitoring** | Drift detection and ongoing surveillance |
| **Subgroup Analysis** | Performance across demographics, conditions |

### Policy Adherence Framework

From PMC's checklist methodology for healthcare AI [9]:

**Clinical/Technical Validation:**
- Evidence-based performance assessment
- Real-world validation
- MDR compliance (EU Medical Device Regulation)
- GDPR adherence
- Post-deployment monitoring

**Governance:**
- AI Act conformity assessment
- Decision traceability
- Human oversight mechanisms
- AI literacy requirements
- Audit mechanisms

### Error Rate Requirements

Healthcare has no universal accuracy threshold—performance requirements are tied to specific device claims [10]:

| Precedent | Performance |
|-----------|-------------|
| **IDx-DR** (first autonomous AI diagnostic) | 87% sensitivity, 90% specificity |
| **FDA-authorized AI devices** | Over 1,250 as of July 2025 |
| **Performance targets** | Device-specific, claim-dependent |

### Human Oversight

> "AI must be accompanied by human oversight. Overreliance can lead to critical errors. Tools cannot be solely relied upon for diagnosis without human judgment." — Morgan Lewis [11]

| Oversight Requirement | Purpose |
|----------------------|---------|
| Human review of AI decisions | Catch AI errors before patient impact |
| Explainable outputs | Enable clinician understanding |
| Override capability | Allow human correction |
| Audit trails | Regulatory compliance and learning |
| Escalation protocols | Clear paths for uncertain cases |

### Success Criteria

| Criterion | Target |
|-----------|--------|
| Diagnostic accuracy | Device-specific (FDA approval threshold) |
| Sensitivity | Meet or exceed device claims |
| Specificity | Meet or exceed device claims |
| HIPAA compliance | 100% |
| Explainability | Clear reasoning for all recommendations |
| Human override rate | Tracked and analyzed |
| Adverse event rate | Below acceptable threshold |

---

## 24.5 Financial Services Agents

Financial services agents operate under intense regulatory scrutiny, requiring 100% auditable decision-making and real-time compliance monitoring.

### Accuracy Requirements

| Requirement | Standard | Framework |
|-------------|----------|-----------|
| **Decision Traceability** | 100% auditable | FINRA 2026 Report |
| **SOX Compliance** | Full audit trail | Sarbanes-Oxley |
| **GDPR Adherence** | Data privacy | EU regulation |
| **DORA Compliance** | Digital operational resilience | Effective January 2025 |
| **Continuous Monitoring** | Real-time compliance | Industry standard |

### Regulatory Compliance

FINRA's 2026 Report establishes that generative AI now demands the **same compliance rigor as any critical system** [12]:

| Requirement | Description |
|-------------|-------------|
| Narrow scope | Agents limited to defined functions |
| Explicit permissions | Clear authorization for actions |
| Audit trails | Complete decision logging |
| Human checkpoints | Review for significant decisions |
| Regular validation | Ongoing accuracy verification |

### Audit Trail Requirements

From Grid Dynamics' agentic AI compliance strategy [13]:

| Requirement | Implementation |
|-------------|----------------|
| **Decision Lineage** | Who/what made decisions, why, under what conditions |
| **Timestamp Preservation** | Every change preserved for tamper-proof trails |
| **Explainable AI Dashboards** | Human-readable reasoning for each decision |
| **Reproducibility** | Inputs, outputs, and human overrides documented |
| **Immutable Logs** | Cannot be modified after creation |

### Continuous Monitoring

The GAO Report GAO-25-107197 emphasizes an **"always-on" assessment model** replacing point-in-time reviews [14]:

| Monitoring Type | Purpose |
|-----------------|---------|
| Compliance documentation | Continuous policy adherence |
| Risk indicators | Real-time risk flagging |
| Transaction activity | Anomaly detection |
| SOX audit preparation | 40% time reduction with AI tools |
| FINRA compliance | Real-time violation detection |

### AI Agent Capabilities for Finance

Per RTS Labs' 2026 guide [15]:

| Capability | Benefit |
|------------|---------|
| Built-in rule engines | Cross-verify every transaction |
| Behavior-based fraud detection | Reduced false positives |
| Network correlation | Risk scoring across entities |
| Pattern recognition | Anomaly identification |
| Regulatory reporting | Automated compliance documentation |

### Success Criteria

| Criterion | Target |
|-----------|--------|
| Audit trail completeness | 100% decision traceability |
| Regulatory compliance | Zero violations |
| Transaction accuracy | >99.9% |
| Fraud detection rate | Industry-benchmark dependent |
| False positive rate | <5% for fraud alerts |
| Response time for compliance queries | <24 hours |
| Explainability | Clear reasoning for all decisions |

---

## 24.6 Voice and Conversational Agents

Voice agents introduce unique challenges around latency, turn-taking, and real-time interaction quality that don't apply to text-based agents.

### Latency Requirements

Voice conversation has strict latency requirements that fundamentally constrain architecture [16]:

| Metric | Target | Notes |
|--------|--------|-------|
| **Human Expectation** | 300-500ms | Natural conversation rhythm |
| **Maximum Acceptable** | 800ms | Beyond this, conversations feel stilted |
| **Time-to-First-Token (TTFT)** | 250ms (local) to >1s (large third-party) | Model dependent |
| **Traditional IVR** | ~950ms (Twilio benchmark) | Falls short of modern expectations |

**The 600ms Bar**: 600ms is the bare minimum for realistic conversations. Semantic turn detection models can reduce response times to **under 300ms** without cutting users off [17].

### Key Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Mouth-to-Ear Turn Gap** | User perception: from user stops speaking to agent reply reaches ear | <800ms |
| **Platform Turn Gap** | Latency within voice agent platform only | <400ms |
| **Turn-Level Latency** | Measured at every exchange, not just average | Track distribution |
| **Interruption Count** | Agent talking over customer frequency | Minimize |
| **Agent Talk Ratio** | Percentage of time agent holds floor | Context-dependent |
| **ASR Accuracy** | Speech recognition correctness | >95% |
| **TTS Quality** | Speech synthesis naturalness | User satisfaction >4/5 |

### Turn-Taking Quality

Turn-taking is more than just timing—it carries meaning [18]:

| Aspect | Importance |
|--------|------------|
| **Silence as signal** | Pauses function as paralinguistic communication |
| **Rhythm interpretation** | Turn-taking patterns shape meaning |
| **Interruption handling** | Graceful recovery from overlapping speech |
| **Backchanneling** | "Uh-huh," "I see" to maintain engagement |
| **End-of-turn detection** | Knowing when user has finished speaking |

### Evaluation Tools (2025)

| Tool | Specialization |
|------|----------------|
| **Maxim AI** | End-to-end simulation, evaluation, observability |
| **Roark** | Production monitoring, 10M+ minutes processed |
| **Coval** | Latency, accuracy, tool-call effectiveness, instruction compliance |
| **Future AGI** | Audio-level evaluation and latency detection |
| **Hamming AI** | 4-layer evaluation framework |

### The 4-Layer Framework

Hamming AI's framework for voice agent quality assurance [18]:

| Layer | Evaluation Focus |
|-------|-----------------|
| **ASR/TTS Quality** | Speech recognition and synthesis accuracy |
| **Tool Calls** | External function execution |
| **LLM Reasoning** | Response generation quality |
| **Timing Analysis** | Component-by-component delay breakdown |

### Success Criteria

| Criterion | Target |
|-----------|--------|
| End-to-end latency | <800ms, target <500ms |
| ASR accuracy | >95% word recognition |
| Turn-taking quality | Natural conversation feel |
| Interruption handling | Graceful recovery |
| Task completion rate | Domain-dependent, but >85% |
| User satisfaction | >4/5 rating |
| Call completion rate | >90% without user hang-up |

---

## Summary: Cross-Domain Comparison

| Domain | Primary Metric | Key Threshold | Unique Consideration |
|--------|---------------|---------------|---------------------|
| **Customer Support** | Containment Rate | 60-80% at maturity | Same scorecard as humans |
| **Research/Analysis** | Citation Accuracy | 70%+ statements supported | 50-90% may be unsupported |
| **Code Generation** | Pass@1 + Security | 45% fail security tests | Functional ≠ Secure |
| **Healthcare** | Compliance + Safety | FDA-specific per device | Human oversight mandatory |
| **Financial Services** | Audit Trail | 100% traceable | Real-time compliance monitoring |
| **Voice/Conversational** | Latency | <800ms, target <500ms | Turn-taking affects meaning |

---

## Key Takeaways

1. **No one-size-fits-all**: Each domain requires specialized metrics aligned with its unique success criteria

2. **Customer support is mature**: Clear benchmarks exist (FCR 80%+, containment 60-80%), making it a good starting point for agent deployment

3. **Research agents struggle with citations**: Even the best models have 30-50% of statements unsupported—verification is essential

4. **Code security ≠ functional correctness**: 45% of AI code fails security tests even when functionally correct—security evaluation is non-negotiable

5. **Healthcare demands regulatory compliance**: FDA, EU AI Act, and HIPAA requirements define what's possible, not just desirable

6. **Financial services require 100% auditability**: Regulatory requirements mandate complete decision traceability

7. **Voice agents live or die by latency**: 800ms maximum, 500ms target—architectural decisions are constrained by this requirement

8. **Human oversight varies by domain**: Critical in healthcare and finance, optional (but valuable) in customer support and research

---

## References

1. Quidget. (2025). *The 6 Metrics That Actually Matter for AI Customer Support in 2025*. [https://quidget.ai/blog/ai-automation/the-6-metrics-that-actually-matter-for-ai-customer-support-in-2025/](https://quidget.ai/blog/ai-automation/the-6-metrics-that-actually-matter-for-ai-customer-support-in-2025/)

2. Plivo. (2025). *Contact Center Statistics and Benchmarks 2025*. [https://www.plivo.com/blog/contact-center-statistics-benchmarks-2025/](https://www.plivo.com/blog/contact-center-statistics-benchmarks-2025/)

3. Nature Communications. (2025). *SourceCheckup: A Framework for Evaluating LLM Citation Accuracy*. [https://www.nature.com/articles/s41467-025-58551-6](https://www.nature.com/articles/s41467-025-58551-6)

4. Future AGI. (2025). *RAG Evaluation Metrics 2025*. [https://futureagi.com/blogs/rag-evaluation-metrics-2025](https://futureagi.com/blogs/rag-evaluation-metrics-2025)

5. arXiv. (2025). *Citation Classification with Fine-Tuned Models*. [https://arxiv.org/html/2511.16198v1](https://arxiv.org/html/2511.16198v1)

6. Veracode. (2025). *GenAI Code Security Report*. [https://www.veracode.com/blog/genai-code-security-report/](https://www.veracode.com/blog/genai-code-security-report/)

7. Google DeepMind. (2025). *Introducing CodeMender: An AI Agent for Code Security*. [https://deepmind.google/blog/introducing-codemender-an-ai-agent-for-code-security/](https://deepmind.google/blog/introducing-codemender-an-ai-agent-for-code-security/)

8. Oriel STAT A MATRIX. (2025). *FDA 2025 Guidance on AI-Enabled Medical Devices*. [https://www.orielstat.com/blog/fda-2025-guidance-ai-enabled-medical-devices/](https://www.orielstat.com/blog/fda-2025-guidance-ai-enabled-medical-devices/)

9. PMC. (2025). *Healthcare AI Policy Checklist Methodology*. [https://pmc.ncbi.nlm.nih.gov/articles/PMC12465464/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12465464/)

10. American Heart Association. (2025). *New Guidance Offered for Responsible AI Use in Health Care*. [https://newsroom.heart.org/news/new-guidance-offered-for-responsible-ai-use-in-health-care](https://newsroom.heart.org/news/new-guidance-offered-for-responsible-ai-use-in-health-care)

11. Morgan Lewis. (2025). *AI in Healthcare: Opportunities, Enforcement Risks, and the Need for AI-Specific Compliance*. [https://www.morganlewis.com/pubs/2025/07/ai-in-healthcare-opportunities-enforcement-risks-and-false-claims-and-the-need-for-ai-specific-compliance](https://www.morganlewis.com/pubs/2025/07/ai-in-healthcare-opportunities-enforcement-risks-and-false-claims-and-the-need-for-ai-specific-compliance)

12. Shumaker. (2025). *Generative AI in Financial Services: A Practical Compliance Playbook for 2026*. [https://www.shumaker.com/insight/client-alert-generative-artificial-intelligence-in-financial-services-a-practical-compliance-playbook-for-2026/](https://www.shumaker.com/insight/client-alert-generative-artificial-intelligence-in-financial-services-a-practical-compliance-playbook-for-2026/)

13. Grid Dynamics. (2025). *Agentic AI Regulatory Compliance Strategy*. [https://www.griddynamics.com/blog/agentic-ai-regulatory-compliance-strategy](https://www.griddynamics.com/blog/agentic-ai-regulatory-compliance-strategy)

14. GAO. (2025). *AI in Financial Services Report (GAO-25-107197)*. [https://www.gao.gov/products/gao-25-107197](https://www.gao.gov/products/gao-25-107197)

15. RTS Labs. (2026). *AI Agents for Finance*. [https://rtslabs.com/ai-agents-for-finance/](https://rtslabs.com/ai-agents-for-finance/)

16. Twilio. (2025). *Guide to Core Latency for AI Voice Agents*. [https://www.twilio.com/en-us/blog/developers/best-practices/guide-core-latency-ai-voice-agents](https://www.twilio.com/en-us/blog/developers/best-practices/guide-core-latency-ai-voice-agents)

17. Retell AI. (2025). *AI Voice Agent Latency Face-Off 2025*. [https://www.retellai.com/resources/ai-voice-agent-latency-face-off-2025](https://www.retellai.com/resources/ai-voice-agent-latency-face-off-2025)

18. Hamming AI. (2025). *Guide to AI Voice Agents Quality Assurance*. [https://hamming.ai/resources/guide-to-ai-voice-agents-quality-assurance](https://hamming.ai/resources/guide-to-ai-voice-agents-quality-assurance)

---

**Navigation:**
← [Previous Section: Success Patterns and Anti-Patterns](23_success_patterns_and_anti_patterns.md) | [Table of Contents](../README.md) | [Next Section: Vendor and Expert Insights](25_vendor_and_expert_insights.md) →
