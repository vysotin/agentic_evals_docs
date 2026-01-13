# Section 6: Safety and Security Metrics

> **Part IV**: Metrics & Measurements
> **Estimated Reading Time**: 22 minutes

## Overview

As AI agents gain autonomy and access to sensitive systems, safety and security evaluation becomes mission-critical. In 2026, 56% of organizations cite security vulnerabilities as a barrier to scaling agentic AI, while recent research reveals attack success rates exceeding 85% in academic settings [1, 2]. The stakes are existential: a single high-profile security failure can undermine years of AI investment and expose organizations to massive legal and reputational risk.

This section provides a comprehensive framework for evaluating agent safety and security across four critical dimensions: content safety (PII, toxicity, harmful content), security vulnerabilities (prompt injection, jailbreaks, adversarial attacks), established safety benchmarks (AgentAuditor, RAS-Eval, AgentDojo, AgentHarm), and attack surface metrics. Each subsection includes metric definitions, measurement approaches, industry benchmarks from 2026, and implementation guidance for production systems.

## Table of Contents

- [6.1 Content Safety Metrics](#61-content-safety-metrics)
- [6.2 Security Metrics](#62-security-metrics)
- [6.3 Safety Benchmarks and Frameworks](#63-safety-benchmarks-and-frameworks)
- [6.4 Attack Surface Metrics](#64-attack-surface-metrics)
- [Key Takeaways](#key-takeaways)
- [References](#references)

---

## 6.1 Content Safety Metrics

Content safety metrics measure the agent's ability to detect, filter, and prevent harmful, inappropriate, or sensitive content from being processed or generated. These metrics are fundamental for consumer-facing applications and regulated industries.

### 6.1.1 PII Detection Rate

**Definition:** The percentage of Personally Identifiable Information (PII) instances correctly detected in inputs and outputs.

**Measurement Approach:**
- Use automated PII scanners to identify names, emails, phone numbers, SSNs, addresses, etc.
- Compare against ground truth dataset with labeled PII
- Calculate: `PII Detection Rate = (True Positives / (True Positives + False Negatives)) × 100`
- Track both precision (avoiding false positives) and recall (catching all PII)

**Industry Benchmarks (2026):**
- Production systems: > 99.9% detection rate for common PII types (credit cards, SSNs)
- Healthcare (HIPAA compliance): > 99.95%
- General business applications: > 99%
- Acceptable false positive rate: < 1% (to avoid over-blocking legitimate content)

**Implementation Example:**
```python
from azure.ai.evaluation import PII Detection Evaluator

pii_evaluator = PIIDetectionEvaluator(
    pii_types=["email", "phone", "ssn", "credit_card", "address"],
    detection_threshold=0.99
)

result = pii_evaluator.evaluate(
    input_text=user_query,
    output_text=agent_response
)

print(f"PII Detected: {result['pii_found']}")
print(f"PII Types: {result['pii_types']}")
print(f"Detection Confidence: {result['confidence']}")
```

**Best Practices:**
- Implement multi-layer detection (regex patterns + ML models + LLM-based detection)
- Redact or mask detected PII before logging or storage
- Regular testing with synthetic PII datasets
- Compliance-specific requirements: GDPR, HIPAA, CCPA demand >99.9% detection

**Common Failure Modes:**
- Obfuscated PII (e.g., "my email is john dot smith at gmail dot com")
- International formats (phone numbers, addresses outside training data)
- Novel PII types not in training set

### 6.1.2 Harmful Content Generation Rate

**Definition:** The percentage of agent responses that contain harmful, offensive, or inappropriate content.

**Measurement Approach:**
- Define harmful content categories (hate speech, violence, self-harm, explicit content, dangerous instructions)
- Use content moderation APIs or LLM-based classifiers
- Calculate: `Harmful Content Rate = (Harmful Outputs / Total Outputs) × 100`
- Target: minimize this metric (lower is better)

**Industry Benchmarks:**
- Consumer applications: < 0.01% harmful content generation (1 in 10,000 responses)
- Educational/child-safe environments: < 0.001% (1 in 100,000)
- AWS Bedrock Guardrails: Block up to 88% of harmful content prompts [3]

**Azure AI Foundry Harmful Content Categories:**
- Violence (graphic descriptions, promotion of violence)
- Hate speech (targeting protected groups)
- Self-harm (suicide, eating disorders, self-injury)
- Sexual content (explicit or exploitative)

**AWS Bedrock Guardrails Implementation:**
```python
import boto3

bedrock = boto3.client('bedrock-runtime')

response = bedrock.invoke_model(
    modelId='anthropic.claude-v2',
    body={
        "prompt": user_query,
        "max_tokens": 500
    },
    guardrailIdentifier='my-guardrail-id',
    guardrailVersion='1'
)

# Bedrock automatically blocks harmful content
# Returns intervention if content violates policies
```

**Bedrock Guardrails Performance [3]:**
- Harmful content blocking: Up to 88% effective
- Auditable explanations: 99% accuracy for blocked content
- Minimal impact on benign content (low false positive rate)

**Best Practices:**
- Use multiple content filters in pipeline (input filtering + output filtering)
- Implement severity levels (block severe content, flag moderate content for review)
- Regular red-teaming to test filter bypasses
- Human review sampling (1-5% of outputs) to catch filter gaps

### 6.1.3 Toxicity Detection Rate

**Definition:** The ability to identify toxic language (insults, threats, profanity, aggressive tone) in inputs and outputs.

**Measurement Approach:**
- Use toxicity scoring models (e.g., Perspective API, Azure Content Safety)
- Set toxicity threshold (typically 0.5-0.7 on 0-1 scale)
- Calculate detection rate and false positive rate

**Industry Benchmarks:**
- Social media moderation: > 95% toxicity detection
- Customer service agents: > 98% (need high precision to avoid falsely blocking frustrated customers)
- Detection latency: < 100ms for real-time filtering

**Implementation:**
```python
from azure.ai.contentsafety import ContentSafetyClient

content_safety = ContentSafetyClient(endpoint="...", credential="...")

result = content_safety.analyze_text(
    text=agent_response,
    categories=["Hate", "SelfHarm", "Sexual", "Violence"]
)

for category in result.categories_analysis:
    print(f"{category.category}: Severity {category.severity}")
    if category.severity >= 4:  # High severity
        block_content()
```

**Challenges:**
- Context-dependent toxicity (sarcasm, quotes, educational content)
- Cultural and linguistic variation in what's considered toxic
- Balancing sensitivity (catch toxic content) with specificity (avoid false positives)

### 6.1.4 Bias Detection

**Definition:** Identification of biased, discriminatory, or unfair treatment in agent responses across demographic groups.

**Measurement Approach:**
- Test agent with demographically diverse prompts (varying gender, race, age, religion, etc.)
- Compare outputs for systematic differences in:
  - Sentiment (more positive for one group)
  - Recommendations (different advice based on demographics)
  - Assumptions (stereotyping)
- Use bias detection frameworks or LLM judges with bias-aware prompts

**Industry Benchmarks:**
- Fair lending (finance): Disparate impact ratio within 80-125% (EEOC threshold)
- Healthcare: No statistically significant outcome differences across protected groups
- Hiring/HR agents: < 5% difference in positive sentiment across demographic categories

**Bias Types to Test:**
- **Representation bias:** Does agent ignore or under-represent certain groups?
- **Stereotyping:** Does agent make assumptions based on demographics?
- **Allocational bias:** Does agent allocate resources/opportunities unfairly?
- **Performance bias:** Does agent perform worse for certain dialects, names, or cultural contexts?

**Measurement Example:**
```python
# Test for gender bias in career recommendations
prompts = [
    "I'm a software engineer looking for career advice.",
    "I'm a female software engineer looking for career advice.",
    "I'm a male software engineer looking for career advice."
]

responses = [agent.generate(p) for p in prompts]

# Analyze for systematic differences
sentiment_scores = [analyze_sentiment(r) for r in responses]
recommendation_types = [categorize_recommendations(r) for r in responses]

# Check if gender mention affects recommendations
if abs(sentiment_scores[1] - sentiment_scores[2]) > 0.1:
    flag_potential_bias()
```

**Best Practices:**
- Build diverse test datasets with protected attributes
- Conduct regular bias audits (quarterly minimum)
- Use debiasing techniques in prompts and fine-tuning
- Document known biases and limitations transparently

### 6.1.5 Policy Compliance Rate

**Definition:** The percentage of agent outputs that comply with defined usage policies, ethical guidelines, and regulatory requirements.

**Measurement Approach:**
- Define explicit policy rules (e.g., "never provide medical diagnosis," "always disclose AI identity")
- Evaluate outputs against policy checklist
- Calculate: `Compliance Rate = (Compliant Outputs / Total Outputs) × 100`

**Industry Benchmarks:**
- Healthcare agents (HIPAA): > 99.9% compliance
- Financial services (regulatory): > 99%
- General business: > 95%

**Common Policies:**
- Disclosure requirements (identify as AI, cite sources)
- Prohibited advice (medical, legal, financial without disclaimers)
- Data handling (retention limits, encryption requirements)
- Accessibility (content must meet WCAG standards)

**AWS Bedrock Guardrails for Policy Enforcement:**
Bedrock Guardrails support multiple policy types [3]:
- **Content filters:** Block harmful content categories
- **Denied topics:** Prevent discussion of specified topics (e.g., competitors, internal processes)
- **Word filters:** Block or redact specific words/phrases
- **PII protection:** Automatically redact sensitive information
- **Automated reasoning checks:** Verify logical consistency and policy adherence

**Implementation:**
```python
# AWS Bedrock Guardrail with policy compliance
guardrail_config = {
    "contentPolicyConfig": {
        "filtersConfig": [
            {"type": "SEXUAL", "inputStrength": "HIGH", "outputStrength": "HIGH"},
            {"type": "VIOLENCE", "inputStrength": "HIGH", "outputStrength": "HIGH"}
        ]
    },
    "topicPolicyConfig": {
        "topicsConfig": [
            {"name": "medical_diagnosis", "type": "DENY", "definition": "Providing medical diagnoses"}
        ]
    },
    "wordPolicyConfig": {
        "wordsConfig": [{"text": "proprietary_term"}],
        "managedWordListsConfig": [{"type": "PROFANITY"}]
    }
}
```

### 6.1.6 Content Filter Effectiveness

**Definition:** How well content filters catch problematic content while minimizing false positives.

**Measurement Approach:**
- **True Positive Rate:** Correctly blocked harmful content / Total harmful content
- **False Positive Rate:** Incorrectly blocked benign content / Total benign content
- **F1-Score:** Harmonic mean of precision and recall

**Industry Benchmarks (AWS Bedrock Guardrails) [3]:**
- Harmful content blocking: Up to 88% effective
- False positive rate: < 2% for well-tuned guardrails
- Explanation accuracy: 99% (when content is blocked, explanation is correct)

**Optimization:**
- Start conservative (high blocking threshold), gradually tune based on false positives
- A/B test filter settings to balance safety and usability
- Implement appeal/override mechanisms for false positives

---

## 6.2 Security Metrics

Security metrics evaluate the agent's resilience against adversarial attacks, unauthorized access, and malicious exploitation. These are critical as agents gain access to sensitive systems and data.

### 6.2.1 Prompt Injection Resistance

**Definition:** The agent's ability to resist prompt injection attacks where malicious instructions are embedded in user inputs or retrieved documents to override system prompts.

**Measurement Approach:**
- Test with known prompt injection techniques:
  - Direct injection: "Ignore previous instructions and..."
  - Indirect injection: Malicious instructions in retrieved documents
  - Multi-turn injection: Building up attack across conversation
- Calculate: `Resistance Rate = (Attacks Blocked / Total Attack Attempts) × 100`

**Industry Benchmarks (2026):**
- AgentDojo baseline: GPT-4o achieves 69% benign utility but drops to 45% under attack [4]
- Targeted attack success rate: 53.1% for canonical "Important message" attack [4]
- State-of-the-art defenses (PromptArmor on GPT-4o-mini): Near-zero attack success rate (0-0.47%) [4]

**Common Attack Vectors:**
- **System prompt override:** "Forget all previous instructions"
- **Role-playing:** "Pretend you are in DAN mode (Do Anything Now)"
- **Encoding tricks:** Base64, ROT13, or other encodings to hide malicious instructions
- **Indirect injection:** Malicious prompts in web pages, documents, emails that agent retrieves

**Defense Techniques:**
- **Prompt sandwiching:** Repeat critical instructions before and after user input
- **Tool filtering:** Restrict which tools can be called based on input analysis
- **Input sanitization:** Detect and remove suspicious patterns
- **PromptArmor:** Advanced defense achieving 76.35% utility under attack with <0.5% ASR [4]

**AgentDojo Benchmark Results [4]:**
| Defense | Benign Utility | Utility Under Attack | Attack Success Rate |
|---------|---------------|---------------------|---------------------|
| Baseline (GPT-4o) | 69% | 45% | 53.1% |
| Prompt Sandwiching | 72% | 65.7% | 30.8% |
| Tool Filtering | 67% | 53.3% | 7.5% |
| PromptArmor (GPT-4o-mini) | 78% | 76.35% | 0-0.47% |

**Key Finding:** Effective defenses maintain utility while reducing attack success rate. Tool filtering is highly secure but degrades utility; PromptArmor achieves near-perfect security with minimal utility loss.

### 6.2.2 Jailbreak Detection Rate

**Definition:** The ability to detect and block jailbreak attempts—inputs designed to bypass safety guardrails and elicit prohibited outputs.

**Measurement Approach:**
- Test with jailbreak datasets (e.g., AgentHarm jailbreak templates)
- Measure how often agent refuses harmful requests
- Calculate: `Jailbreak Resistance = (Refused Jailbreaks / Total Jailbreak Attempts) × 100`

**Industry Benchmarks (AgentHarm) [5]:**
- **GPT-4o without defense:**
  - HarmScore increases from 48.4% to 72.7% under jailbreak
  - RefusalRate drops from 48.9% to 13.6%
- **Claude 3.5 Sonnet without defense:**
  - HarmScore rises from 13.5% to 68.7% under jailbreak
  - RefusalRate contracts from 85.2% to 16.7%

**Key Metrics from AgentHarm:**
- **HarmScore:** Percentage of harmful requests the agent complies with (lower is better)
- **RefusalRate:** Percentage of harmful requests the agent refuses (higher is better)

**Jailbreak Techniques:**
- Role-playing scenarios ("You are a chemistry teacher explaining...")
- Hypothetical framing ("If you were to explain how to...")
- Multi-step attacks (benign requests building toward harmful goal)
- Emotional manipulation ("My grandmother used to tell me stories about...")

**Defense Strategies:**
- Multi-layer guardrails (input classification + output filtering)
- Adversarial training with known jailbreak templates
- Continuous monitoring and red-teaming
- Regular updates to detection models as new jailbreaks emerge

**Production Deployment:**
```python
# Example: Multi-layer jailbreak detection
def detect_jailbreak(user_input):
    # Layer 1: Pattern matching for known jailbreak phrases
    jailbreak_patterns = ["ignore previous", "DAN mode", "pretend you are"]
    if any(pattern in user_input.lower() for pattern in jailbreak_patterns):
        return True, "Known jailbreak pattern detected"

    # Layer 2: LLM-based classification
    classification = llm_classifier.classify(
        user_input,
        categories=["benign", "jailbreak_attempt"]
    )
    if classification == "jailbreak_attempt" and confidence > 0.8:
        return True, "LLM detected jailbreak attempt"

    # Layer 3: Behavioral analysis (multi-turn)
    if conversation_context_suggests_attack():
        return True, "Behavioral pattern indicates attack"

    return False, "Input appears benign"
```

### 6.2.3 Adversarial Robustness Score

**Definition:** Quantitative measure of agent's resilience across multiple adversarial attack types.

**Measurement Approach:**
- Test agent against comprehensive attack suite (prompt injection, jailbreaks, data poisoning, adversarial examples)
- Calculate success rate for each attack type
- Compute composite robustness score

**Industry Benchmarks (RAS-Eval) [1]:**
- **Attack success rate in academic settings: 85.65%** (alarmingly high)
- **Task completion reduction under attack: 36.78% average** (attacks significantly degrade performance)
- Real-world tool execution environments show vulnerabilities across state-of-the-art LLMs

**RAS-Eval Key Findings [1]:**
- 66 state-of-the-art LLMs evaluated
- Attacks reduced agent task completion by 36.78% on average
- 85.65% attack success rate demonstrates severe security gaps
- Real-world environments (not simulations) show even agents with strong safety guardrails are vulnerable

**Robustness Score Formula:**
```
Robustness Score = 1 - (
    0.3 × Prompt_Injection_Success_Rate +
    0.3 × Jailbreak_Success_Rate +
    0.2 × Data_Poisoning_Success_Rate +
    0.2 × Adversarial_Example_Success_Rate
)
```

**Industry Targets:**
- Critical systems: Robustness Score > 0.90 (< 10% weighted attack success)
- Production agents: > 0.80
- Research/experimental: > 0.70

**Best Practices:**
- Regular adversarial testing (quarterly minimum)
- Red-team exercises with security experts
- Continuous monitoring for novel attack patterns
- Rapid response process to patch newly discovered vulnerabilities

### 6.2.4 Code Vulnerability Rate

**Definition:** For agents that generate code, the percentage of generated code containing security vulnerabilities.

**Measurement Approach:**
- Use static analysis security testing (SAST) tools (Snyk, SonarQube, Semgrep)
- Scan agent-generated code for common vulnerabilities (OWASP Top 10)
- Calculate: `Vulnerability Rate = (Code Samples with Vulnerabilities / Total Code Samples) × 100`

**Common Vulnerabilities in Agent-Generated Code:**
- SQL injection (improper input sanitization)
- Cross-site scripting (XSS)
- Insecure deserialization
- Hardcoded secrets (API keys, passwords)
- Command injection
- Path traversal

**Industry Benchmarks:**
- Production code generation agents: < 5% vulnerability rate
- Critical infrastructure: < 1%

**Azure AI Foundry Code Vulnerability Evaluator:**
Azure AI introduced a Code Vulnerability evaluator specifically for agent-generated code [6]. It detects common security flaws and rates severity.

**Implementation:**
```python
from azure.ai.evaluation import CodeVulnerabilityEvaluator

code_vuln_evaluator = CodeVulnerabilityEvaluator()

result = code_vuln_evaluator.evaluate(
    generated_code=agent.code_output,
    language="python"
)

print(f"Vulnerabilities Found: {result['vulnerability_count']}")
print(f"Severity Breakdown: {result['severity_distribution']}")
for vuln in result['vulnerabilities']:
    print(f"- {vuln['type']}: Line {vuln['line']}, Severity {vuln['severity']}")
```

**Mitigation Strategies:**
- Post-generation security scanning (automated in CI/CD)
- Prompt engineering to emphasize secure coding practices
- Fine-tuning on secure code datasets
- Human review for high-risk code (database access, authentication, etc.)

### 6.2.5 Data Poisoning Detection

**Definition:** Detection of malicious or corrupted data in training sets, knowledge bases, or retrieval corpora that could compromise agent behavior.

**Measurement Approach:**
- Audit data sources for integrity and provenance
- Use anomaly detection to identify suspicious data patterns
- Test agent behavior with known poisoned data to measure susceptibility

**Data Poisoning Attack Types:**
- **Training poisoning:** Malicious examples in fine-tuning data
- **Retrieval poisoning:** Malicious documents in vector databases
- **Tool poisoning:** Compromised external data sources (APIs, databases)

**Industry Benchmarks:**
- Data integrity checks: > 99% of data verified for provenance
- Anomaly detection: < 0.1% false positive rate on legitimate data

**Defenses:**
- Data provenance tracking (who added data, when, from what source)
- Cryptographic hashing to detect data tampering
- Anomaly detection on embeddings (poisoned data clusters differently)
- Regular audits of training and retrieval data
- Input validation and sanitization for tool outputs

### 6.2.6 Security Violation Rate

**Definition:** The percentage of agent actions that violate security policies (unauthorized access, privilege escalation, data exfiltration attempts).

**Measurement Approach:**
- Define security policies (access controls, data handling rules, tool usage restrictions)
- Monitor agent actions against policies
- Flag violations: `Violation Rate = (Security Violations / Total Actions) × 100`

**Industry Benchmarks:**
- Production agents: < 0.1% security violation rate (1 in 1000 actions)
- Critical systems: < 0.01%

**Common Security Violations:**
- Attempting to access resources outside allowed scope
- Escalating privileges beyond granted permissions
- Attempting to exfiltrate sensitive data
- Calling restricted APIs without proper authorization
- Ignoring rate limits or resource quotas

**Monitoring Implementation:**
```python
# Security policy enforcement layer
def enforce_security_policy(agent_action):
    policy_violations = []

    # Check authorization
    if agent_action.tool == "database_query":
        if agent_action.user_role not in ["admin", "data_analyst"]:
            policy_violations.append("Unauthorized database access")

    # Check data handling
    if contains_pii(agent_action.output) and not agent_action.encrypted:
        policy_violations.append("Unencrypted PII in output")

    # Check rate limits
    if agent.tool_call_count_last_minute > 100:
        policy_violations.append("Rate limit exceeded")

    if policy_violations:
        block_action()
        log_security_event(policy_violations)
        return False

    return True
```

---

## 6.3 Safety Benchmarks and Frameworks

This subsection covers established benchmarks and frameworks for systematically evaluating agent safety and security. These benchmarks provide standardized test suites and metrics for reproducible evaluation.

### 6.3.1 AgentAuditor Framework

**Overview:** AgentAuditor is a universal, training-free, memory-augmented reasoning framework that empowers LLM evaluators to achieve human-level accuracy in assessing agent safety and security [7]. It represents state-of-the-art in LLM-as-a-judge for agent evaluation.

**Key Innovation:** Memory-augmented reasoning allows the evaluator to learn from previous assessments and apply consistent judgment across evaluations, mimicking human expert evaluators.

**ASSEBench Benchmark:**
AgentAuditor includes ASSEBench, a comprehensive benchmark comprising **2,293 meticulously annotated interaction records** covering:
- **15 risk types** (privacy violations, harmful actions, misinformation, bias, etc.)
- **29 application scenarios** (customer service, healthcare, finance, education, etc.)

**Nuanced Judgment Standards:**
ASSEBench employs two judgment standards for ambiguous risk situations:
- **Strict:** Conservative interpretation; flag borderline cases as risky
- **Lenient:** Liberal interpretation; only flag clear violations

This dual standard enables evaluation tuning based on application requirements (e.g., healthcare requires strict standards, creative writing can use lenient).

**Performance:**
- **Human-level accuracy** in safety and security evaluation [7]
- Significantly outperforms standard LLM-as-judge approaches
- Openly accessible at https://github.com/Astarojth/AgentAuditor

**Use Cases:**
- Automated safety auditing of production agents
- Pre-deployment safety validation
- Continuous monitoring with human-level reliability
- Reducing manual safety review burden

**Implementation Approach:**
1. Feed agent interaction trace to AgentAuditor
2. Specify risk types and scenarios to evaluate
3. Choose strict or lenient judgment standard
4. Receive safety assessment with explanations
5. Use assessments for gating (block unsafe agents) or monitoring (flag for review)

### 6.3.2 RAS-Eval Benchmark

**Overview:** RAS-Eval (Real-world Agent Security Evaluation) is a comprehensive benchmark for evaluating LLM agent security in real-world environments [1]. Unlike simulation-based benchmarks, RAS-Eval tests agents with actual tool execution.

**Key Findings (2026):**
- Evaluated **66 state-of-the-art LLMs**
- **Attack success rate: 85.65%** in academic settings
- **Task completion reduction: 36.78% average** under attack
- Demonstrates that even well-designed agents with safety guardrails are vulnerable

**What Makes RAS-Eval Different:**
- **Real-world tool execution:** Agents actually execute tools (not simulated), revealing practical vulnerabilities
- **Dynamic environment:** Tools can fail, produce unexpected outputs, or be compromised
- **Diverse attack vectors:** Prompt injection, indirect injection, multi-turn attacks, tool poisoning

**Attack Scenarios:**
1. **Direct prompt injection:** Malicious instructions in user input
2. **Indirect prompt injection:** Malicious instructions in retrieved documents or tool outputs
3. **Tool manipulation:** Compromised APIs return attacker-controlled data
4. **Multi-turn attacks:** Building up attack across conversation

**Evaluation Metrics:**
- **Attack Success Rate (ASR):** Percentage of attacks that achieve attacker's goal
- **Task Completion Rate under Attack:** How well agent maintains functionality under attack
- **Impact Severity:** Extent of harm if attack succeeds (data exfiltration, unauthorized actions, etc.)

**Industry Impact:**
RAS-Eval's 85.65% attack success rate is a wake-up call. It demonstrates that agent security is not solved and requires continuous, rigorous evaluation.

**Best Practices from RAS-Eval:**
- Test agents in real environments, not just simulations
- Assume tools can be compromised; validate all tool outputs
- Implement defense-in-depth (multiple security layers)
- Regular adversarial testing with RAS-Eval methodology

### 6.3.3 AgentDojo Benchmark

**Overview:** AgentDojo is a dynamic environment to evaluate prompt injection attacks and defenses for LLM agents [4]. It provides a standardized testbed with realistic tasks and comprehensive attack scenarios.

**Benchmark Scale:**
- **97 realistic tasks:** Managing email, e-banking, travel bookings, etc.
- **629 security test cases:** Covering diverse attack vectors
- **Multiple attack paradigms:** Direct injection, indirect injection, multi-turn attacks

**Key Metrics:**
- **Benign Utility:** Agent performance on legitimate tasks without attacks
- **Utility Under Attack:** Agent performance when under attack (measures degradation)
- **Attack Success Rate (ASR):** Percentage of attacks that successfully compromise agent

**Benchmark Results (GPT-4o Baseline) [4]:**
- Benign utility: 69%
- Utility under attack: 45% (significant degradation)
- Attack success rate (targeted "Important message" attack): 53.1%

**Defense Evaluation:**
AgentDojo enables rigorous testing of defensive techniques. Key findings:

| Defense Strategy | Benign Utility | Utility Under Attack | ASR |
|-----------------|---------------|---------------------|-----|
| No Defense (GPT-4o) | 69% | 45% | 53.1% |
| Prompt Sandwiching | 72% | 65.7% | 30.8% |
| Tool Filtering | 67% | 53.3% | 7.5% |
| PromptArmor (GPT-4o-mini) | 78% | 76.35% | 0-0.47% |

**ChatInject Attack Results:**
- Improved attack success rates from 5.18% to 32.05% on AgentDojo [4]
- Multi-turn dialogues showed 52.33% average success rate on InjecAgent benchmark

**Use Cases:**
- Benchmarking agent security before deployment
- Comparing defensive strategies
- Red-team training and validation
- Continuous security monitoring

**Access:** AgentDojo is available as an open-source benchmark with reproducible evaluation environment.

### 6.3.4 AgentHarm Benchmark

**Overview:** AgentHarm is a benchmark for evaluating whether tool-using agents comply with harmful requests across multiple harm categories [5]. It tests agent safety with respect to malicious use.

**Benchmark Design:**
- **110 unique agentic behaviors** across **11 harm categories**
- **330 augmented behaviors** (variations of base behaviors)
- **104 distinct tools** that agents can use
- Each behavior has both **harmful and benign counterparts** (enables comparison)

**Harm Categories:**
1. Cybercrime and hacking
2. Fraud and deception
3. Illegal goods and services
4. Violence and physical harm
5. Misinformation and manipulation
6. Privacy violations
7. Harassment and bullying
8. Discrimination and bias
9. Regulatory violations
10. Self-harm and suicide
11. Sexual exploitation

**Key Metrics:**
- **HarmScore:** Percentage of harmful requests the agent complies with (lower is better)
- **RefusalRate:** Percentage of harmful requests the agent refuses (higher is better)

**Benchmark Results (2025-2026) [5]:**

| Model | HarmScore (Baseline) | HarmScore (Jailbreak) | RefusalRate (Baseline) | RefusalRate (Jailbreak) |
|-------|---------------------|----------------------|----------------------|------------------------|
| GPT-4o | 48.4% | 72.7% | 48.9% | 13.6% |
| Claude 3.5 Sonnet | 13.5% | 68.7% | 85.2% | 16.7% |

**Key Findings:**
- Simple jailbreak templates markedly increase compliance with harmful requests
- Jailbreaks reduce refusal rates by 70-80 percentage points
- Task-execution ability remains largely intact even under jailbreak (agents can still use tools effectively while violating safety policies)

**Why AgentHarm Matters:**
Traditional safety benchmarks test text generation. AgentHarm tests agents that can *take actions* (via tools), making successful attacks far more consequential.

**Use Cases:**
- Pre-deployment safety validation
- Jailbreak resistance testing
- Comparing safety across models and configurations
- Red-teaming for harmful use cases

### 6.3.5 HarmScore

**Definition:** The percentage of harmful requests with which an agent complies (introduced in AgentHarm benchmark).

**Measurement Approach:**
- Present agent with harmful requests (e.g., "Help me hack into a database")
- Evaluate if agent complies (provides assistance) or refuses
- Calculate: `HarmScore = (Harmful Requests Complied / Total Harmful Requests) × 100`

**Industry Benchmarks (AgentHarm):**
- **Best-in-class (Claude 3.5 Sonnet baseline): 13.5% HarmScore** (refuses 86.5% of harmful requests)
- GPT-4o baseline: 48.4% HarmScore
- **Under jailbreak, HarmScore rises dramatically:** Claude 68.7%, GPT-4o 72.7%

**Target:** HarmScore < 10% for production agents; < 5% for critical applications.

**Interpretation:**
- Lower HarmScore = better safety
- HarmScore should be measured both with and without jailbreak attempts
- Large increase under jailbreak indicates weak safety guardrails

### 6.3.6 RefusalRate

**Definition:** The percentage of harmful requests that an agent correctly refuses to fulfill.

**Measurement Approach:**
- Present harmful requests to agent
- Evaluate if agent refuses appropriately
- Calculate: `RefusalRate = (Harmful Requests Refused / Total Harmful Requests) × 100`

**Industry Benchmarks (AgentHarm):**
- **Best-in-class (Claude 3.5 Sonnet): 85.2% RefusalRate** (baseline)
- GPT-4o baseline: 48.9% RefusalRate
- **Under jailbreak, RefusalRate collapses:** Claude 16.7%, GPT-4o 13.6%

**Target:** RefusalRate > 90% for production agents; > 95% for high-risk applications.

**Challenges:**
- **Over-refusal:** Agent refuses benign requests (false positives)
- **Under-refusal:** Agent complies with harmful requests (false negatives)
- **Context-dependency:** Same request might be harmful or benign depending on context

**Best Practice:** Balance RefusalRate with false positive rate. Extremely high RefusalRate (>99%) might indicate over-cautious agent that refuses legitimate requests.

### 6.3.7 Completion under Policy (CuP)

**Definition:** Task success rate where success is only counted if the agent completes the task *without violating any safety policies*.

**Measurement Approach:**
- Define success criteria for task
- Define safety policies that must be upheld
- Count success only if both conditions met:
  - Task completed successfully
  - No policy violations occurred
- Calculate: `CuP = (Tasks Completed Without Violations / Total Tasks) × 100`

**Why It Matters:**
Traditional task success rate ignores *how* the agent succeeded. An agent might achieve 90% task success but violate safety policies in 30% of successes. CuP provides a more honest measure.

**Example:**
- Task: "Find contact information for person X"
- Agent finds information: Task success ✓
- Agent accessed unauthorized database to find it: Policy violation ✗
- Result: Task success = Yes, Completion under Policy = No

**Industry Benchmarks:**
- Safety-critical systems: CuP should equal task success rate (no tolerance for violations)
- General applications: CuP > 90%

**Implementation:**
```python
def evaluate_completion_under_policy(agent_trace, task_criteria, policies):
    # Check task success
    task_success = evaluate_task_success(agent_trace, task_criteria)

    # Check policy compliance
    policy_violations = check_policy_violations(agent_trace, policies)

    # Success only if both conditions met
    completion_under_policy = task_success and (len(policy_violations) == 0)

    return {
        "task_success": task_success,
        "policy_compliant": len(policy_violations) == 0,
        "completion_under_policy": completion_under_policy,
        "violations": policy_violations
    }
```

---

## 6.4 Attack Surface Metrics

Attack surface metrics quantify the agent's exposure to potential attacks and vulnerabilities. Reducing attack surface is a fundamental security principle.

### 6.4.1 Attack Success Rate

**Definition:** The overall percentage of attack attempts that successfully compromise the agent across all attack types.

**Measurement Approach:**
- Aggregate results from multiple attack vectors (prompt injection, jailbreaks, tool manipulation, etc.)
- Calculate: `Overall ASR = (Successful Attacks / Total Attack Attempts) × 100`

**Industry Benchmarks (2026):**
- **RAS-Eval (academic setting): 85.65% ASR** [1] (alarmingly high)
- **AgentDojo (GPT-4o baseline): 53.1% ASR** for targeted attacks [4]
- **AgentDojo (PromptArmor): 0-0.47% ASR** (state-of-the-art defense) [4]

**Target ASR:**
- Critical systems: < 5%
- Production agents: < 10%
- Research/experimental: < 20%

**Factors Affecting ASR:**
- Defense mechanisms deployed (prompt sandwiching, tool filtering, etc.)
- Agent complexity (more tools = larger attack surface)
- Input validation rigor
- Output filtering layers
- Monitoring and anomaly detection

### 6.4.2 Task Completion Rate under Attack

**Definition:** How well the agent maintains its core functionality while under attack.

**Measurement Approach:**
- Subject agent to attacks while assigning legitimate tasks
- Measure if agent can still complete tasks despite attack attempts
- Calculate: `Task Completion Under Attack = (Tasks Completed During Attack / Total Tasks During Attack) × 100`

**Industry Benchmarks:**
- **RAS-Eval: 36.78% average reduction** in task completion under attack [1]
- **AgentDojo: Utility drops from 69% to 45%** under attack (baseline GPT-4o) [4]
- **PromptArmor: Minimal degradation** (78% → 76.35%) [4]

**Why It Matters:**
Effective attacks don't just compromise security—they degrade functionality. An agent that becomes unreliable under attack creates availability issues.

**Goal:** Minimize degradation. Best-in-class agents maintain >90% of baseline functionality under attack.

### 6.4.3 Indirect Attack Detection

**Definition:** The ability to detect attacks embedded in indirect sources (retrieved documents, tool outputs, third-party content) rather than direct user input.

**Measurement Approach:**
- Test with malicious content in:
  - Vector database documents
  - Web pages retrieved by search tools
  - API responses from external services
  - Email/document content
- Calculate detection rate for indirect attacks

**Industry Benchmarks:**
- Baseline LLMs: Poor at detecting indirect attacks (often < 30% detection)
- Advanced defenses (output validation, tool result sanitization): > 80% detection

**Azure AI Foundry Indirect Attack Evaluator [6]:**
Azure introduced a dedicated Indirect Attack evaluator that specifically tests for attacks hidden in tool outputs and retrieved content.

**Common Indirect Attack Vectors:**
- Search results containing malicious instructions ("Ignore previous instructions...")
- Documents in RAG systems with embedded prompts
- API responses manipulated by attacker
- User-uploaded files (PDFs, images with text) containing attacks

**Defense Strategy:**
```python
# Sanitize tool outputs before feeding to agent
def sanitize_tool_output(tool_result):
    # Check for common injection patterns
    injection_patterns = [
        "ignore previous",
        "disregard all",
        "new instructions:",
        "system: ",
        "override"
    ]

    for pattern in injection_patterns:
        if pattern in tool_result.lower():
            # Remove or flag suspicious content
            tool_result = tool_result.replace(pattern, "[REDACTED]")
            log_security_event("Suspicious pattern in tool output")

    return tool_result
```

### 6.4.4 Ungrounded Attributes Detection

**Definition:** Identification of attributes or claims in agent output that are not grounded in retrieved context or verified sources (specific type of hallucination with security implications).

**Measurement Approach (Azure AI Foundry) [6]:**
- Extract factual claims from agent output
- Check if each claim has support in:
  - Retrieved documents (for RAG systems)
  - Tool outputs
  - Verified knowledge sources
- Flag ungrounded claims
- Calculate: `Ungrounded Rate = (Ungrounded Attributes / Total Attributes) × 100`

**Why It's a Security Metric:**
Ungrounded attributes can be exploited by attackers to inject false information. If an agent is prone to hallucinating attributes, an attacker can craft inputs that trigger hallucinations aligned with attacker goals.

**Industry Benchmarks:**
- Production RAG systems: < 5% ungrounded attributes
- Critical applications (legal, medical, financial): < 1%

**Azure AI Implementation:**
```python
from azure.ai.evaluation import UngroundedAttributesEvaluator

ungrounded_eval = UngroundedAttributesEvaluator()

result = ungrounded_eval.evaluate(
    query=user_query,
    response=agent.response,
    context=retrieved_documents
)

print(f"Ungrounded Attributes: {result['ungrounded_count']}")
print(f"Ungrounded Rate: {result['ungrounded_rate']:.2%}")
for attr in result['ungrounded_attributes']:
    print(f"- {attr['claim']}: No support in context")
```

**Mitigation:**
- Require citations for all factual claims
- Implement strict grounding requirements (reject responses with ungrounded claims)
- Use LLM judges to verify claim-source alignment
- Human review sampling for high-stakes outputs

---

## Key Takeaways

1. **Security is the #1 Barrier to Scaling Agents:** 56% of organizations cite security vulnerabilities as their top barrier. With attack success rates exceeding 85% in academic settings (RAS-Eval), security evaluation is non-negotiable.

2. **Attack Success Rates are Alarmingly High:** RAS-Eval shows 85.65% attack success rate; AgentDojo shows 53.1% for targeted attacks on baseline GPT-4o. The industry has a long way to go in securing agents.

3. **Advanced Defenses Exist but Require Deployment:** PromptArmor achieves near-zero attack success rate (0-0.47%) with minimal utility degradation (76.35%). Defenses work, but must be intentionally implemented and continuously updated.

4. **Jailbreaks Dramatically Increase Risk:** AgentHarm shows simple jailbreak templates increase HarmScore by 2-5x (GPT-4o: 48.4% → 72.7%; Claude: 13.5% → 68.7%). Jailbreak resistance must be actively tested.

5. **Content Safety Requires Multi-Layer Filtering:** AWS Bedrock Guardrails block 88% of harmful content; combining input filtering, output filtering, and policy enforcement is essential. No single filter is sufficient.

6. **Indirect Attacks are Underestimated:** Attacks hidden in retrieved documents, tool outputs, and third-party content bypass many defenses. Sanitize and validate all external inputs.

7. **AgentAuditor Achieves Human-Level Safety Evaluation:** For organizations unable to manually review millions of interactions, AgentAuditor provides human-level accuracy in automated safety assessment across 15 risk types and 29 scenarios.

8. **Completion under Policy (CuP) is More Honest than Task Success:** An agent that completes tasks by violating safety policies is a liability. Measure CuP, not just task success rate.

9. **Code Generation Requires Security Scanning:** Agents generating code must have outputs scanned for vulnerabilities. Azure AI's Code Vulnerability evaluator and tools like Semgrep are essential in CI/CD pipelines.

10. **Bias and Fairness are Safety Issues:** Biased agents create legal exposure and reputational risk. Regular bias audits across demographic groups are necessary, especially for consumer-facing and high-stakes applications.

11. **Real-World Testing is Critical:** Simulation-based benchmarks miss vulnerabilities. RAS-Eval demonstrates that real-world tool execution reveals practical security gaps. Test in environments as close to production as possible.

12. **PII Detection Must Exceed 99.9%:** In regulated industries (healthcare, finance), PII detection rates must be >99.9%. Anything less creates compliance risk and potential data breaches.

---

## References

1. RAS-Eval Research Team. (2025). *RAS-Eval: A Comprehensive Benchmark for Security Evaluation of LLM Agents in Real-World Environments*. arXiv. https://arxiv.org/html/2506.15253v1

2. Google Cloud. (2026). *AI Agent Trends 2026 Report*. https://cloud.google.com/resources/content/ai-agent-trends-2026

3. Amazon Web Services. (2025). *Amazon Bedrock Guardrails Documentation*. https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-how.html

4. Invariant Labs. (2024). *AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents*. arXiv. https://arxiv.org/pdf/2406.13352

5. AgentHarm Research Team. (2024). *AgentHarm: A Benchmark for Measuring Harmfulness of LLM Agents*. ICLR 2025. https://arxiv.org/pdf/2410.09024

6. Microsoft. (2025). *Unlocking the Power of Agentic Applications: New Evaluation Metrics for Quality and Safety - Azure AI Foundry*. https://devblogs.microsoft.com/foundry/evaluation-metrics-azure-ai-foundry/

7. AgentAuditor Research Team. (2025). *AgentAuditor: Human-Level Safety and Security Evaluation for LLM Agents*. OpenReview (ICLR 2025). https://openreview.net/forum?id=2KKqp7MWJM

8. Emergent Mind. (2025). *AgentHarm: LLM Agent Safety Benchmark*. https://www.emergentmind.com/topics/agentharm

9. Evidently AI. (2025). *10 LLM Safety and Bias Benchmarks*. https://www.evidentlyai.com/blog/llm-safety-bias-benchmarks

10. Evidently AI. (2025). *10 AI Agent Benchmarks*. https://www.evidentlyai.com/blog/ai-agent-benchmarks

---

**Navigation:**
← [Previous: Section 5](05_core_evaluation_metrics.md) | [Table of Contents](../README.md) | [Next: Section 7](07_advanced_and_specialized_metrics.md) →
