# Section 16: Cloud Provider Evaluation Platforms

> **Part VII**: Tools & Platforms
> **Estimated Reading Time**: 15 minutes

## Overview

Major cloud providers have recognized the critical importance of AI agent evaluation and have developed native evaluation capabilities within their platforms. These cloud-native solutions offer deep integration with provider-specific services, managed infrastructure, and enterprise-grade security and compliance. This section examines the evaluation platforms from AWS, Google Cloud, and Microsoft Azure, highlighting their unique strengths and trajectory-based evaluation capabilities.

## Table of Contents

- [16.1 Google Vertex AI](#161-google-vertex-ai)
- [16.2 Amazon Bedrock](#162-amazon-bedrock)
- [16.3 Microsoft Azure AI Foundry](#163-microsoft-azure-ai-foundry)
- [16.4 Provider Comparison Matrix](#164-provider-comparison-matrix)

---

## 16.1 Google Vertex AI

### 16.1.1 Overview

**Google Vertex AI Agent Builder** provides a comprehensive platform for building, deploying, and evaluating AI agents. The integrated Gen AI Evaluation service offers sophisticated trajectory-based metrics and evaluation capabilities specifically designed for agent workflows.

### 16.1.2 Vertex AI Agent Engine

**Vertex AI Agent Engine** is a set of services enabling developers to deploy, manage, and scale AI agents in production with built-in evaluation capabilities.

**Core Services:**
- **Fully-Managed Runtime**: Production deployment infrastructure
- **Evaluation Service**: Integrated agent evaluation capabilities
- **Sessions**: Stateful conversation management
- **Memory Bank**: Persistent agent memory
- **Code Execution**: Safe code execution environment

**Pricing Note (January 2026):** Sessions, Memory Bank, and Code Execution began charging for usage on January 28, 2026.

### 16.1.3 Agent Evaluation Metrics

The Gen AI Evaluation service provides comprehensive **trajectory-based evaluation metrics** that assess agent behavior at both the action level and overall execution quality.

**Trajectory Match Metrics:**

| Metric | Description | Scoring |
|--------|-------------|---------|
| `trajectory_exact_match` | Predicted trajectory is identical to reference with exact same tool calls in exact same order | 1 if identical, 0 otherwise |
| `trajectory_in_order_match` | Predicted trajectory contains all reference tool calls in same order (may have extra calls) | 1 if contains all in order, 0 otherwise |
| `trajectory_any_order_match` | Predicted trajectory contains all reference tool calls regardless of order (may have extra calls) | 1 if contains all, 0 otherwise |
| `trajectory_single_tool_use` | Checks if a specific tool is used in the predicted trajectory | 1 if tool present, 0 otherwise |

**Trajectory Quality Metrics:**

| Metric | Description | Calculation |
|--------|-------------|-------------|
| `trajectory_precision` | Measures how many predicted actions are correct according to reference | (Matching actions in predicted) / (Total actions in predicted) |
| `trajectory_recall` | Measures how many reference actions are captured in prediction | (Matching actions in reference captured) / (Total actions in reference) |

### 16.1.4 Evaluation Dataset Requirements

To use the evaluation service, datasets must contain:
- **User Prompt**: The input provided to the agent
- **Reference Trajectory**: The expected sequence of actions the agent should take
- **Generated Trajectory**: The actual sequence of actions the agent took
- **Response**: The generated response given the agent's sequence of actions

### 16.1.5 User Simulator

Google acknowledges that evaluating non-deterministic systems presents major challenges. To address this, the **Evaluation Layer** includes a **User Simulator** that enables developers to:
- Simulate realistic user interactions
- Test edge cases and failure scenarios
- Scale evaluation coverage
- Reduce dependence on human testers

### 16.1.6 Tool Governance

The platform includes **Enhanced Tool Governance** features for controlling agent tool usage:
- Define permitted tools per agent
- Set tool usage policies
- Monitor tool call patterns
- Enforce security constraints

```python
# Example: Vertex AI Agent Evaluation
from google.cloud import aiplatform
from vertexai.preview.evaluation import EvalTask

# Define evaluation task
eval_task = EvalTask(
    dataset="gs://my-bucket/agent-eval-dataset.jsonl",
    metrics=[
        "trajectory_exact_match",
        "trajectory_precision",
        "trajectory_recall"
    ]
)

# Run evaluation
results = eval_task.evaluate(
    model="projects/my-project/locations/us-central1/agents/my-agent"
)

print(f"Trajectory Precision: {results.metrics['trajectory_precision']}")
print(f"Trajectory Recall: {results.metrics['trajectory_recall']}")
```

---

## 16.2 Amazon Bedrock

### 16.2.1 Overview

**Amazon Bedrock** provides a comprehensive platform for building generative AI applications with native guardrails and evaluation capabilities. The platform focuses on safety, governance, and integration with the broader AWS ecosystem.

### 16.2.2 Amazon Bedrock Guardrails

**Bedrock Guardrails** implements configurable safeguards to detect and filter harmful content, providing an evaluation-as-protection approach.

**Key Capabilities:**
- **Industry-Leading Safety**: Blocks up to 88% of harmful content
- **Verifiable Explanations**: Mathematically verifiable explanations for validation decisions with 99% accuracy
- **Configurable Policies**: Customizable filtering and governance rules

**Supported Policies:**

| Policy Type | Description |
|-------------|-------------|
| **Content Filters** | Detect and filter harmful text/images across categories: Hate, Insults, Sexual, Violence, Misconduct, Prompt Attack |
| **Denied Topics** | Block specific topic categories |
| **Word Filters** | Blacklist specific terms or patterns |
| **Sensitive Information Filters** | PII detection and masking |
| **Contextual Grounding** | Verify responses are grounded in provided context |
| **Custom Policies** | Define organization-specific rules |

### 16.2.3 Guardrail Evaluation Process

When a guardrail is configured, it evaluates:

1. **Input Evaluation**: Assesses user input prompts against defined policies
2. **Output Evaluation**: Evaluates FM completions against policies
3. **Context Handling**: For RAG applications, can evaluate only user input while discarding system instructions

**Configuration Options:**
- Apply guardrails via `guardrail_config` parameter
- Set output scope to 'FULL' to see all evaluations (not just breaches)
- Enable tracing to view grounding scores and detailed evaluation information

### 16.2.4 Agent Integration

Guardrails integrate with Bedrock Agents through:

**Direct Integration:**
- Associate guardrails when creating or updating agents
- Automatic evaluation of agent inputs and outputs
- Policy enforcement at each interaction

**Framework Integration:**
- Compatible with Strands Agents framework
- Works with agents deployed via Amazon Bedrock AgentCore
- LangChain integration via `langchain-aws`

### 16.2.5 Amazon Bedrock AgentCore Observability

**AgentCore** provides comprehensive observability through integration with third-party platforms:

**Dynatrace Integration (2025-2026):**
- Native, end-to-end observability for AgentCore agents
- Unified tracing with cost and latency analytics
- Guardrail monitoring out of the box
- OpenTelemetry signals with generative AI semantic attributes

**Key Monitoring Capabilities:**
- Token consumption forecasting
- Performance anomaly detection
- Toxicity monitoring
- PII detection tracking
- Denied topics compliance

### 16.2.6 Contextual Grounding

**Contextual Grounding** evaluation ensures responses are factually grounded:

- Evaluates if outputs are supported by provided context
- Returns grounding scores for assessment
- Critical for RAG applications
- Requires tracing enabled to view detailed scores

```python
# Example: Bedrock Agent with Guardrails
import boto3

bedrock_agent = boto3.client('bedrock-agent-runtime')

response = bedrock_agent.invoke_agent(
    agentId='my-agent-id',
    agentAliasId='my-alias',
    sessionId='session-123',
    inputText='What is the refund policy?',
    enableTrace=True,  # Enable to see grounding scores
    guardrailConfiguration={
        'guardrailIdentifier': 'my-guardrail',
        'guardrailVersion': '1'
    }
)

# Access trace for evaluation details
for event in response['completion']:
    if 'trace' in event:
        print(event['trace'])
```

---

## 16.3 Microsoft Azure AI Foundry

### 16.3.1 Overview

**Microsoft Azure AI Foundry** (formerly Azure AI Studio) provides comprehensive evaluation capabilities through the Azure AI Evaluation SDK, with specific focus on agentic application assessment including quality and safety metrics.

### 16.3.2 Agent-Specific Evaluation Metrics

Azure AI Foundry introduced new evaluation metrics specifically designed for agentic applications:

**Quality Metrics:**

| Metric | Description |
|--------|-------------|
| **Intent Resolution** | Measures how well the agent identifies user requests, including scoping intent, asking clarifying questions, and communicating capabilities |
| **Tool Call Accuracy** | Evaluates ability to select appropriate tools, process correct parameters, and call tools in optimal order |
| **Task Adherence** | Measures how well responses adhere to assigned tasks per system message and prior steps |
| **Response Completeness** | Measures comprehensiveness compared to ground truth |
| **Relevance** | Assesses response relevance to the query |
| **Coherence** | Evaluates logical flow and consistency |
| **Fluency** | Measures language quality and readability |
| **Groundedness** | Assesses factual accuracy against provided context |

**Safety Metrics:**

| Metric | Description |
|--------|-------------|
| **Code Vulnerability** | Detects security vulnerabilities in generated code (injection, SQL injection, stack trace exposure) across Python, Java, C++, C#, Go, JavaScript |
| **Content Safety** | Evaluates outputs for harmful content |
| **Indirect Attack** | Detects prompt injection attempts |

### 16.3.3 Azure AI Evaluation SDK

The SDK provides programmatic access to evaluation capabilities:

**Available Evaluators:**
- `IntentResolution`
- `ToolCallAccuracy`
- `TaskAdherence`
- `RelevanceEvaluator`
- `CoherenceEvaluator`
- `GroundednessEvaluator`
- `FluencyEvaluator`
- `CodeVulnerabilityEvaluator`
- `ContentSafetyEvaluator`
- `IndirectAttackEvaluator`

**Agent Framework Support:**
Seamless evaluation support through converters for:
- Microsoft Foundry Agent Service
- Semantic Kernel agents
- Custom agent implementations

### 16.3.4 Continuous Evaluation for Agents

Azure AI Foundry provides **near real-time observability and monitoring** through continuous evaluation:

**Capabilities:**
- Continuous evaluation of agent interactions at configurable sampling rates
- Quality, safety, and performance metric surfacing
- Integration with Foundry Observability dashboard
- Automated alerting on metric degradation

### 16.3.5 Observability Features

**Foundry Control Plane Observability** provides:
- End-to-end tracing of agent workflows
- Performance monitoring and alerting
- Cost tracking and optimization
- Resource utilization insights

**Tracing and Observation:**
The SDK enables detailed tracing of agent interactions:
- Capture all intermediate steps
- Log tool invocations and responses
- Track reasoning chains
- Monitor error patterns

### 16.3.6 Pricing and Availability

**Preview Status:** Billing for managed hosting runtime will be enabled no earlier than February 1, 2026. Quality and evaluation features are available in Preview.

```python
# Example: Azure AI Foundry Agent Evaluation
from azure.ai.evaluation import (
    IntentResolutionEvaluator,
    ToolCallAccuracyEvaluator,
    TaskAdherenceEvaluator
)

# Initialize evaluators
intent_evaluator = IntentResolutionEvaluator(
    credential=DefaultAzureCredential()
)
tool_evaluator = ToolCallAccuracyEvaluator(
    credential=DefaultAzureCredential()
)

# Evaluate agent response
intent_result = intent_evaluator.evaluate(
    query="What's the weather like today?",
    response=agent_response,
    conversation_history=conversation
)

tool_result = tool_evaluator.evaluate(
    tools_called=agent_tools_called,
    expected_tools=["weather_api"],
    tool_parameters=tool_params
)

print(f"Intent Resolution Score: {intent_result.score}")
print(f"Tool Call Accuracy: {tool_result.score}")
```

---

## 16.4 Provider Comparison Matrix

### 16.4.1 Evaluation Capabilities

| Capability | Google Vertex AI | Amazon Bedrock | Azure AI Foundry |
|------------|-----------------|----------------|------------------|
| **Trajectory Evaluation** | ✅ Native | ❌ Via third-party | ❌ Via third-party |
| **Tool Call Accuracy** | ✅ Via trajectory | ❌ | ✅ Native metric |
| **Intent Resolution** | ❌ | ❌ | ✅ Native metric |
| **Task Adherence** | ❌ | ❌ | ✅ Native metric |
| **Content Safety** | ✅ | ✅ Guardrails | ✅ Native evaluator |
| **Grounding/Faithfulness** | ✅ | ✅ Contextual | ✅ Groundedness |
| **Code Vulnerability** | ❌ | ❌ | ✅ Native evaluator |
| **User Simulation** | ✅ | ❌ | ❌ |
| **Continuous Evaluation** | ✅ | Via AgentCore | ✅ Native |

### 16.4.2 Safety and Governance

| Feature | Google Vertex AI | Amazon Bedrock | Azure AI Foundry |
|---------|-----------------|----------------|------------------|
| **Guardrails/Safety Filters** | ✅ | ✅ Industry-leading | ✅ |
| **PII Detection** | ✅ | ✅ | ✅ |
| **Custom Policies** | ✅ | ✅ | ✅ |
| **Prompt Injection Detection** | ✅ | ✅ | ✅ |
| **Harmful Content Filtering** | ✅ | ✅ (88% block rate) | ✅ |
| **Audit Logging** | ✅ | ✅ | ✅ |

### 16.4.3 Integration and Ecosystem

| Aspect | Google Vertex AI | Amazon Bedrock | Azure AI Foundry |
|--------|-----------------|----------------|------------------|
| **Framework Support** | LangChain, LlamaIndex | LangChain, Strands | Semantic Kernel, LangChain |
| **OpenTelemetry** | ✅ | Via Dynatrace | ✅ |
| **Self-Hosted Option** | ❌ Cloud only | ❌ Cloud only | ❌ Cloud only |
| **SDK Languages** | Python, Java, Go, Node.js | Python, Java, JavaScript | Python, C#, JavaScript |
| **Third-Party Observability** | Various | Dynatrace, Arize | Various |

### 16.4.4 Unique Strengths

**Google Vertex AI:**
- Most comprehensive trajectory-based evaluation metrics
- Native user simulation for scalable testing
- Deep integration with Google AI ecosystem
- Enhanced tool governance capabilities

**Amazon Bedrock:**
- Industry-leading safety with 88% harmful content blocking
- Mathematically verifiable explanations (99% accuracy)
- Broadest model selection across providers
- Strong enterprise governance features

**Azure AI Foundry:**
- Purpose-built agentic metrics (Intent Resolution, Tool Call Accuracy)
- Code vulnerability detection across multiple languages
- Deep integration with Microsoft development tools
- Strong enterprise security and compliance

---

## Key Takeaways

1. **Trajectory evaluation is differentiating**: Google Vertex AI leads with comprehensive trajectory-based metrics; other providers require third-party tools for similar capabilities

2. **Safety features are table stakes**: All three providers offer robust guardrails and content filtering, with Bedrock claiming highest harmful content blocking rates

3. **Agent-specific metrics emerge**: Azure AI Foundry's Intent Resolution and Tool Call Accuracy metrics represent the industry's recognition that agents need specialized evaluation

4. **Continuous evaluation is essential**: Both Google and Azure offer native continuous evaluation; Bedrock requires third-party integration (Dynatrace)

5. **Cloud lock-in considerations**: All three platforms are cloud-only with no self-hosted options—consider this for data residency and compliance requirements

6. **Complementary approaches**: Consider combining cloud provider evaluation with open-source frameworks for comprehensive coverage

---

## References

1. Google Cloud. (2026). *Vertex AI Agent Builder Overview*. [https://docs.cloud.google.com/agent-builder/overview](https://docs.cloud.google.com/agent-builder/overview)

2. Google Cloud. (2025). *Evaluate Gen AI Agents*. [https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-agents](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-agents)

3. Google Cloud. (2025). *Evaluate Your AI Agents with Vertex Gen AI Evaluation Service*. [https://cloud.google.com/blog/products/ai-machine-learning/introducing-agent-evaluation-in-vertex-ai-gen-ai-evaluation-service](https://cloud.google.com/blog/products/ai-machine-learning/introducing-agent-evaluation-in-vertex-ai-gen-ai-evaluation-service)

4. Google Cloud. (2025). *Enhanced Tool Governance in Vertex AI Agent Builder*. [https://cloud.google.com/blog/products/ai-machine-learning/new-enhanced-tool-governance-in-vertex-ai-agent-builder](https://cloud.google.com/blog/products/ai-machine-learning/new-enhanced-tool-governance-in-vertex-ai-agent-builder)

5. AWS. (2026). *Amazon Bedrock Guardrails*. [https://aws.amazon.com/bedrock/guardrails/](https://aws.amazon.com/bedrock/guardrails/)

6. AWS. (2025). *Implement Safeguards for Your Application with Guardrails*. [https://docs.aws.amazon.com/bedrock/latest/userguide/agents-guardrail.html](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-guardrail.html)

7. Dynatrace. (2025). *Announcing Amazon Bedrock AgentCore Agent Observability*. [https://www.dynatrace.com/news/blog/announcing-amazon-bedrock-agentcore-agent-observability/](https://www.dynatrace.com/news/blog/announcing-amazon-bedrock-agentcore-agent-observability/)

8. Microsoft. (2025). *Agent Evaluation with the Microsoft Foundry SDK*. [https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/agent-evaluate-sdk](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/agent-evaluate-sdk)

9. Microsoft. (2025). *Unlocking the Power of Agentic Applications: New Evaluation Metrics*. [https://devblogs.microsoft.com/foundry/evaluation-metrics-azure-ai-foundry/](https://devblogs.microsoft.com/foundry/evaluation-metrics-azure-ai-foundry/)

10. Microsoft. (2025). *Continuously Evaluate Your AI Agents*. [https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/continuous-evaluation-agents](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/continuous-evaluation-agents)

11. Arize AI. (2025). *Evaluating and Improving AI Agents at Scale with Microsoft Foundry*. [https://arize.com/blog/evaluating-and-improving-ai-agents-at-scale-with-microsoft-foundry/](https://arize.com/blog/evaluating-and-improving-ai-agents-at-scale-with-microsoft-foundry/)

12. InfoWorld. (2025). *Google Boosts Vertex AI Agent Builder with New Observability and Deployment Tools*. [https://www.infoworld.com/article/4085736/google-boosts-vertex-ai-agent-builder-with-new-observability-and-deployment-tools.html](https://www.infoworld.com/article/4085736/google-boosts-vertex-ai-agent-builder-with-new-observability-and-deployment-tools.html)

---

**Navigation:**
← [Previous Section: Evaluation Frameworks and Libraries](15_evaluation_frameworks_and_libraries.md) | [Table of Contents](../README.md) | [Next Section: Observability Features in AI Development Frameworks](17_observability_features_ai_frameworks.md) →
