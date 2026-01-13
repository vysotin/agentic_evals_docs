# Section 5: Core Evaluation Metrics

> **Part IV**: Metrics & Measurements
> **Estimated Reading Time**: 25 minutes

## Overview

Core evaluation metrics form the foundation of any AI agent assessment strategy. Unlike traditional ML systems where accuracy and F1-scores suffice, agentic AI requires measuring success across multiple dimensions: whether the agent completes tasks, how it executes them, which tools it selects, the quality of its outputs, and the efficiency of its operations.

This section provides a comprehensive catalog of core metrics spanning five critical categories: task completion and success, process quality, tool and action correctness, outcome quality, and performance efficiency. Each metric includes definition, measurement approach, industry benchmarks, and implementation guidance based on 2026 production deployments.

## Table of Contents

- [5.1 Task Completion and Success Metrics](#51-task-completion-and-success-metrics)
- [5.2 Process Quality Metrics](#52-process-quality-metrics)
- [5.3 Tool and Action Metrics](#53-tool-and-action-metrics)
- [5.4 Outcome Quality Metrics](#54-outcome-quality-metrics)
- [5.5 Performance & Efficiency Metrics](#55-performance--efficiency-metrics)
- [Key Takeaways](#key-takeaways)
- [References](#references)

---

## 5.1 Task Completion and Success Metrics

Task completion metrics measure the fundamental question: did the agent accomplish what it was designed to do? These metrics form the baseline for all agent evaluations.

### 5.1.1 Task Success Rate / Task Completion Rate

**Definition:** The percentage of user requests that an agent successfully completes end-to-end without human intervention.

**Measurement Approach:**
- Define clear success criteria for each task type
- Track whether final output meets all requirements
- Binary classification: success (1) or failure (0)
- Aggregate across all attempts: `Success Rate = (Successful Tasks / Total Tasks) × 100`

**Industry Benchmarks (2026):**
- Customer support agents: 75-85% baseline, 90%+ for mature deployments
- Research/analysis agents: 70-80%
- Code generation agents: 65-75% (higher complexity)
- Voice conversational agents: 80-90% for scripted flows

**Implementation Example:**
```python
from deepeval.metrics import TaskCompletionMetric
from deepeval.test_case import LLMTestCase

# Define the metric
task_completion = TaskCompletionMetric(threshold=0.7)

# Create test case with agent trace
test_case = LLMTestCase(
    input="Book a flight from NYC to SF for next Monday",
    actual_output=agent_response,
    tools_called=agent_trace.tools,
    expected_tools=["search_flights", "book_flight", "confirm_booking"]
)

# Evaluate
task_completion.measure(test_case)
print(f"Task Completion Score: {task_completion.score}")
```

**Best Practices:**
- Set task-specific success criteria; avoid one-size-fits-all definitions
- Account for partial completion (e.g., 80% of sub-tasks completed)
- Track success rate trends over time to detect degradation
- Segment by task complexity to identify capability gaps

### 5.1.2 Task Resolution

**Definition:** Whether the agent fully resolved the user's underlying need, not just completed mechanical steps.

**Measurement Approach:**
- Use LLM-as-a-judge to assess if the user's intent was truly satisfied
- Consider follow-up questions or escalations as indicators of incomplete resolution
- Evaluate based on user query, agent response, and any subsequent interactions

**Industry Benchmarks:**
- High-performing customer service agents: 85-90% resolution rate
- Research assistants: 75-85%
- Administrative task agents: 80-90%

**Distinction from Task Success:** An agent might successfully complete defined steps (high task success rate) but fail to resolve the user's actual problem (low resolution rate). For example, an agent might retrieve a document as requested, but if the document doesn't contain the needed information, the task is completed but not resolved.

### 5.1.3 Error Rate

**Definition:** The percentage of agent runs that result in errors, failures, or exceptions.

**Measurement Approach:**
- Track system errors (crashes, timeouts, API failures)
- Track logical errors (incorrect tool selection, invalid parameters)
- Track policy violations (attempts to access unauthorized resources)
- Calculate: `Error Rate = (Failed Executions / Total Executions) × 100`

**Industry Benchmarks:**
- Production agents: < 5% total error rate
- Critical systems (healthcare, finance): < 1% error rate
- Tool call errors: < 2%
- API timeout errors: < 1%

**Error Categorization:**
- **Fatal errors:** Agent cannot recover, task fails completely
- **Recoverable errors:** Agent can retry or use alternative approach
- **Silent errors:** Agent continues but produces incorrect output (most dangerous)

### 5.1.4 Containment Rate

**Definition:** The percentage of customer interactions resolved end-to-end by the agent without human agent escalation. The #1 KPI for measuring AI agent ROI in customer service [1].

**Measurement Approach:**
- Track interactions from start to completion
- Flag any hand-off to human agents
- Calculate: `Containment Rate = (Fully Automated Resolutions / Total Interactions) × 100`

**Industry Benchmarks (2026):**
- **70%+ containment:** Strong ROI, millions in cost savings [1]
- **80-90% containment:** Best-in-class performance (e.g., Alhena AI e-commerce clients) [1]
- **Below 40%:** Negative ROI, poor customer experience [1]
- Voice agents in contact centers: 65-80% typical

**Business Impact:**
- Conversational AI expected to reduce customer service costs by $80 billion by 2026 [2]
- Each 10% increase in containment rate can save hundreds of thousands annually

**Important Note:** High containment doesn't guarantee quality. Customers might abandon interactions rather than escalate, artificially inflating containment rate. Always pair with Customer Satisfaction (CSAT) scores to ensure customers are actually satisfied with automated resolutions [2].

### 5.1.5 Completion Rate

**Definition:** The percentage of started interactions that reach a defined endpoint (successful or not).

**Measurement Approach:**
- Track sessions from initiation to termination
- Identify abandoned sessions (user quits mid-interaction)
- Calculate: `Completion Rate = (Completed Sessions / Started Sessions) × 100`

**Industry Benchmarks:**
- E-commerce agents: 75-85% completion rate
- Information retrieval agents: 80-90%
- Multi-step workflow agents: 65-80%

**Why It Matters:** Low completion rates indicate friction points, confusing interactions, or agents that fail to maintain user engagement. Analyzing where users abandon provides crucial UX insights.

### 5.1.6 First Contact Resolution (FCR)

**Definition:** The percentage of customer issues resolved in a single interaction without requiring follow-ups or escalations. One of the most critical KPIs alongside Average Handle Time (AHT) and CSAT [3].

**Measurement Approach:**
- Track if customer returns with same issue within defined timeframe (typically 7 days)
- Survey customers post-interaction: "Was your issue fully resolved?"
- Analyze for absence of follow-up tickets on same topic

**Industry Benchmarks:**
- Traditional call centers: 70-75% FCR baseline
- AI-augmented agents: 75-85%
- Best-in-class AI agents: 85-90%
- Voice AI agents: 60-75% (lower due to complexity)

**Business Impact:**
- Each 1% improvement in FCR can reduce operational costs by 1-2%
- High FCR strongly correlates with improved CSAT and Net Promoter Score (NPS)
- Reduces agent handling time and increases productivity

**Measurement Challenges:** Difficult to track across channels; requires robust session management and customer identification across touchpoints.

---

## 5.2 Process Quality Metrics

Process quality metrics evaluate *how* the agent accomplishes tasks, not just whether it succeeds. These metrics expose issues in reasoning, planning, and execution strategy.

### 5.2.1 Plan Quality (PlanQualityMetric)

**Definition:** Assesses whether the plan or strategy the agent generates is logical, complete, and efficient for accomplishing the given task [4].

**Measurement Approach (DeepEval Implementation):**
1. Extract the task from the agent's trace (user goal)
2. Extract the plan from the agent's reasoning steps
3. Use an LLM judge to evaluate the plan against criteria:
   - **Logical:** Does the plan follow sound reasoning?
   - **Complete:** Does it address all aspects of the task?
   - **Efficient:** Is it the most direct path, or are there unnecessary steps?
4. Score on scale of 0-1 or 1-5

**Evaluation Criteria:**
```python
from deepeval.metrics import PlanQualityMetric
from deepeval.test_case import LLMTestCase

plan_quality = PlanQualityMetric(
    threshold=0.7,
    model="gpt-4o",  # Judge model
    include_reason=True
)

test_case = LLMTestCase(
    input="Analyze Q3 sales data and identify top 3 growth opportunities",
    actual_output=agent.response,
    retrieval_context=agent.trace.reasoning_steps  # Plan is extracted from here
)

plan_quality.measure(test_case)
print(f"Plan Quality Score: {plan_quality.score}")
print(f"Reasoning: {plan_quality.reason}")
```

**Industry Benchmarks:**
- High-performing agents: Plan Quality > 0.8
- Acceptable baseline: > 0.7
- Scores < 0.6 indicate poor planning capabilities

**Best Practices:**
- If no explicit plan exists in trace, metric passes by default (score = 1.0) [4]
- Calibrate the LLM judge against human expert assessments
- Track plan quality trends to detect model drift or prompt degradation

### 5.2.2 Plan Coherence

**Definition:** Measures whether the agent follows a logical, consistent plan without getting sidetracked or contradicting itself.

**Measurement Approach:**
- Analyze chain-of-thought for topic drift
- Check for logical contradictions between steps
- Use LLM judge to score coherence of reasoning flow [5]
- Detect "dead-end" steps that don't contribute to final goal

**Scoring Rubric (1-5 scale):**
- **5:** Perfectly coherent, all steps contribute logically
- **4:** Coherent with minor tangents that self-correct
- **3:** Some off-topic exploration but returns to goal
- **2:** Significant drift, multiple contradictions
- **1:** Incoherent, random or contradictory steps

**Industry Benchmarks:**
- Production-ready agents: Average coherence > 4.0
- Research/exploratory agents: > 3.5 (some tangents acceptable)

### 5.2.3 Plan Adherence (PlanAdherenceMetric)

**Definition:** Evaluates whether the agent actually follows the plan it generated, or deviates during execution [4].

**Measurement Approach (DeepEval):**
1. Extract the plan from reasoning trace
2. Extract the execution steps (tools called, actions taken)
3. Use LLM judge to compare: did execution match the plan?
4. Score based on alignment

**Why It Matters:** An agent might generate a perfect plan (high Plan Quality) but then ignore it during execution (low Plan Adherence), leading to suboptimal outcomes.

**Common Failure Modes:**
- Agent gets distracted by intermediate results
- Tool failures cause improvisation without updating plan
- Multi-agent systems where one agent ignores coordinator's plan

**Implementation:**
```python
from deepeval.metrics import PlanAdherenceMetric

plan_adherence = PlanAdherenceMetric(threshold=0.75)

test_case = LLMTestCase(
    input="Create quarterly report with sections on revenue, expenses, and profit",
    actual_output=agent.final_output,
    retrieval_context=[agent.trace.plan, agent.trace.execution_steps]
)

plan_adherence.measure(test_case)
```

**Industry Benchmarks:**
- Disciplined agents: > 0.85 adherence
- Acceptable range: 0.70-0.85
- < 0.70 indicates poor execution discipline

### 5.2.4 Step-wise Contribution

**Definition:** Measures whether each step in the agent's reasoning chain was necessary and valuable, or if there were redundant or wasted steps [5].

**Measurement Approach:**
- Identify each step in the trace
- Evaluate if step's output was used in subsequent reasoning
- Flag steps whose outputs are never referenced (dead-ends)
- Calculate: `Contribution Rate = (Contributory Steps / Total Steps) × 100`

**Industry Benchmarks:**
- Efficient agents: > 85% of steps contribute
- Acceptable: > 75%
- < 70% indicates excessive exploration or poor planning

**Cost Impact:** Redundant steps waste tokens and increase latency. In agents making many LLM calls, step efficiency directly impacts cost.

### 5.2.5 Step-Level Accuracy

**Definition:** The correctness of each individual step in the agent's execution, independent of overall task success.

**Measurement Approach:**
- Evaluate each step against ground truth or expert judgment
- Identify which specific step caused failure in unsuccessful runs
- Calculate: `Step Accuracy = (Correct Steps / Total Steps) × 100`

**Use Case:** Debugging. When an agent fails, step-level accuracy pinpoints *where* it went wrong (retrieval? reasoning? tool call? output formatting?).

**Industry Practice:**
- Critical for forensic analysis after failures
- Enables targeted improvements (e.g., "retrieval step is only 60% accurate")

### 5.2.6 Instruction Adherence

**Definition:** Measures whether the agent follows all system prompts, developer instructions, and behavioral constraints [5].

**Measurement Approach:**
- Define explicit rules (e.g., "never reveal internal reasoning," "always cite sources," "stay within token budget")
- Programmatically check outputs and traces for violations
- Use LLM judge for nuanced rules

**Example Constraints:**
- Output format requirements
- Tone and style guidelines
- Security policies (never expose API keys)
- Ethical boundaries (refuse harmful requests)

**Industry Benchmarks:**
- Production agents: > 95% instruction adherence
- Security-critical systems: > 99%

**Measurement Example:**
```python
# Programmatic check
def check_instruction_adherence(agent_output, rules):
    violations = []
    if rules.get("cite_sources") and "[Source:" not in agent_output:
        violations.append("Missing source citations")
    if rules.get("max_tokens") and len(agent_output.split()) > rules["max_tokens"]:
        violations.append("Exceeded token limit")

    adherence_score = 1.0 - (len(violations) / len(rules))
    return adherence_score, violations
```

---

## 5.3 Tool and Action Metrics

Agentic AI's defining characteristic is the ability to use tools and take actions. These metrics assess the quality of tool usage.

### 5.3.1 Tool Use Accuracy

**Definition:** Did the agent choose the right tools and use them correctly throughout the entire task execution? [5]

**Measurement Approach:**
- Analyze all tool call spans in the trace
- Verify correct tool selection for each sub-task
- Check for properly formatted inputs
- Confirm successful execution
- Aggregate: `Tool Use Accuracy = (Correct Tool Uses / Total Tool Calls) × 100`

**Industry Benchmarks (2026):**
- Production-ready agents: > 95% tool use accuracy
- Acceptable baseline: > 90%
- Research/experimental agents: > 85%

**Common Failure Modes:**
- Wrong tool selected (e.g., using search when calculation needed)
- Correct tool with malformed parameters
- Ignoring tool output or misinterpreting results
- Calling deprecated tools

### 5.3.2 Tool Correctness (ToolCorrectnessMetric)

**Definition:** Evaluates whether the agent selects the right tools and calls them correctly, comparing actual tool calls against expected tools [4].

**DeepEval Implementation:**
```python
from deepeval.metrics import ToolCorrectnessMetric
from deepeval.test_case import LLMTestCase, LLMTestCaseParams

tool_correctness = ToolCorrectnessMetric(
    threshold=0.7,
    strictness="name_only"  # Options: "name_only", "name_and_inputs", "name_inputs_and_output"
)

test_case = LLMTestCase(
    input="What's the weather in San Francisco?",
    actual_output=agent.response,
    tools_called=[
        {"name": "get_current_weather", "input": {"location": "San Francisco", "unit": "celsius"}}
    ],
    expected_tools=[
        {"name": "get_current_weather", "input": {"location": "San Francisco"}}
    ]
)

tool_correctness.measure(test_case)
```

**Strictness Levels [4]:**
- **name_only:** Only tool names must match (default)
- **name_and_inputs:** Tool names and input parameters must match
- **name_inputs_and_output:** Tool names, inputs, AND outputs must match (strictest)

**When to Use Each Level:**
- Name-only: Early development, broad capability testing
- Name + inputs: Production readiness testing
- Full match: Regression testing, ensuring exact reproducibility

### 5.3.3 Tool Selection Accuracy

**Definition:** The percentage of times the agent selects the optimal tool for a given sub-task on the first attempt.

**Measurement Approach:**
- For each decision point, identify available tools
- Determine optimal tool choice (based on task requirements)
- Check if agent selected that tool
- Calculate: `Selection Accuracy = (Optimal Selections / Total Tool Decisions) × 100`

**Industry Benchmarks:**
- Mature agents: > 90% first-choice accuracy
- Acceptable: > 85%
- Multi-tool environments with 10+ tools: > 80% acceptable

**Impact:** Poor tool selection wastes tokens and time on trial-and-error. Agents that frequently select suboptimal tools before finding the right one have higher latency and cost.

### 5.3.4 Tool Call Frequency

**Definition:** The number of tool calls per task or per unit time.

**Measurement Approach:**
- Track tool calls in each agent trace
- Aggregate by task type, user intent, or time period
- Calculate average, median, P95, P99

**Industry Benchmarks:**
- Simple queries: 1-3 tool calls
- Complex workflows: 5-15 tool calls
- Multi-step research tasks: 10-30 tool calls

**Why It Matters:**
- **Too few calls:** Agent might not be leveraging available tools, relying on parametric knowledge (risk of hallucination)
- **Too many calls:** Inefficiency, excessive cost, high latency
- Sudden spikes indicate potential issues (loops, retries)

**Cost Consideration:** Complex agents with tool-calling consume 5-20x more tokens than simple chains due to loops and retries [6]. Monitoring tool call frequency helps control costs.

### 5.3.5 Tool Usage Patterns

**Definition:** Qualitative analysis of how agents use tools over time, including sequences, dependencies, and retry behavior.

**What to Track:**
- Tool call sequences (which tools are called together? in what order?)
- Retry patterns (how often does agent retry failed tool calls?)
- Tool dependencies (does calling tool A always lead to calling tool B?)
- Unused tools (which available tools are never called?)

**Analysis Techniques:**
- Sequence mining to identify common patterns
- Anomaly detection for unusual call sequences
- Dependency graphs showing tool relationships

**Use Cases:**
- Optimizing tool sets (remove unused tools, add missing ones)
- Detecting inefficiencies (unnecessary retry loops)
- Understanding agent strategies

---

## 5.4 Outcome Quality Metrics

Outcome quality metrics evaluate the final output delivered to the user, assessing correctness, relevance, and completeness.

### 5.4.1 Factual Correctness

**Definition:** Is the information provided by the agent accurate and free of hallucinations? [5]

**Measurement Approach:**
- Cross-reference facts in output against trusted knowledge base
- Compare output to retrieved context (for RAG systems)
- Use LLM judge with fact-checking prompts
- Manual spot-checking for high-stakes applications

**Industry Benchmarks:**
- Production agents: > 99% factual correctness for critical domains (healthcare, finance)
- General-purpose assistants: > 95%
- Research/analysis agents: > 90% (acknowledge uncertainty for ambiguous cases)

**Implementation Example:**
```python
from deepeval.metrics import FactualConsistencyMetric

factual_consistency = FactualConsistencyMetric(threshold=0.95)

test_case = LLMTestCase(
    input="What's the capital of France?",
    actual_output="The capital of France is Paris.",
    retrieval_context=["Paris is the capital and most populous city of France."]
)

factual_consistency.measure(test_case)
```

**Challenge:** Hallucination detection is non-trivial. Even state-of-the-art LLMs produce plausible-sounding but incorrect information, and automated detection methods have imperfect recall.

**Best Practices:**
- For critical applications, use hybrid evaluation: automated + human review
- Require agents to cite sources for factual claims
- Implement confidence thresholds (refuse to answer if uncertain)

### 5.4.2 Intent Fulfillment / Intent Resolution

**Definition:** Did the agent understand and address the user's true, underlying intent? [7]

**Measurement Approach (Azure AI Foundry):**
- Use IntentResolutionEvaluator to assess how well the system identifies user intent
- Evaluate how well agent scopes the request
- Check if agent asks clarifying questions when intent is ambiguous
- Measure if agent reminds users of its capabilities
- Output: Likert score (1-5, higher is better) [7]

**Azure AI Implementation:**
```python
from azure.ai.evaluation import IntentResolutionEvaluator

intent_evaluator = IntentResolutionEvaluator(model_config={"api_key": "...", "model": "gpt-4o"})

result = intent_evaluator(
    query="I need to travel next month",
    response=agent.response,
    context=conversation_history
)

print(f"Intent Resolution Score: {result['intent_resolution_score']}")  # 1-5
```

**Industry Benchmarks:**
- High-performing agents: Average score > 4.0
- Acceptable: > 3.5
- Scores < 3.0 indicate poor intent understanding

**Why It Matters:** Users often express needs indirectly or incompletely. An agent that mechanically answers the literal question but misses the underlying intent provides poor UX. Example:
- User: "What's the weather?"
- Literal response: "Weather is atmospheric conditions." (Correct but useless)
- Intent-aware response: "The weather in [your location] is currently sunny, 72°F." (Infers user wants local current weather)

### 5.4.3 Answer Completeness

**Definition:** Did the agent provide all necessary information to fully address the query, without being overly verbose? [5]

**Measurement Approach:**
- Decompose query into sub-questions
- Check if answer addresses each sub-question
- Use LLM judge to assess completeness against rubric
- Penalize excessive verbosity (completeness ≠ length)

**Evaluation Criteria:**
- All required information present
- No critical gaps
- Appropriate level of detail (not too shallow, not overwhelming)
- Well-structured and easy to parse

**Industry Benchmarks:**
- Customer service agents: > 90% completeness
- Research assistants: > 85% (balancing completeness with conciseness)

**Common Failure Modes:**
- Partial answers (addresses only first part of multi-part question)
- Over-explanation (provides far more detail than requested)
- Missing caveats or disclaimers (technically complete but misleading)

### 5.4.4 Contextual Relevancy / Retrieval Relevance

**Definition:** How relevant is the information retrieved or generated to the specific user query?

**Measurement Approach:**
- For RAG systems: Evaluate relevance of retrieved chunks
- For general responses: Assess if content directly addresses query
- Use semantic similarity between query and response
- LLM judge with relevancy criteria

**Industry Benchmarks:**
- RAG systems: > 0.85 relevance score for retrieved chunks
- General responses: > 0.90 relevance

**Metric Example (RAG-specific):**
```python
from ragas.metrics import context_relevancy

# RAGAS implementation
score = context_relevancy.score(
    question="What are the benefits of exercise?",
    contexts=[
        "Regular exercise improves cardiovascular health and reduces disease risk.",
        "The weather today is sunny and warm."  # Irrelevant
    ]
)
# Returns score based on proportion of relevant contexts
```

### 5.4.5 Groundedness

**Definition:** Is the agent's response grounded in and supported by the provided context/retrieved documents, or does it introduce unsupported claims?

**Measurement Approach:**
- Extract claims from agent's response
- Check if each claim has support in retrieved context
- Flag any claims that aren't grounded in provided information
- Calculate: `Groundedness = (Grounded Claims / Total Claims) × 100`

**Industry Benchmarks:**
- RAG systems: > 95% groundedness (critical to avoid hallucinations)
- Agents with access to databases: > 90%

**Azure AI Implementation:**
```python
from azure.ai.evaluation import GroundednessEvaluator

groundedness = GroundednessEvaluator(model_config=model_config)

result = groundedness(
    query="What's our return policy?",
    response=agent.response,
    context=retrieved_documents
)
print(f"Groundedness Score: {result['groundedness_score']}")
```

**Why It Matters:** Groundedness is the primary defense against hallucination in RAG systems. Agents should only make claims supported by retrieved evidence.

### 5.4.6 Response Quality

**Definition:** Overall assessment of response quality across multiple subjective dimensions: clarity, professionalism, appropriateness, helpfulness.

**Measurement Approach:**
- Multi-dimensional rubric evaluated by LLM judge or human raters
- Dimensions typically include:
  - **Clarity:** Easy to understand, well-structured
  - **Accuracy:** Factually correct
  - **Helpfulness:** Actually addresses user need
  - **Tone:** Appropriate for context (professional, friendly, etc.)
  - **Conciseness:** Sufficient detail without verbosity

**Industry Benchmarks:**
- Customer-facing agents: Average quality score > 4.0 / 5.0
- Internal tooling agents: > 3.5 / 5.0

**Implementation:** Often implemented as weighted composite of other metrics:
```
Response Quality = 0.3×Factual_Correctness + 0.25×Completeness + 0.25×Relevance + 0.1×Tone + 0.1×Conciseness
```

---

## 5.5 Performance & Efficiency Metrics

Performance metrics quantify the speed, cost, and resource efficiency of agent operations. In 2026, these are first-class concerns, treated with the same importance as cloud infrastructure cost optimization.

### 5.5.1 End-to-End Latency

**Definition:** Total time from when user submits a query to when they receive the final response.

**Measurement Approach:**
- Timestamp at request initiation
- Timestamp at final output delivery
- Calculate delta: `End-to-End Latency = t_final - t_initial`
- Track distribution (mean, median, P50, P95, P99)

**Industry Benchmarks (2026) [8, 9]:**
- **Simple queries:** P50 < 500ms, P95 < 1000ms
- **Complex workflows:** P50 < 2 seconds, P95 < 4 seconds
- **Multi-agent orchestration:** P50 < 3 seconds, P95 < 6 seconds
- **Voice AI agents:** Sub-1000ms considered acceptable, 2000ms upper limit, leading platforms achieve 500-800ms [8]

**Production Thresholds:**
- Alert when P95 > 800ms for conversational agents [8]
- Critical alert for voice agents if P95 > 1000ms

**Real-World Performance Example:** One deployment reduced P95 latency from 1.4 seconds to 380ms by identifying slow database lookups and external APIs under load [8].

### 5.5.2 Time-to-First-Token (TTFT)

**Definition:** Time from query submission to when the first token of the response is generated.

**Measurement Approach:**
- Track LLM streaming start time
- Calculate: `TTFT = t_first_token - t_request`

**Industry Benchmarks:**
- Interactive applications: < 200ms ideal, < 500ms acceptable
- Batch processing: < 1 second acceptable

**Why It Matters:** TTFT significantly impacts perceived responsiveness. Users experience sub-200ms TTFT as "instant," while anything over 500ms feels sluggish.

### 5.5.3 Response Time / Average Latency

**Definition:** Mean latency across all requests in a given time period.

**Measurement Approach:**
- Aggregate all request latencies
- Calculate mean and standard deviation
- Track trends over time

**Industry Benchmarks:**
- Production agents: Mean < 1.5 seconds for typical queries
- Voice agents: Mean < 800ms [8]

**Monitoring Best Practice:** Track mean alongside percentiles. Mean latency can look healthy while P99 latency is terrible, causing poor experience for outliers.

### 5.5.4 Latency Percentiles (P50, P95, P99)

**Definition:** Latency values at specific percentiles of the distribution [10].

**What Each Percentile Tells You:**
- **P50 (Median):** Experience for typical user
- **P95:** 95% of requests are faster than this; captures slow tail
- **P99:** 99% of requests are faster; exposes severe outliers

**Why Percentiles Matter:**
- Mean is misleading when distribution is skewed
- P99 latency often determines user experience for worst-case scenarios
- Production SLAs typically defined in percentiles (e.g., "P95 latency < 1 second")

**Industry Benchmarks [8, 9]:**
| Use Case | P50 Target | P95 Target | P99 Target |
|----------|-----------|-----------|-----------|
| Simple queries | < 500ms | < 1000ms | < 1500ms |
| Voice agents | < 600ms | < 1000ms | < 1500ms |
| Complex workflows | < 2s | < 4s | < 6s |
| Multi-agent systems | < 3s | < 6s | < 10s |

**Real-World Example:** Major US cities see 250-500ms typical latency for conversational agents [8].

**Implementation:**
```python
import numpy as np

latencies = [fetch_latency_from_logs(trace) for trace in agent_traces]

p50 = np.percentile(latencies, 50)
p95 = np.percentile(latencies, 95)
p99 = np.percentile(latencies, 99)

print(f"P50: {p50}ms, P95: {p95}ms, P99: {p99}ms")

# Alert if P95 exceeds threshold
if p95 > 800:
    send_alert("P95 latency exceeded threshold")
```

### 5.5.5 Execution Time per Agent

**Definition:** Time each agent or sub-agent spends processing in multi-agent systems.

**Measurement Approach:**
- Track span duration for each agent in the orchestration
- Identify bottleneck agents (those taking disproportionate time)
- Calculate: Total execution time, time per agent, time distribution

**Use Case:** In a multi-agent system with Planner, Researcher, and Writer agents, tracking execution time per agent reveals which agent is the bottleneck.

**Optimization Strategy:** If Writer agent takes 80% of total time, optimize that agent specifically (better prompts, faster model, caching).

### 5.5.6 Token Usage and Cost

**Definition:** Total tokens consumed (input + output) and associated cost.

**Measurement Approach [6]:**
- Track input and output tokens for each LLM call
- Aggregate across all calls in a trace
- Calculate cost based on model pricing: `Cost = (Input_Tokens × Input_Price + Output_Tokens × Output_Price)`
- Monitor token usage distribution across agents

**Industry Benchmarks:**
- Simple queries: 500-2000 tokens
- Complex multi-step tasks: 5,000-20,000 tokens
- Research/analysis tasks: 20,000-100,000+ tokens

**Cost Considerations [6]:**
- Complex agents with tool-calling consume **5-20x more tokens** than simple chains due to loops and retries
- Track token usage at the trace level to identify expensive operations
- Monitor token cost per task completion (see 5.5.7)

**Implementation:**
```python
def calculate_trace_cost(trace, pricing):
    total_cost = 0
    for llm_call in trace.llm_calls:
        input_cost = llm_call.input_tokens * pricing[llm_call.model]['input']
        output_cost = llm_call.output_tokens * pricing[llm_call.model]['output']
        total_cost += (input_cost + output_cost)
    return total_cost

# Pricing per 1M tokens (example)
pricing = {
    "gpt-4o": {"input": 2.50, "output": 10.00},
    "gpt-4o-mini": {"input": 0.15, "output": 0.60}
}

cost = calculate_trace_cost(agent_trace, pricing)
print(f"Trace cost: ${cost:.4f}")
```

### 5.5.7 Token Cost per Task

**Definition:** Average cost in tokens to complete a successful task.

**Measurement Approach:**
- Track tokens used only for successful task completions
- Calculate: `Token Cost per Task = Total Tokens (Successful) / Number of Successful Tasks`
- Monitor trends to detect cost drift

**Industry Benchmarks:**
- Customer service queries: 1,000-5,000 tokens per task
- Code generation tasks: 3,000-15,000 tokens per task
- Research/analysis: 10,000-50,000 tokens per task

**Why It Matters:** Tracks efficiency of task completion. Rising cost per task indicates agent is taking more steps, retrying more, or using less efficient strategies.

### 5.5.8 Tokens per Second

**Definition:** Throughput metric measuring token generation rate.

**Measurement Approach:**
- For streaming: Track tokens generated over time window
- Calculate: `Tokens/sec = Output_Tokens / Generation_Duration`

**Industry Benchmarks:**
- Modern LLMs: 20-100 tokens/second depending on model size and infrastructure
- Faster models (e.g., GPT-4o-mini): 50-100 tokens/sec
- Larger models (e.g., GPT-4o): 20-40 tokens/sec

**Use Case:** Helps predict latency for long-form generation. If agent needs to generate 1000-token summary, at 50 tokens/sec that's 20 seconds.

### 5.5.9 Context Utilization

**Definition:** How much of the available context window is being used.

**Measurement Approach:**
- Track total tokens in context (system prompt + conversation history + retrieved docs + current query)
- Calculate: `Context Utilization = (Tokens in Context / Max Context Window) × 100`

**Industry Benchmarks:**
- Healthy range: 40-80% utilization
- **Below 40%:** Possibly under-utilizing available context
- **Above 80%:** Risk of running out of context, may need to truncate

**Why It Matters:**
- Approaching context limits forces truncation, potentially losing important information
- High utilization increases latency and cost
- Monitoring helps decide when to implement context compression or summarization

### 5.5.10 Tool Call Count

**Definition:** Number of tool/function calls made during task execution.

**Measurement Approach:**
- Count tool call spans in trace
- Aggregate by task type
- Track distribution (mean, median, max)

**Industry Benchmarks:**
- Simple tasks: 1-3 tool calls
- Moderate complexity: 4-8 tool calls
- Complex multi-step: 10-20 tool calls
- **Alert threshold:** > 30 tool calls (possible infinite loop)

**Cost Impact:** Each tool call adds latency (network round-trip) and often additional LLM calls to process results. Excessive tool calls are expensive.

### 5.5.11 Agent Efficiency Score

**Definition:** Composite metric measuring how efficiently the agent completes tasks relative to optimal path.

**Measurement Approach:**
- Define optimal solution (minimum steps, minimum tokens, minimum tool calls)
- Compare actual execution to optimal
- Calculate: `Efficiency = 1 - (Actual_Resources - Optimal_Resources) / Optimal_Resources`

**Example:**
- Optimal solution: 5 steps, 2000 tokens
- Actual execution: 8 steps, 3500 tokens
- Step efficiency: 1 - (8-5)/5 = 0.40 (60% overhead in steps)
- Token efficiency: 1 - (3500-2000)/2000 = 0.25 (75% overhead in tokens)

**Industry Benchmarks:**
- High-efficiency agents: > 0.80 (within 20% of optimal)
- Acceptable: > 0.65
- < 0.50 indicates significant inefficiency

### 5.5.12 Step Utility

**Definition:** Ratio of useful steps to total steps; inverse of wasted effort [11].

**Measurement Approach:**
- Identify which steps contributed to final output
- Flag steps that were dead-ends or redundant
- Calculate: `Step Utility = (Contributory Steps / Total Steps) × 100`

**Industry Benchmarks:**
- Efficient agents: > 85% step utility
- Acceptable: > 75%
- < 70% indicates excessive exploration or poor planning

**Related to:** Step-wise Contribution (5.2.4); this is the quantitative version.

### 5.5.13 Cost per Interaction

**Definition:** Total cost (tokens, API calls, infrastructure) per user interaction.

**Measurement Approach:**
- Aggregate all costs for a single interaction:
  - LLM API costs (input + output tokens)
  - Tool/function call costs (external API calls)
  - Infrastructure costs (compute, memory, storage)
- Calculate per-interaction cost

**Industry Benchmarks:**
- Simple chatbot interactions: $0.001 - $0.01
- Complex research agents: $0.05 - $0.50
- Advanced multi-agent systems: $0.10 - $2.00+

**Business Impact:** With millions of interactions, small per-interaction costs compound. Reducing cost per interaction by $0.01 can save tens of thousands annually at scale.

### 5.5.14 Cost per Successful Completion

**Definition:** Cost incurred only for successfully completed tasks (excludes failed attempts).

**Measurement Approach:**
- Track costs only for interactions that resulted in successful task completion
- Calculate: `Cost per Success = Total Costs (Successful Tasks) / Number of Successful Completions`

**Why It Matters:** More meaningful business metric than raw cost per interaction. Measures efficiency of successful outcomes, not total spend including failures.

**Industry Benchmarks:**
- Customer support: $0.05 - $0.25 per successful resolution (vs. $5-$15 for human agent)
- Code generation: $0.10 - $1.00 per successful code solution
- ROI threshold: Must be significantly cheaper than alternative (human labor, manual process)

### 5.5.15 API Call Expenses

**Definition:** Costs specifically from external API calls (not including LLM costs).

**Measurement Approach:**
- Track tool calls to external APIs (databases, search engines, third-party services)
- Sum per-call costs based on API pricing
- Monitor trends and spikes

**Common External API Costs:**
- Search API calls: $0.001 - $0.01 per query
- Database queries: Variable (often free internally, but compute cost)
- Third-party data enrichment: $0.01 - $0.50 per call

**Optimization Strategies:**
- Cache frequently requested data
- Batch API calls when possible
- Use cheaper alternatives for non-critical calls

### 5.5.16 Infrastructure Overhead

**Definition:** Compute, memory, storage, and networking costs for running the agent infrastructure (excluding LLM API costs).

**Measurement Approach:**
- Track compute resources (CPU, GPU hours)
- Monitor memory usage
- Account for storage costs (vector databases, logs, traces)
- Include orchestration overhead (Kubernetes, serverless functions)

**Industry Benchmarks:**
- Serverless agents: Minimal infrastructure overhead ($0.0001 - $0.001 per invocation)
- Self-hosted agents: Higher fixed costs, lower marginal costs
- Enterprise deployments: Infrastructure can be 10-30% of total costs

**Cost Optimization:**
- Use serverless for low-volume, bursty workloads
- Use dedicated instances for high-volume, steady workloads
- Implement auto-scaling to match demand

---

## Key Takeaways

1. **Multi-Dimensional Evaluation is Essential:** Core metrics span five critical dimensions—task success, process quality, tool usage, outcome quality, and performance efficiency. Optimizing a single dimension (e.g., task completion) while ignoring others (e.g., cost, latency) leads to production failures.

2. **Industry Benchmarks Provide Targets:** 2026 production deployments have established clear benchmarks across metrics. Customer service agents achieving 80%+ containment, 85%+ FCR, and sub-800ms P95 latency represent best-in-class performance. Use these as calibration points.

3. **Process Matters as Much as Outcome:** Task completion rate alone doesn't reveal *how* an agent succeeded. Process quality metrics (plan quality, plan adherence, step utility) expose inefficiencies, wasted effort, and fragile strategies that will break under stress.

4. **Tool Correctness is Non-Negotiable:** Agentic AI's power comes from tool use. Tool correctness, tool selection accuracy, and tool use accuracy must exceed 90% for production readiness. Even small percentages of tool errors compound in multi-step workflows.

5. **Latency Percentiles (P95, P99) Matter More Than Mean:** Mean latency hides poor tail performance. Voice agents must hit sub-1000ms P95 latency. Complex workflows should stay under 4-second P95. Monitor percentiles, not just averages.

6. **Cost is a First-Class Metric:** In 2026, cost optimization is as critical as accuracy. Agents consuming 5-20x more tokens than necessary due to inefficient planning are uneconomical at scale. Track cost per successful completion, not just raw spend.

7. **Containment Rate Requires CSAT Context:** High containment rates (80%+) are valuable only if customers are satisfied. Pair containment metrics with CSAT scores to ensure customers aren't abandoning interactions in frustration rather than escalating.

8. **Groundedness Prevents Hallucination:** For RAG systems and knowledge-grounded agents, groundedness (>95% of claims supported by context) is the primary defense against hallucination. Measure it rigorously.

9. **Intent Resolution Beats Literal Accuracy:** Agents must understand underlying user intent, not just answer literal questions. Intent resolution scores (4.0+/5.0) separate good agents from great ones.

10. **Efficiency Metrics Drive Scalability:** Step utility, tool call count, and token efficiency determine whether an agent can scale economically. Agents that complete tasks correctly but inefficiently won't survive production economics.

---

## References

1. Alhena AI. (2025). *What is AI Containment Rate & Deflection Rate? (E-commerce Chatbot Benchmarks)*. https://alhena.ai/blog/ai-chatbot-containment-vs-deflection-rate/

2. Replicant. (2025). *How to measure voice AI success: Metrics that actually matter*. https://www.replicant.com/blog/how-to-measure-voice-ai-success-metrics-that-actually-matter

3. Hakuna Matata Tech. (2025). *KPIs for AI Voice Agents in Contact Centers | Key Metrics*. https://www.hakunamatatatech.com/our-resources/blog/ai-voice-agents-in-contact-centers

4. DeepEval. (2025). *AI Agent Evaluation Metrics | DeepEval by Confident AI*. https://deepeval.com/guides/guides-ai-agent-evaluation-metrics

5. Galileo AI. (2025). *AI Agent Evaluation Guide (Section 2.2)*. Internal documentation.

6. DEV Community. (2025). *How to Ensure Your AI Agents Do Not Consume Too Many Tokens*. https://dev.to/kuldeep_paul/how-to-ensure-your-ai-agents-do-not-consume-too-many-tokens-120p

7. Microsoft. (2025). *Agent Evaluators for Generative AI - Azure AI Foundry*. https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/evaluation-evaluators/agent-evaluators

8. Hamming AI. (2025). *How to Evaluate AI Voice Agents Performance in Production*. https://hamming.ai/blog/how-to-evaluate-ai-voice-agents-performance-in-production

9. Aviso. (2025). *How to Evaluate AI Agents: Latency, Cost, Safety, ROI*. https://www.aviso.com/blog/how-to-evaluate-ai-agents-latency-cost-safety-roi

10. OneUptime. (2025). *P50 vs P95 vs P99 Latency: What These Percentiles Actually Mean*. https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view

11. Maxim AI. (2025). *Evaluating Agentic AI Systems: Frameworks, Metrics, and Best Practices*. https://www.getmaxim.ai/articles/evaluating-agentic-ai-systems-frameworks-metrics-and-best-practices/

---

**Navigation:**
← [Previous: Section 4](04_evaluation_frameworks_and_methodologies.md) | [Table of Contents](../README.md) | [Next: Section 6](06_safety_and_security_metrics.md) →
