# Section 8: Defining Custom Metrics

> **Part IV**: Metrics & Measurements
> **Estimated Reading Time**: 20 minutes

## Overview

While standard metrics (task success, latency, cost) provide foundational insights, every AI agent deployment has unique success criteria shaped by business context, domain requirements, and user expectations. Custom metrics bridge the gap between generic evaluation frameworks and specific organizational needs.

This section provides a comprehensive guide to creating custom metrics: when they're necessary, how to design effective composite metrics, domain-specific metric development strategies, implementation approaches (code-based evaluators and natural language definitions), and weighted scoring frameworks like the CLASSic approach. By the end, you'll have practical methodologies to define, implement, and validate custom metrics that align evaluation with your actual definition of success.

## Table of Contents

- [8.1 When and Why to Create Custom Metrics](#81-when-and-why-to-create-custom-metrics)
- [8.2 Composite Metrics Design](#82-composite-metrics-design)
- [8.3 Domain-Specific Metric Development](#83-domain-specific-metric-development)
- [8.4 Implementing Custom Metrics](#84-implementing-custom-metrics)
- [8.5 Weighted Scoring Frameworks](#85-weighted-scoring-frameworks)
- [Key Takeaways](#key-takeaways)
- [References](#references)

---

## 8.1 When and Why to Create Custom Metrics

### When Standard Metrics Fall Short

Standard metrics like task success rate, latency, and cost provide baseline insights but often miss dimensions critical to your specific use case. Consider creating custom metrics when:

**1. Business Goals Aren't Captured**

Standard metrics measure generic success, but your business may care about specific outcomes.

**Examples:**
- E-commerce agent: Standard metrics don't capture upsell success rate or average order value impact
- Healthcare triage agent: Generic accuracy doesn't measure appropriate urgency escalation
- Legal research agent: Task completion doesn't reflect citation quality or precedent relevance

**2. Domain-Specific Quality Criteria Exist**

Your domain has established quality standards that generic metrics don't address.

**Examples:**
- Journalism agents: Adherence to AP Style, fact-checking rigor, source diversity
- Scientific research agents: Peer-review citation rate, experimental reproducibility
- Education agents: Pedagogical appropriateness, grade-level alignment, Bloom's taxonomy coverage

**3. User Experience Nuances Matter**

Your users have specific expectations about tone, style, or interaction patterns.

**Examples:**
- Luxury brand agent: Sophistication and elegance of language (not measured by standard clarity metrics)
- Children's education agent: Age-appropriate vocabulary and engagement
- Crisis support agent: Empathy, de-escalation effectiveness, appropriate resource referrals

**4. Regulatory or Compliance Requirements**

Your industry has specific compliance needs that must be measured.

**Examples:**
- Financial services: Suitability assessment, risk disclosure completeness
- Healthcare: HIPAA compliance scoring, appropriate medical disclaimers
- Legal: Conflict of interest detection, privilege protection

**5. Existing Metrics Are Too Coarse**

Standard metrics aggregate behavior in ways that hide important distinctions.

**Examples:**
- Task success rate treats all successes equally, but you care about *how* tasks are completed
- Latency averages hide the difference between consistently fast and inconsistently varying response times
- Cost per interaction doesn't distinguish between high-value and low-value interactions

### Decision Framework: Custom Metric vs. Standard Metric

Use this framework to decide whether to create a custom metric:

```
Decision Tree:
├── Does a standard metric directly measure this dimension?
│   ├── Yes → Use standard metric
│   └── No → Continue
├── Can the dimension be measured by combining existing metrics?
│   ├── Yes → Create composite metric (Section 8.2)
│   └── No → Continue
├── Is this dimension critical to success in your use case?
│   ├── No → Deprioritize; use standard metrics
│   └── Yes → Continue
├── Can you define clear success criteria for this dimension?
│   ├── No → Refine definition before creating metric
│   └── Yes → Create custom metric
```

### Cost-Benefit Analysis

Creating custom metrics has costs:
- **Development time:** Writing evaluators, calibrating judges, validating
- **Computational cost:** Running additional evaluations increases expense
- **Maintenance burden:** Custom metrics need updating as requirements evolve

Justify custom metrics when:
- **High business impact:** Dimension directly affects key business outcomes
- **Frequent use:** Metric will be used across many evaluations
- **Unique insight:** Custom metric reveals information not available elsewhere
- **Regulatory requirement:** Compliance mandates specific measurement

**Rule of Thumb:** If a dimension affects >20% of business value, create a dedicated metric for it.

---

## 8.2 Composite Metrics Design

Composite metrics combine multiple individual metrics into a single aggregate score, providing a holistic view of agent performance.

### Benefits of Composite Metrics

1. **Simplified Reporting:** Single score easier to communicate to stakeholders than 20 individual metrics
2. **Trade-off Visibility:** Weighted combinations make performance trade-offs explicit
3. **Goal Alignment:** Weights encode organizational priorities
4. **Threshold Setting:** Single score simplifies gating decisions (e.g., "Agent must score >0.80 to deploy")

### Design Principles

**Principle 1: Balanced Dimensions**

Include metrics across multiple evaluation dimensions to avoid over-optimization on one aspect.

**Example - Customer Service Agent Quality Score:**
```
Quality Score = 0.3×Task_Success + 0.25×CSAT + 0.2×Response_Time + 0.15×Tone_Appropriateness + 0.1×Policy_Compliance
```

This balances task completion (30%), user satisfaction (25%), efficiency (20%), interaction quality (15%), and safety (10%).

**Principle 2: Normalized Components**

Ensure all component metrics are on the same scale (typically 0-1) before combining.

**Normalization Example:**
```python
def normalize_metric(value, min_acceptable, max_possible):
    """Normalize metric to 0-1 scale"""
    return (value - min_acceptable) / (max_possible - min_acceptable)

# Example: Normalize latency (lower is better)
latency_ms = 800
normalized_latency = 1 - normalize_metric(
    value=latency_ms,
    min_acceptable=0,
    max_possible=2000  # 2 seconds is unacceptable
)
# Result: 1 - (800/2000) = 0.6

# Example: Normalize task success (higher is better)
task_success = 0.85
normalized_success = normalize_metric(
    value=task_success,
    min_acceptable=0,
    max_possible=1.0
)
# Result: 0.85 (already normalized)
```

**Principle 3: Data-Driven Weighting**

Set weights based on:
- **Business impact:** Dimensions with higher revenue/cost impact get higher weights
- **User priorities:** Survey users on what matters most
- **Empirical correlation:** Analyze which dimensions correlate most with overall success
- **Domain expertise:** Consult subject matter experts

**Principle 4: Interpretability**

Keep the formula simple enough that stakeholders understand it.

**Good (Interpretable):**
```
Agent Performance = 0.4×Accuracy + 0.3×Speed + 0.3×Cost_Efficiency
```

**Bad (Opaque):**
```
Agent Performance = Σ(wi × fi(xi)^αi × log(βi)) where...
```

### Common Composite Metric Patterns

#### Pattern 1: Weighted Average

Most straightforward approach. Combine normalized metrics with weights summing to 1.

```python
def weighted_average_composite(metrics, weights):
    """
    metrics: dict of normalized metric values (0-1)
    weights: dict of weights (sum to 1.0)
    """
    assert abs(sum(weights.values()) - 1.0) < 0.01, "Weights must sum to 1"

    score = sum(metrics[key] * weights[key] for key in metrics)
    return score

# Usage
metrics = {
    "task_success": 0.85,
    "latency_normalized": 0.75,
    "cost_efficiency": 0.90,
    "safety_score": 0.95
}

weights = {
    "task_success": 0.40,
    "latency_normalized": 0.25,
    "cost_efficiency": 0.20,
    "safety_score": 0.15
}

overall_score = weighted_average_composite(metrics, weights)
# Result: 0.8425
```

#### Pattern 2: Minimum Threshold Composite

Requires all components to meet minimum thresholds, then averages excess performance.

```python
def threshold_composite(metrics, thresholds, weights):
    """
    Composite score where all metrics must exceed thresholds.
    Returns 0 if any metric below threshold.
    """
    # Check all thresholds met
    for metric, threshold in thresholds.items():
        if metrics[metric] < threshold:
            return 0.0  # Fail if any threshold missed

    # Calculate weighted average of excess performance
    excess_performance = {
        key: (metrics[key] - thresholds[key]) / (1.0 - thresholds[key])
        for key in metrics
    }

    score = sum(excess_performance[key] * weights[key] for key in metrics)
    return score

# Usage
metrics = {"accuracy": 0.92, "safety": 0.88, "speed": 0.75}
thresholds = {"accuracy": 0.85, "safety": 0.90, "speed": 0.70}
weights = {"accuracy": 0.5, "safety": 0.3, "speed": 0.2}

score = threshold_composite(metrics, thresholds, weights)
# Result: 0 (safety below threshold of 0.90)
```

**Use Case:** High-stakes applications where minimum standards are non-negotiable (e.g., safety must be >0.90).

#### Pattern 3: Harmonic Mean

Penalizes imbalance more than arithmetic mean. Useful when you want balanced performance across all dimensions.

```python
def harmonic_mean_composite(metrics, weights):
    """
    Harmonic mean composite - heavily penalizes low scores in any dimension
    """
    weighted_reciprocals = sum(weights[key] / metrics[key] for key in metrics)
    harmonic_mean = 1.0 / weighted_reciprocals
    return harmonic_mean

# Usage
metrics = {"accuracy": 0.95, "speed": 0.90, "cost": 0.50}  # Note: cost is poor
weights = {"accuracy": 0.33, "speed": 0.33, "cost": 0.34}

# Harmonic mean
h_mean = harmonic_mean_composite(metrics, weights)
# Result: ~0.67 (heavily penalized by low cost score)

# Compare to arithmetic mean
a_mean = sum(metrics[k] * weights[k] for k in metrics)
# Result: 0.783 (less penalty)
```

**Use Case:** When you need balanced performance and want to discourage "gaming" by excelling in one dimension while neglecting others.

#### Pattern 4: Gated Composite

Different weights or formulas for different agent maturity levels.

```python
def gated_composite(metrics, maturity_level):
    """
    Different weighting schemes based on maturity
    """
    weights = {
        "prototype": {
            "accuracy": 0.60,  # Prioritize correctness early
            "speed": 0.10,     # Don't worry about speed yet
            "cost": 0.10,      # Don't worry about cost yet
            "safety": 0.20     # Basic safety
        },
        "production": {
            "accuracy": 0.30,  # Correctness still important but balanced
            "speed": 0.25,     # Speed matters in production
            "cost": 0.25,      # Cost matters at scale
            "safety": 0.20     # Safety remains important
        }
    }

    w = weights[maturity_level]
    score = sum(metrics[key] * w[key] for key in metrics)
    return score
```

### Case Study: E-commerce Agent Quality Score

**Business Context:** E-commerce recommendation agent must balance accuracy, user experience, and business metrics.

**Custom Composite Metric:**
```python
E-commerce Agent Score = (
    0.25 × Product_Relevance +
    0.20 × User_Engagement +
    0.20 × Conversion_Impact +
    0.15 × Response_Quality +
    0.10 × Cost_Efficiency +
    0.10 × Safety_Compliance
)

Where:
- Product_Relevance: How well recommended products match user intent (0-1)
- User_Engagement: Click-through rate on recommendations (normalized)
- Conversion_Impact: Lift in conversion rate vs. baseline (normalized)
- Response_Quality: Helpfulness + Clarity (LLM judge, 0-1)
- Cost_Efficiency: 1 - (Actual_Cost / Budget)
- Safety_Compliance: PII detection + Inappropriate content filtering (0-1)
```

**Rationale for Weights:**
- Product relevance (25%): Core value proposition
- User engagement (20%): Leading indicator of value
- Conversion impact (20%): Direct business outcome
- Response quality (15%): User experience
- Cost efficiency (10%): Must be economical at scale
- Safety (10%): Table stakes, non-negotiable minimum

**Deployment Threshold:** Agent must score ≥ 0.75 to deploy to production.

---

## 8.3 Domain-Specific Metric Development

Domain-specific metrics capture success criteria unique to particular industries or applications.

### Process for Developing Domain-Specific Metrics

**Step 1: Identify Domain Success Criteria**

Engage domain experts to define what "good" looks like in your context.

**Methods:**
- Stakeholder interviews (product managers, subject matter experts, end users)
- Analyze existing quality assurance rubrics
- Review regulatory guidelines and industry standards
- Study failure cases (what does "bad" look like?)

**Example - Legal Research Agent:**
Expert interviews reveal success criteria:
- Cites binding precedents from relevant jurisdiction
- Distinguishes binding vs. persuasive authority
- Identifies overruled or questioned precedents
- Provides case context (holding, reasoning, factual similarity)

**Step 2: Operationalize Criteria into Measurable Metrics**

Translate qualitative criteria into quantifiable metrics.

**Legal Research Example:**
```python
# Custom metric: Precedent Quality Score
def precedent_quality_score(cited_cases, query_jurisdiction, query_topic):
    score = 0
    total_weight = 0

    for case in cited_cases:
        weight = 1.0

        # Binding precedent from correct jurisdiction
        if case.jurisdiction == query_jurisdiction and case.binding:
            weight += 0.5

        # Precedent is still good law (not overruled)
        if case.status == "good_law":
            weight += 0.3
        elif case.status == "overruled":
            weight = 0  # Citing overruled cases is bad

        # Topical relevance
        relevance = calculate_topical_relevance(case.topic, query_topic)
        weight *= relevance

        # Case recency (recent cases weighted higher for evolving areas)
        recency_weight = calculate_recency_weight(case.year)
        weight *= recency_weight

        score += weight
        total_weight += 1.0

    return score / total_weight if total_weight > 0 else 0
```

**Step 3: Validate with Domain Experts**

Have experts review metric output on sample cases.

**Validation Process:**
1. Run metric on 50-100 test cases
2. Have experts independently score same cases
3. Calculate correlation between metric and expert scores
4. Target: Correlation > 0.80
5. If correlation low, refine metric and iterate

**Step 4: Establish Benchmarks**

Determine acceptable score ranges through empirical testing.

**Approach:**
- Evaluate high-performing human baselines (top 10% of human experts)
- Test on diverse cases (easy, medium, hard)
- Set benchmarks:
  - **Minimum acceptable:** 25th percentile of human performance
  - **Good performance:** 50th percentile (median human)
  - **Excellent:** 75th percentile or higher

### Domain-Specific Metric Examples

#### Healthcare: Clinical Appropriateness Score

**Criteria:**
- Recommendations align with clinical guidelines (e.g., AHA, AAP)
- Appropriate urgency escalation (emergency vs. urgent vs. routine)
- Considers patient contraindications and history
- Provides evidence-based treatment options

**Metric Implementation:**
```python
def clinical_appropriateness_score(agent_recommendation, patient_context):
    score_components = {}

    # Check guideline alignment
    guidelines = retrieve_relevant_guidelines(patient_context.condition)
    guideline_alignment = calculate_alignment(agent_recommendation, guidelines)
    score_components["guideline_alignment"] = guideline_alignment

    # Check urgency appropriateness
    recommended_urgency = agent_recommendation.urgency
    expected_urgency = clinical_decision_tree(patient_context.symptoms)
    urgency_appropriate = (recommended_urgency == expected_urgency)
    score_components["urgency_match"] = 1.0 if urgency_appropriate else 0.0

    # Check contraindication awareness
    if check_contraindications(agent_recommendation, patient_context.allergies):
        score_components["contraindication_check"] = 0.0  # FAIL
    else:
        score_components["contraindication_check"] = 1.0

    # Check evidence quality
    evidence_quality = evaluate_evidence_base(agent_recommendation.citations)
    score_components["evidence_quality"] = evidence_quality

    # Weighted composite
    weights = {
        "guideline_alignment": 0.35,
        "urgency_match": 0.30,
        "contraindication_check": 0.25,  # High weight - safety critical
        "evidence_quality": 0.10
    }

    total_score = sum(score_components[k] * weights[k] for k in weights)
    return total_score, score_components
```

#### Journalism: Article Quality Score

**Criteria:**
- Adherence to style guide (AP, Chicago, NYT)
- Source diversity and quality
- Fact verification against multiple sources
- Balanced presentation of perspectives
- Headline accuracy (doesn't overpromise)

**Metric:**
```python
def journalism_quality_score(article, style_guide="AP"):
    scores = {}

    # Style guide compliance
    style_violations = check_style_guide(article.text, style_guide)
    scores["style"] = 1.0 - (len(style_violations) / len(article.sentences))

    # Source quality
    source_score = evaluate_sources(
        article.citations,
        criteria=["credibility", "diversity", "recency"]
    )
    scores["sources"] = source_score

    # Fact verification
    facts = extract_factual_claims(article.text)
    verified = count_verified_facts(facts, article.citations)
    scores["fact_verification"] = verified / len(facts)

    # Balance
    perspectives = identify_perspectives(article.text)
    balance_score = calculate_perspective_balance(perspectives)
    scores["balance"] = balance_score

    # Headline accuracy
    headline_claims = extract_claims(article.headline)
    body_supports = check_headline_support(headline_claims, article.text)
    scores["headline_accuracy"] = body_supports / len(headline_claims)

    # Composite
    weights = {"style": 0.15, "sources": 0.25, "fact_verification": 0.30,
               "balance": 0.20, "headline_accuracy": 0.10}

    overall = sum(scores[k] * weights[k] for k in weights)
    return overall, scores
```

#### Education: Pedagogical Alignment Score

**Criteria:**
- Grade-level appropriate vocabulary and concepts
- Alignment with learning objectives
- Scaffolding and progressive difficulty
- Engagement and interactivity
- Formative assessment integration

**Metric:**
```python
def pedagogical_alignment_score(lesson_content, grade_level, learning_objectives):
    scores = {}

    # Grade-level appropriateness
    readability = calculate_readability(lesson_content.text)
    expected_range = get_grade_level_range(grade_level)
    scores["grade_appropriate"] = 1.0 if expected_range[0] <= readability <= expected_range[1] else 0.5

    # Learning objective alignment
    content_concepts = extract_concepts(lesson_content.text)
    objective_concepts = extract_concepts(learning_objectives)
    coverage = len(content_concepts.intersection(objective_concepts)) / len(objective_concepts)
    scores["objective_alignment"] = coverage

    # Scaffolding quality
    difficulty_progression = analyze_difficulty_progression(lesson_content.sections)
    scores["scaffolding"] = difficulty_progression.smoothness_score

    # Engagement elements
    interactive_elements = count_elements(lesson_content, types=["question", "activity", "example"])
    scores["engagement"] = min(interactive_elements / 5, 1.0)  # Target: 5+ per lesson

    # Assessment integration
    formative_checks = count_formative_assessments(lesson_content)
    scores["assessment"] = min(formative_checks / 3, 1.0)  # Target: 3+ per lesson

    # Composite
    weights = {"grade_appropriate": 0.25, "objective_alignment": 0.30,
               "scaffolding": 0.20, "engagement": 0.15, "assessment": 0.10}

    overall = sum(scores[k] * weights[k] for k in weights)
    return overall, scores
```

---

## 8.4 Implementing Custom Metrics

Custom metrics can be implemented through code-based evaluators or natural language definitions (LLM-as-a-judge).

### 8.4.1 Code-Based Evaluators

Code-based evaluators use programmatic logic to calculate metrics. Best for objective, rule-based criteria.

**When to Use Code-Based:**
- Criteria are objective and well-defined
- Evaluation requires structured data (e.g., database queries, API calls)
- Performance and cost efficiency are priorities (faster and cheaper than LLM judges)
- Deterministic evaluation is required (same input always produces same score)

**Implementation Pattern:**
```python
from deepeval.metrics import BaseMetric
from deepeval.test_case import LLMTestCase

class CustomDomainMetric(BaseMetric):
    def __init__(self, threshold: float = 0.7):
        self.threshold = threshold
        self.score = None
        self.reason = None

    def measure(self, test_case: LLMTestCase):
        # Extract relevant data from test case
        output = test_case.actual_output
        context = test_case.retrieval_context
        expected = test_case.expected_output

        # Implement evaluation logic
        score = self._calculate_score(output, context, expected)

        # Store results
        self.score = score
        self.success = score >= self.threshold
        self.reason = self._generate_reason(score)

        return self.score

    def _calculate_score(self, output, context, expected):
        # Your custom scoring logic here
        score_components = {}

        # Example: Check citation quality
        citations = extract_citations(output)
        score_components["citation_count"] = min(len(citations) / 3, 1.0)

        # Example: Check factual alignment
        facts = extract_facts(output)
        verified = verify_against_context(facts, context)
        score_components["factual_accuracy"] = verified / len(facts)

        # Weighted average
        weights = {"citation_count": 0.3, "factual_accuracy": 0.7}
        total_score = sum(score_components[k] * weights[k] for k in weights)

        return total_score

    def _generate_reason(self, score):
        if score >= 0.9:
            return "Excellent performance across all dimensions"
        elif score >= self.threshold:
            return "Meets minimum quality standards"
        else:
            return "Below threshold; needs improvement"

    def is_successful(self):
        return self.success

    @property
    def __name__(self):
        return "Custom Domain Metric"
```

**Usage:**
```python
from deepeval import evaluate

# Create metric instance
custom_metric = CustomDomainMetric(threshold=0.75)

# Create test case
test_case = LLMTestCase(
    input="Query here",
    actual_output=agent_output,
    retrieval_context=retrieved_docs,
    expected_output=expected_result
)

# Evaluate
custom_metric.measure(test_case)
print(f"Score: {custom_metric.score}")
print(f"Success: {custom_metric.success}")
print(f"Reason: {custom_metric.reason}")
```

### 8.4.2 Natural Language Metric Definitions

Natural language definitions leverage LLM-as-a-judge to evaluate subjective or complex criteria.

**When to Use LLM-as-Judge:**
- Criteria are subjective (tone, style, appropriateness)
- Evaluation requires nuanced interpretation
- Rapid prototyping (easier than writing code logic)
- Criteria change frequently (easier to update prompts than code)

**Galileo AI's Natural Language Metrics [1]:**
Galileo allows defining custom metrics in plain English, which are automatically translated into LLM-based evaluators.

**Example:**
```
Metric Name: Brand Voice Consistency

Definition: Evaluate whether the agent's response aligns with our brand voice guidelines:
- Professional yet approachable tone
- Uses "we" language to create partnership
- Avoids jargon unless necessary, then explains it
- Demonstrates empathy and understanding
- Provides actionable next steps

Score 1-5 where:
5 = Perfect brand voice alignment
4 = Good alignment with minor deviations
3 = Acceptable but noticeable inconsistencies
2 = Poor alignment, significant deviations
1 = Does not reflect brand voice at all
```

**Implementation (DeepEval Style):**
```python
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCase

# Define custom metric with natural language
brand_voice_metric = GEval(
    name="Brand Voice Consistency",
    criteria="Evaluate brand voice alignment based on: professional yet approachable tone, uses 'we' language, avoids jargon, demonstrates empathy, provides actionable steps",
    evaluation_steps=[
        "Analyze the tone of the response for professionalism and approachability",
        "Check for partnership language ('we', 'let's', 'together')",
        "Identify any jargon and verify it's explained",
        "Assess empathy and understanding of user needs",
        "Verify presence of clear, actionable next steps",
        "Assign score 0-1 based on alignment across all dimensions"
    ],
    evaluation_params=[
        LLMTestCaseParams.ACTUAL_OUTPUT
    ],
    threshold=0.7
)

# Use the metric
test_case = LLMTestCase(
    input=user_query,
    actual_output=agent_response
)

brand_voice_metric.measure(test_case)
print(f"Brand Voice Score: {brand_voice_metric.score}")
```

**Best Practices for LLM-as-Judge Metrics:**
1. **Be specific:** Vague criteria lead to inconsistent evaluation
2. **Provide examples:** Include example responses at each score level
3. **Calibrate:** Validate judge against human expert ratings
4. **Use structured output:** Request JSON format for easier parsing
5. **Multiple runs:** Run same evaluation 3-5 times, average scores to reduce variance

### 8.4.3 Integration with Evaluation Frameworks

Once defined, integrate custom metrics into your evaluation pipeline.

**DeepEval Integration:**
```python
from deepeval import assert_test
from deepeval.test_case import LLMTestCase

@pytest.mark.parametrize("test_case", test_cases)
def test_agent_with_custom_metrics(test_case):
    assert_test(
        test_case,
        metrics=[
            task_success_metric,
            custom_domain_metric,  # Your custom metric
            brand_voice_metric,     # Another custom metric
            safety_metric
        ]
    )
```

**Continuous Evaluation:**
```python
# Run custom metrics on production traffic
def evaluate_production_interaction(interaction):
    test_case = LLMTestCase(
        input=interaction.user_query,
        actual_output=interaction.agent_response,
        retrieval_context=interaction.retrieved_docs
    )

    results = {}
    for metric in [custom_domain_metric, brand_voice_metric, safety_metric]:
        metric.measure(test_case)
        results[metric.__name__] = {
            "score": metric.score,
            "success": metric.success,
            "reason": metric.reason
        }

    log_metrics(interaction.id, results)

    # Alert if any metric below threshold
    if any(not metric.success for metric in [custom_domain_metric, brand_voice_metric, safety_metric]):
        trigger_alert(interaction.id, results)

    return results
```

---

## 8.5 Weighted Scoring Frameworks

Weighted scoring frameworks provide structured approaches to combining multiple metrics into overall assessments. The CLASSic framework is a prominent example from Aisera.

### 8.5.1 The CLASSic Framework (Aisera)

The CLASSic framework evaluates AI agents across five dimensions [2]:

- **C**ost
- **L**atency
- **A**ccuracy
- **S**ecurity
- **S**tability (the second 'S')

Each dimension is scored independently, then combined using configurable weights based on organizational priorities.

### 8.5.2 CLASSic Dimension Breakdown

**Cost (Weight: 15-25%)**

**Metrics:**
- Token cost per task
- API call expenses
- Infrastructure overhead
- Cost per successful completion

**Scoring:**
- Compare to budget or baseline
- Normalize: `Cost Score = 1 - (Actual Cost / Target Cost)`
- Cap at 0 if over budget

**Latency (Weight: 20-30%)**

**Metrics:**
- End-to-end latency (P50, P95, P99)
- Time-to-first-token
- Average response time

**Scoring:**
- Define acceptable latency targets (e.g., P95 < 1000ms)
- Normalize: `Latency Score = 1 - (Actual Latency / Max Acceptable Latency)`
- Cap at 0 if exceeds maximum

**Accuracy (Weight: 25-35%)**

**Metrics:**
- Task success rate
- Factual correctness
- Intent resolution
- Tool use accuracy

**Scoring:**
- Direct measurement (already 0-1 scale)
- Can use weighted combination if multiple accuracy metrics

**Security (Weight: 15-25%)**

**Metrics:**
- PII detection rate
- Policy compliance
- Attack resistance
- Harmful content generation rate (inverted)

**Scoring:**
- Most security metrics already 0-1
- Ensure higher scores mean more secure
- Security failures can be gating (score = 0 if critical violation)

**Stability (Weight: 10-20%)**

**Metrics:**
- Error rate (inverted)
- Uptime/availability
- Consistency across runs (1 - variance)
- Recovery rate

**Scoring:**
- Stability Score = 1 - Error Rate
- Or use composite of multiple reliability metrics

### 8.5.3 Default CLASSic Weighting

Aisera's recommended default weights [2]:

```python
class ClassicScore:
    DEFAULT_WEIGHTS = {
        "cost": 0.20,
        "latency": 0.25,
        "accuracy": 0.30,
        "security": 0.15,
        "stability": 0.10
    }

    def __init__(self, weights=None):
        self.weights = weights or self.DEFAULT_WEIGHTS
        assert abs(sum(self.weights.values()) - 1.0) < 0.01

    def calculate(self, metrics):
        """
        metrics: dict with keys matching dimensions
        Returns: composite score 0-1
        """
        score = sum(metrics[dim] * self.weights[dim] for dim in self.weights)
        return score

# Usage
metrics = {
    "cost": 0.85,       # 85% of budget target
    "latency": 0.78,    # 22% slower than target
    "accuracy": 0.92,   # 92% task success
    "security": 0.95,   # 95% security score
    "stability": 0.88   # 12% error rate
}

classic = ClassicScore()
overall_score = classic.calculate(metrics)
print(f"CLASSic Score: {overall_score:.2f}")  # 0.87
```

### 8.5.4 Customizing Weights for Use Cases

Different use cases warrant different weight distributions.

**High-Stakes Healthcare Agent:**
```python
healthcare_weights = {
    "cost": 0.10,       # Less important than safety
    "latency": 0.15,    # Important but not critical
    "accuracy": 0.40,   # CRITICAL - medical accuracy
    "security": 0.25,   # CRITICAL - HIPAA compliance
    "stability": 0.10
}
```

**Real-Time Voice Agent:**
```python
voice_weights = {
    "cost": 0.15,
    "latency": 0.40,    # CRITICAL - sub-1000ms required
    "accuracy": 0.25,
    "security": 0.10,
    "stability": 0.10
}
```

**Cost-Optimized Batch Processing Agent:**
```python
batch_weights = {
    "cost": 0.40,       # CRITICAL - cost efficiency at scale
    "latency": 0.10,    # Not time-sensitive
    "accuracy": 0.30,
    "security": 0.15,
    "stability": 0.05
}
```

**Enterprise Production Agent (Balanced):**
```python
enterprise_weights = {
    "cost": 0.20,
    "latency": 0.25,
    "accuracy": 0.30,
    "security": 0.15,
    "stability": 0.10
}
```

### 8.5.5 Gating Decisions with Weighted Scores

Use composite scores for deployment gating.

**Example Gating Logic:**
```python
def deployment_gate(classic_score, individual_metrics, min_score=0.80):
    """
    Determine if agent passes deployment gate
    """
    # Overall score must meet minimum
    if classic_score < min_score:
        return False, f"Overall score {classic_score:.2f} below {min_score}"

    # Individual dimensions must meet minimums (hard requirements)
    hard_requirements = {
        "accuracy": 0.85,   # Must be at least 85% accurate
        "security": 0.90,   # Must be at least 90% secure
        "stability": 0.75   # Must be at least 75% stable
    }

    for dimension, min_threshold in hard_requirements.items():
        if individual_metrics[dimension] < min_threshold:
            return False, f"{dimension} score {individual_metrics[dimension]:.2f} below required {min_threshold}"

    return True, "Passed all deployment gates"

# Usage
classic_score = 0.87
metrics = {"cost": 0.85, "latency": 0.78, "accuracy": 0.92, "security": 0.95, "stability": 0.88}

passed, message = deployment_gate(classic_score, metrics)
if passed:
    deploy_agent()
else:
    block_deployment(reason=message)
```

### 8.5.6 Temporal Weighting Evolution

Weights can evolve as agents mature.

**Prototype Phase:**
```python
prototype_weights = {
    "cost": 0.05,       # Don't optimize cost yet
    "latency": 0.10,    # Don't optimize speed yet
    "accuracy": 0.60,   # Focus on getting it right
    "security": 0.20,   # Basic security
    "stability": 0.05   # Expect some instability
}
```

**Beta Phase:**
```python
beta_weights = {
    "cost": 0.15,       # Start caring about cost
    "latency": 0.20,    # Start optimizing speed
    "accuracy": 0.40,   # Still priority but balanced
    "security": 0.20,   # Maintain security
    "stability": 0.05
}
```

**Production Phase:**
```python
production_weights = {
    "cost": 0.20,       # Cost matters at scale
    "latency": 0.25,    # Speed critical for UX
    "accuracy": 0.30,   # Balanced with other factors
    "security": 0.15,   # Maintain security
    "stability": 0.10   # Reliability important
}
```

---

## Key Takeaways

1. **Custom Metrics are Necessary When Standard Metrics Miss Business-Critical Dimensions:** If >20% of business value depends on a dimension not captured by standard metrics, create a custom metric for it.

2. **Composite Metrics Provide Holistic Views:** Weighted combinations of metrics simplify reporting and make trade-offs explicit. Ensure components are normalized to the same scale (0-1) before combining.

3. **Domain Expertise is Essential:** Develop domain-specific metrics by engaging subject matter experts, operationalizing qualitative criteria, and validating against expert judgment (target correlation > 0.80).

4. **Code-Based Evaluators for Objective Criteria, LLM-Judges for Subjective:** Use code when criteria are deterministic and well-defined; use LLM-as-a-judge for nuanced, subjective dimensions like tone or appropriateness.

5. **Calibrate LLM-as-Judge Metrics:** Natural language metric definitions are powerful but require calibration against human expert ratings to ensure reliability. Run multiple evaluations and average scores to reduce variance.

6. **The CLASSic Framework Provides Industry-Standard Structure:** Cost, Latency, Accuracy, Security, Stability (CLASSic) covers the essential dimensions for production agents. Default weights (20%, 25%, 30%, 15%, 10%) work for balanced deployments.

7. **Weights Should Reflect Use Case Priorities:** Healthcare agents prioritize accuracy (40%) and security (25%); voice agents prioritize latency (40%); batch processing prioritizes cost (40%). Customize weights to align with business context.

8. **Gating Requires Both Composite Scores and Individual Minimums:** An agent might score 0.85 overall but have unacceptable security (0.70). Enforce hard requirements on critical dimensions regardless of composite score.

9. **Weights Evolve with Agent Maturity:** Prototype phase focuses on accuracy (60%); production phase balances cost (20%), latency (25%), accuracy (30%), security (15%), stability (10%).

10. **Composite Metrics Enable Simplified Reporting:** Stakeholders understand "Agent Quality Score: 0.87" better than 20 individual metrics. Use composite scores for communication while tracking individual components for debugging.

11. **Harmonic Mean Penalizes Imbalance:** Use harmonic mean instead of arithmetic mean when balanced performance across all dimensions is critical. This prevents "gaming" by excelling in one dimension while neglecting others.

12. **Validation is Non-Negotiable:** Custom metrics must be validated against expert judgment, production outcomes, or existing benchmarks before use. Unvalidated metrics risk optimizing for the wrong objectives.

---

## References

1. Galileo AI. (2025). *Four New Agent Evaluation Metrics*. https://galileo.ai/blog/four-new-agent-evaluation-metrics

2. Aisera. (2025). *What is AI Agent Evaluation: A CLASSic Approach*. https://aisera.com/blog/ai-agent-evaluation/

3. DeepEval. (2025). *Building Custom LLM Metrics | DeepEval Documentation*. https://deepeval.com/guides/guides-building-custom-metrics

4. Confident AI. (2025). *LLM Evaluation Metrics: Everything You Need for LLM Evaluation*. https://www.confident-ai.com/blog/llm-evaluation-metrics-everything-you-need-for-llm-evaluation

5. Shukla, M. (2025). *Evaluating Agentic AI Systems: A Balanced Framework for Performance, Robustness, Safety and Beyond*. SSRN. https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5402054

6. testRigor. (2025). *Different Evals for Agentic AI: Methods, Metrics & Best Practices*. https://testrigor.com/blog/different-evals-for-agentic-ai/

7. Medium. (2025). *Evaluating AI Agents: Metrics, Challenges, and Practices*. https://medium.com/@TechforHumans/evaluating-ai-agents-metrics-challenges-and-practices-c5a0444876cd

---

**Navigation:**
← [Previous: Section 7](07_advanced_and_specialized_metrics.md) | [Table of Contents](../README.md) | [Next: Section 9](09_full_trace_observability.md) →
