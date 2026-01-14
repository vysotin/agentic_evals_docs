# Appendix E: Code Examples

> **Appendices**
> **Reference Document**

## Overview

This appendix provides comprehensive, production-ready code examples for implementing AI agent evaluation, observability, and monitoring. Examples cover the major frameworks (DeepEval, RAGAS, OpenLLMetry) and cloud providers (Azure, AWS, GCP), with complete implementations you can adapt to your needs.

---

## Table of Contents

- [E.1 Basic Setup and Configuration](#e1-basic-setup-and-configuration)
- [E.2 DeepEval Agent Evaluation](#e2-deepeval-agent-evaluation)
- [E.3 RAGAS RAG Evaluation](#e3-ragas-rag-evaluation)
- [E.4 OpenTelemetry Instrumentation](#e4-opentelemetry-instrumentation)
- [E.5 LLM-as-Judge Implementation](#e5-llm-as-judge-implementation)
- [E.6 CI/CD Integration](#e6-cicd-integration)
- [E.7 Production Monitoring](#e7-production-monitoring)
- [E.8 Custom Metric Implementation](#e8-custom-metric-implementation)

---

## E.1 Basic Setup and Configuration

### E.1.1 Environment Setup

```bash
# Create virtual environment
python -m venv agent-eval-env
source agent-eval-env/bin/activate  # On Windows: agent-eval-env\Scripts\activate

# Install core packages
pip install deepeval ragas langchain openai anthropic

# Install observability packages
pip install opentelemetry-api opentelemetry-sdk opentelemetry-exporter-otlp
pip install openllmetry-sdk traceloop-sdk

# Install Langfuse client
pip install langfuse
```

### E.1.2 Configuration File

```python
# config.py
import os
from dataclasses import dataclass
from typing import Optional

@dataclass
class EvaluationConfig:
    """Configuration for evaluation pipeline"""
    # API Keys
    openai_api_key: str = os.getenv("OPENAI_API_KEY", "")
    anthropic_api_key: str = os.getenv("ANTHROPIC_API_KEY", "")

    # Evaluation Settings
    judge_model: str = "gpt-4o"
    evaluation_threshold: float = 0.8

    # Observability Settings
    langfuse_secret_key: str = os.getenv("LANGFUSE_SECRET_KEY", "")
    langfuse_public_key: str = os.getenv("LANGFUSE_PUBLIC_KEY", "")
    langfuse_host: str = os.getenv("LANGFUSE_HOST", "https://cloud.langfuse.com")

    # OTLP Settings
    otlp_endpoint: str = os.getenv("OTLP_ENDPOINT", "http://localhost:4317")

    # Testing Settings
    test_data_path: str = "./test_data"
    results_path: str = "./results"

config = EvaluationConfig()
```

---

## E.2 DeepEval Agent Evaluation

### E.2.1 Basic Agent Evaluation

```python
# deepeval_basic.py
from deepeval import evaluate
from deepeval.test_case import LLMTestCase
from deepeval.metrics import (
    TaskCompletionMetric,
    ToolCorrectnessMetric,
    AnswerRelevancyMetric,
    FaithfulnessMetric
)

def evaluate_agent_response(
    user_query: str,
    agent_response: str,
    tools_called: list,
    expected_tools: list,
    retrieval_context: list = None
) -> dict:
    """
    Evaluate a single agent response using DeepEval metrics.

    Args:
        user_query: The user's input query
        agent_response: The agent's response
        tools_called: List of tools the agent called
        expected_tools: List of tools that should have been called
        retrieval_context: Retrieved context for RAG evaluation

    Returns:
        Dictionary containing evaluation results
    """
    # Create test case
    test_case = LLMTestCase(
        input=user_query,
        actual_output=agent_response,
        expected_output=None,
        tools_called=tools_called,
        expected_tools=expected_tools,
        retrieval_context=retrieval_context or []
    )

    # Define metrics
    metrics = [
        TaskCompletionMetric(threshold=0.7, model="gpt-4o"),
        ToolCorrectnessMetric(threshold=0.8),
        AnswerRelevancyMetric(threshold=0.7, model="gpt-4o"),
    ]

    # Add faithfulness if context provided
    if retrieval_context:
        metrics.append(FaithfulnessMetric(threshold=0.8, model="gpt-4o"))

    # Run evaluation
    results = evaluate([test_case], metrics)

    # Format results
    return {
        "passed": all(m.score >= m.threshold for m in metrics),
        "scores": {
            m.__class__.__name__: {
                "score": m.score,
                "threshold": m.threshold,
                "passed": m.score >= m.threshold,
                "reason": getattr(m, 'reason', None)
            }
            for m in metrics
        },
        "test_case_id": test_case.id
    }

# Example usage
if __name__ == "__main__":
    result = evaluate_agent_response(
        user_query="What is the refund policy for electronics?",
        agent_response="Our electronics refund policy allows returns within 30 days with receipt.",
        tools_called=["search_policy_docs", "get_category_rules"],
        expected_tools=["search_policy_docs", "get_category_rules"],
        retrieval_context=[
            "Policy: Electronics may be returned within 30 days of purchase with original receipt.",
            "Policy: All returns require proof of purchase."
        ]
    )
    print(f"Evaluation Result: {result}")
```

### E.2.2 Conversational Agent Evaluation

```python
# deepeval_conversation.py
from deepeval import evaluate
from deepeval.test_case import ConversationalTestCase, LLMTestCase
from deepeval.metrics import ConversationCompletenessMetric

def evaluate_conversation(turns: list[dict]) -> dict:
    """
    Evaluate a multi-turn conversation.

    Args:
        turns: List of conversation turns with 'user' and 'agent' keys

    Returns:
        Evaluation results for the conversation
    """
    # Convert to DeepEval format
    test_case_turns = [
        LLMTestCase(
            input=turn["user"],
            actual_output=turn["agent"]
        )
        for turn in turns
    ]

    conv_test_case = ConversationalTestCase(turns=test_case_turns)

    # Evaluate conversation completeness
    metric = ConversationCompletenessMetric(
        threshold=0.7,
        model="gpt-4o"
    )

    results = evaluate([conv_test_case], [metric])

    return {
        "conversation_score": metric.score,
        "passed": metric.score >= metric.threshold,
        "reason": metric.reason
    }

# Example usage
if __name__ == "__main__":
    conversation = [
        {
            "user": "I need help with my order",
            "agent": "I'd be happy to help with your order. Could you provide your order number?"
        },
        {
            "user": "It's #12345",
            "agent": "Thank you. I found order #12345. It's currently in transit and expected to arrive tomorrow."
        },
        {
            "user": "Can I change the delivery address?",
            "agent": "I've updated the delivery address to your new location. You'll receive a confirmation email shortly."
        }
    ]

    result = evaluate_conversation(conversation)
    print(f"Conversation Score: {result['conversation_score']:.2f}")
```

### E.2.3 Batch Evaluation with pytest

```python
# test_agent_evaluation.py
import pytest
from deepeval import assert_test
from deepeval.test_case import LLMTestCase
from deepeval.metrics import TaskCompletionMetric, AnswerRelevancyMetric
from deepeval.dataset import EvaluationDataset

# Load test dataset
@pytest.fixture
def test_dataset():
    """Load evaluation dataset from file or create programmatically"""
    return EvaluationDataset(
        test_cases=[
            LLMTestCase(
                input="What are your business hours?",
                actual_output="We are open Monday-Friday 9am-5pm EST.",
                expected_output="Business hours are Monday-Friday 9am-5pm EST."
            ),
            LLMTestCase(
                input="How do I reset my password?",
                actual_output="Go to Settings > Security > Reset Password and follow the prompts.",
                expected_output="Navigate to account settings to reset password."
            ),
            # Add more test cases...
        ]
    )

@pytest.mark.parametrize(
    "test_case",
    [
        LLMTestCase(
            input="What is your return policy?",
            actual_output="We offer 30-day returns with full refund.",
        ),
    ]
)
def test_task_completion(test_case):
    """Test that agent completes tasks successfully"""
    metric = TaskCompletionMetric(threshold=0.7)
    assert_test(test_case, [metric])

@pytest.mark.parametrize(
    "test_case",
    [
        LLMTestCase(
            input="Explain quantum computing",
            actual_output="Quantum computing uses quantum bits (qubits) to perform calculations.",
        ),
    ]
)
def test_answer_relevancy(test_case):
    """Test that responses are relevant to queries"""
    metric = AnswerRelevancyMetric(threshold=0.7)
    assert_test(test_case, [metric])

def test_full_dataset(test_dataset):
    """Run evaluation on full test dataset"""
    metrics = [
        TaskCompletionMetric(threshold=0.7),
        AnswerRelevancyMetric(threshold=0.7)
    ]

    results = []
    for test_case in test_dataset.test_cases:
        try:
            assert_test(test_case, metrics)
            results.append({"passed": True})
        except AssertionError as e:
            results.append({"passed": False, "error": str(e)})

    pass_rate = sum(1 for r in results if r["passed"]) / len(results)
    assert pass_rate >= 0.8, f"Pass rate {pass_rate:.2%} below threshold 80%"
```

---

## E.3 RAGAS RAG Evaluation

### E.3.1 Basic RAG Evaluation

```python
# ragas_evaluation.py
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_precision,
    context_recall,
    answer_correctness
)
from datasets import Dataset

def evaluate_rag_response(
    question: str,
    answer: str,
    contexts: list[str],
    ground_truth: str = None
) -> dict:
    """
    Evaluate a RAG response using RAGAS metrics.

    Args:
        question: User's question
        answer: Generated answer
        contexts: Retrieved contexts used to generate answer
        ground_truth: Expected correct answer (optional)

    Returns:
        Dictionary of metric scores
    """
    # Prepare data in RAGAS format
    data = {
        "question": [question],
        "answer": [answer],
        "contexts": [contexts],
    }

    if ground_truth:
        data["ground_truth"] = [ground_truth]

    dataset = Dataset.from_dict(data)

    # Select metrics based on available data
    metrics_to_use = [
        faithfulness,
        answer_relevancy,
        context_precision,
    ]

    if ground_truth:
        metrics_to_use.extend([context_recall, answer_correctness])

    # Run evaluation
    result = evaluate(dataset, metrics=metrics_to_use)

    return result.to_pandas().to_dict(orient="records")[0]

# Example usage
if __name__ == "__main__":
    result = evaluate_rag_response(
        question="What is the maximum file upload size?",
        answer="The maximum file upload size is 100MB according to our upload policy.",
        contexts=[
            "Upload Policy v2.3: Maximum file size for uploads is 100MB.",
            "Files exceeding the limit will be rejected with error code 413."
        ],
        ground_truth="The maximum file upload size is 100MB."
    )

    print("RAG Evaluation Results:")
    for metric, score in result.items():
        print(f"  {metric}: {score:.3f}")
```

### E.3.2 Batch RAG Evaluation

```python
# ragas_batch.py
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision
from datasets import Dataset
import pandas as pd
import json

def load_test_data(filepath: str) -> list[dict]:
    """Load test data from JSON file"""
    with open(filepath, 'r') as f:
        return json.load(f)

def evaluate_rag_batch(test_data: list[dict]) -> pd.DataFrame:
    """
    Evaluate a batch of RAG responses.

    Args:
        test_data: List of dicts with question, answer, contexts keys

    Returns:
        DataFrame with evaluation results
    """
    # Prepare data
    data = {
        "question": [d["question"] for d in test_data],
        "answer": [d["answer"] for d in test_data],
        "contexts": [d["contexts"] for d in test_data],
    }

    # Add ground truth if available
    if "ground_truth" in test_data[0]:
        data["ground_truth"] = [d["ground_truth"] for d in test_data]

    dataset = Dataset.from_dict(data)

    # Evaluate
    metrics = [faithfulness, answer_relevancy, context_precision]
    result = evaluate(dataset, metrics=metrics)

    return result.to_pandas()

def generate_report(results_df: pd.DataFrame, output_path: str):
    """Generate evaluation report"""
    report = {
        "summary": {
            "total_samples": len(results_df),
            "avg_faithfulness": results_df["faithfulness"].mean(),
            "avg_answer_relevancy": results_df["answer_relevancy"].mean(),
            "avg_context_precision": results_df["context_precision"].mean(),
        },
        "thresholds": {
            "faithfulness": 0.8,
            "answer_relevancy": 0.7,
            "context_precision": 0.7,
        },
        "pass_rates": {
            "faithfulness": (results_df["faithfulness"] >= 0.8).mean(),
            "answer_relevancy": (results_df["answer_relevancy"] >= 0.7).mean(),
            "context_precision": (results_df["context_precision"] >= 0.7).mean(),
        }
    }

    # Save report
    with open(output_path, 'w') as f:
        json.dump(report, f, indent=2)

    # Save detailed results
    results_df.to_csv(output_path.replace('.json', '_detailed.csv'), index=False)

    return report

# Example usage
if __name__ == "__main__":
    # Sample test data
    test_data = [
        {
            "question": "What is the refund policy?",
            "answer": "We offer full refunds within 30 days of purchase.",
            "contexts": ["Refund Policy: Full refunds available within 30 days."],
            "ground_truth": "30-day full refund policy"
        },
        {
            "question": "How do I contact support?",
            "answer": "Contact support at support@example.com or call 1-800-EXAMPLE.",
            "contexts": ["Support: Email support@example.com or call 1-800-EXAMPLE"],
            "ground_truth": "Email or phone support available"
        }
    ]

    results = evaluate_rag_batch(test_data)
    report = generate_report(results, "rag_evaluation_report.json")
    print(f"Average Faithfulness: {report['summary']['avg_faithfulness']:.3f}")
```

---

## E.4 OpenTelemetry Instrumentation

### E.4.1 Basic OpenLLMetry Setup

```python
# otel_setup.py
from traceloop.sdk import Traceloop
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

def setup_tracing(
    service_name: str = "my-agent",
    otlp_endpoint: str = "http://localhost:4317"
):
    """
    Set up OpenTelemetry tracing with OpenLLMetry.

    Args:
        service_name: Name of the service for traces
        otlp_endpoint: OTLP collector endpoint
    """
    # Initialize Traceloop (OpenLLMetry)
    Traceloop.init(
        app_name=service_name,
        api_endpoint=otlp_endpoint,
        disable_batch=False
    )

    print(f"Tracing initialized for {service_name}")

def setup_manual_tracing(
    service_name: str = "my-agent",
    otlp_endpoint: str = "http://localhost:4317"
):
    """Manual OpenTelemetry setup without Traceloop"""
    # Create tracer provider
    provider = TracerProvider()

    # Add OTLP exporter
    otlp_exporter = OTLPSpanExporter(endpoint=otlp_endpoint)
    provider.add_span_processor(BatchSpanProcessor(otlp_exporter))

    # Set as global provider
    trace.set_tracer_provider(provider)

    return trace.get_tracer(service_name)

# Example usage with tracing
if __name__ == "__main__":
    setup_tracing()

    # Now any LLM calls will be automatically traced
    from openai import OpenAI

    client = OpenAI()
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Hello!"}]
    )

    print(response.choices[0].message.content)
```

### E.4.2 Langfuse Integration

```python
# langfuse_integration.py
from langfuse import Langfuse
from langfuse.decorators import observe, langfuse_context
import os

# Initialize Langfuse
langfuse = Langfuse(
    secret_key=os.getenv("LANGFUSE_SECRET_KEY"),
    public_key=os.getenv("LANGFUSE_PUBLIC_KEY"),
    host=os.getenv("LANGFUSE_HOST", "https://cloud.langfuse.com")
)

@observe()
def process_user_query(query: str) -> str:
    """Process a user query with automatic tracing"""
    # This entire function is traced

    # Add custom metadata
    langfuse_context.update_current_observation(
        metadata={"query_type": classify_query(query)}
    )

    # Call your agent
    result = call_agent(query)

    # Add score for this generation
    langfuse_context.score_current_observation(
        name="response_quality",
        value=0.9,
        comment="High quality response"
    )

    return result

@observe(as_type="generation")
def call_llm(prompt: str, model: str = "gpt-4o") -> str:
    """Call LLM with generation-specific tracing"""
    from openai import OpenAI
    client = OpenAI()

    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )

    return response.choices[0].message.content

@observe()
def call_agent(query: str) -> str:
    """Multi-step agent with traced sub-operations"""
    # Each nested @observe creates child spans
    context = retrieve_context(query)
    response = call_llm(f"Context: {context}\n\nQuery: {query}")
    return response

@observe(as_type="retrieval")
def retrieve_context(query: str) -> str:
    """Retrieve context with retrieval-specific tracing"""
    # Simulated retrieval
    return "Retrieved context here..."

def classify_query(query: str) -> str:
    """Classify query type for metadata"""
    if "help" in query.lower():
        return "support"
    elif "?" in query:
        return "question"
    else:
        return "statement"

# Example usage
if __name__ == "__main__":
    result = process_user_query("How do I reset my password?")
    print(f"Result: {result}")

    # Flush to ensure traces are sent
    langfuse.flush()
```

### E.4.3 Custom Span Attributes

```python
# custom_spans.py
from opentelemetry import trace
from opentelemetry.trace import Status, StatusCode
from contextlib import contextmanager
import time

tracer = trace.get_tracer(__name__)

@contextmanager
def trace_agent_step(step_name: str, step_type: str = "generic"):
    """
    Context manager for tracing agent steps with custom attributes.

    Args:
        step_name: Name of the step
        step_type: Type of step (planning, tool_call, reasoning, etc.)
    """
    with tracer.start_as_current_span(step_name) as span:
        start_time = time.time()

        # Set standard attributes
        span.set_attribute("agent.step.type", step_type)
        span.set_attribute("agent.step.name", step_name)

        try:
            yield span

            # Success
            span.set_status(Status(StatusCode.OK))

        except Exception as e:
            # Error handling
            span.set_status(Status(StatusCode.ERROR, str(e)))
            span.record_exception(e)
            raise

        finally:
            # Record duration
            duration = time.time() - start_time
            span.set_attribute("agent.step.duration_ms", duration * 1000)

def trace_tool_call(tool_name: str, tool_input: dict, tool_output: any):
    """Record a tool call with full details"""
    with tracer.start_as_current_span(f"tool_call.{tool_name}") as span:
        span.set_attribute("gen_ai.tool.name", tool_name)
        span.set_attribute("gen_ai.tool.input", str(tool_input))
        span.set_attribute("gen_ai.tool.output", str(tool_output)[:1000])  # Truncate
        span.set_attribute("gen_ai.tool.success", True)

def trace_llm_call(
    model: str,
    prompt: str,
    response: str,
    tokens_input: int,
    tokens_output: int
):
    """Record an LLM call with token metrics"""
    with tracer.start_as_current_span("llm_call") as span:
        span.set_attribute("gen_ai.system", "openai")
        span.set_attribute("gen_ai.request.model", model)
        span.set_attribute("gen_ai.usage.input_tokens", tokens_input)
        span.set_attribute("gen_ai.usage.output_tokens", tokens_output)
        span.set_attribute("gen_ai.usage.total_tokens", tokens_input + tokens_output)
        # Don't log full prompt/response in production for privacy
        span.set_attribute("gen_ai.prompt.length", len(prompt))
        span.set_attribute("gen_ai.response.length", len(response))

# Example usage
if __name__ == "__main__":
    with trace_agent_step("process_query", "main"):
        with trace_agent_step("planning", "planning"):
            # Planning logic
            pass

        with trace_agent_step("tool_execution", "tool_call"):
            trace_tool_call(
                tool_name="search_docs",
                tool_input={"query": "refund policy"},
                tool_output={"results": ["doc1", "doc2"]}
            )

        with trace_agent_step("response_generation", "generation"):
            trace_llm_call(
                model="gpt-4o",
                prompt="Generate response...",
                response="Here is your answer...",
                tokens_input=100,
                tokens_output=50
            )
```

---

## E.5 LLM-as-Judge Implementation

### E.5.1 Custom LLM Judge

```python
# llm_judge.py
from openai import OpenAI
from dataclasses import dataclass
from typing import Optional
import json

@dataclass
class JudgeResult:
    score: float
    reasoning: str
    passed: bool
    raw_response: dict

class LLMJudge:
    """Custom LLM-as-Judge implementation"""

    def __init__(
        self,
        model: str = "gpt-4o",
        temperature: float = 0.0
    ):
        self.client = OpenAI()
        self.model = model
        self.temperature = temperature

    def evaluate(
        self,
        criterion: str,
        rubric: str,
        user_query: str,
        agent_response: str,
        context: Optional[str] = None,
        threshold: float = 0.7
    ) -> JudgeResult:
        """
        Evaluate a response using LLM-as-Judge.

        Args:
            criterion: What to evaluate (e.g., "relevance", "accuracy")
            rubric: Scoring guidelines
            user_query: The user's original query
            agent_response: The agent's response to evaluate
            context: Additional context if available
            threshold: Pass/fail threshold (0-1)

        Returns:
            JudgeResult with score, reasoning, and pass status
        """
        prompt = self._build_prompt(
            criterion, rubric, user_query, agent_response, context
        )

        response = self.client.chat.completions.create(
            model=self.model,
            temperature=self.temperature,
            messages=[
                {"role": "system", "content": "You are an expert evaluator."},
                {"role": "user", "content": prompt}
            ],
            response_format={"type": "json_object"}
        )

        result = json.loads(response.choices[0].message.content)
        score = result.get("score", 0) / 5  # Normalize to 0-1

        return JudgeResult(
            score=score,
            reasoning=result.get("reasoning", ""),
            passed=score >= threshold,
            raw_response=result
        )

    def _build_prompt(
        self,
        criterion: str,
        rubric: str,
        user_query: str,
        agent_response: str,
        context: Optional[str]
    ) -> str:
        """Build the evaluation prompt"""
        prompt = f"""
Evaluate the following AI response for {criterion}.

SCORING RUBRIC:
{rubric}

USER QUERY:
{user_query}

AGENT RESPONSE:
{agent_response}
"""
        if context:
            prompt += f"""
CONTEXT PROVIDED:
{context}
"""

        prompt += """
Provide your evaluation as JSON with the following structure:
{
    "score": <1-5 score>,
    "reasoning": "<detailed explanation of your score>",
    "strengths": ["<list of strengths>"],
    "weaknesses": ["<list of weaknesses>"]
}
"""
        return prompt

# Predefined rubrics
RUBRICS = {
    "relevance": """
1 - Completely irrelevant to the query
2 - Tangentially related but misses the point
3 - Partially addresses the query
4 - Addresses the query well with minor gaps
5 - Perfectly addresses the query
""",
    "accuracy": """
1 - Contains major factual errors
2 - Contains some factual errors
3 - Mostly accurate with minor issues
4 - Accurate with negligible issues
5 - Completely accurate
""",
    "helpfulness": """
1 - Not helpful at all
2 - Minimally helpful
3 - Somewhat helpful
4 - Helpful with actionable information
5 - Extremely helpful and comprehensive
"""
}

# Example usage
if __name__ == "__main__":
    judge = LLMJudge()

    result = judge.evaluate(
        criterion="relevance",
        rubric=RUBRICS["relevance"],
        user_query="What is your return policy?",
        agent_response="We offer 30-day returns with full refund for most items.",
        threshold=0.7
    )

    print(f"Score: {result.score:.2f}")
    print(f"Passed: {result.passed}")
    print(f"Reasoning: {result.reasoning}")
```

### E.5.2 Multi-Criteria Evaluation

```python
# multi_criteria_judge.py
from llm_judge import LLMJudge, RUBRICS, JudgeResult
from dataclasses import dataclass
from typing import Dict, List

@dataclass
class MultiCriteriaResult:
    overall_score: float
    overall_passed: bool
    criteria_scores: Dict[str, JudgeResult]
    weighted_score: float

class MultiCriteriaJudge:
    """Evaluate responses across multiple criteria"""

    def __init__(self, model: str = "gpt-4o"):
        self.judge = LLMJudge(model=model)

        # Default criteria weights
        self.default_weights = {
            "relevance": 0.3,
            "accuracy": 0.3,
            "helpfulness": 0.2,
            "clarity": 0.2
        }

    def evaluate(
        self,
        user_query: str,
        agent_response: str,
        criteria: List[str] = None,
        weights: Dict[str, float] = None,
        context: str = None,
        threshold: float = 0.7
    ) -> MultiCriteriaResult:
        """
        Evaluate response across multiple criteria.

        Args:
            user_query: User's query
            agent_response: Agent's response
            criteria: List of criteria to evaluate
            weights: Weights for each criterion (must sum to 1)
            context: Additional context
            threshold: Pass threshold for each criterion

        Returns:
            MultiCriteriaResult with individual and aggregate scores
        """
        criteria = criteria or list(self.default_weights.keys())
        weights = weights or {c: self.default_weights.get(c, 0.25) for c in criteria}

        # Normalize weights
        total_weight = sum(weights.values())
        weights = {k: v/total_weight for k, v in weights.items()}

        # Evaluate each criterion
        criteria_scores = {}
        for criterion in criteria:
            rubric = RUBRICS.get(criterion, RUBRICS["relevance"])
            result = self.judge.evaluate(
                criterion=criterion,
                rubric=rubric,
                user_query=user_query,
                agent_response=agent_response,
                context=context,
                threshold=threshold
            )
            criteria_scores[criterion] = result

        # Calculate weighted score
        weighted_score = sum(
            criteria_scores[c].score * weights[c]
            for c in criteria
        )

        # Calculate overall pass (all criteria must pass)
        overall_passed = all(r.passed for r in criteria_scores.values())

        return MultiCriteriaResult(
            overall_score=sum(r.score for r in criteria_scores.values()) / len(criteria),
            overall_passed=overall_passed,
            criteria_scores=criteria_scores,
            weighted_score=weighted_score
        )

# Example usage
if __name__ == "__main__":
    judge = MultiCriteriaJudge()

    result = judge.evaluate(
        user_query="How do I cancel my subscription?",
        agent_response="To cancel your subscription, go to Account Settings > Subscription > Cancel. Your access will continue until the end of your billing period.",
        criteria=["relevance", "helpfulness", "clarity"],
        weights={"relevance": 0.4, "helpfulness": 0.4, "clarity": 0.2}
    )

    print(f"Weighted Score: {result.weighted_score:.2f}")
    print(f"Overall Passed: {result.overall_passed}")
    for criterion, score in result.criteria_scores.items():
        print(f"  {criterion}: {score.score:.2f} ({'PASS' if score.passed else 'FAIL'})")
```

---

## E.6 CI/CD Integration

### E.6.1 GitHub Actions Workflow

```yaml
# .github/workflows/agent-evaluation.yml
name: Agent Evaluation

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  evaluate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install deepeval ragas pytest

      - name: Run evaluations
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          LANGFUSE_SECRET_KEY: ${{ secrets.LANGFUSE_SECRET_KEY }}
          LANGFUSE_PUBLIC_KEY: ${{ secrets.LANGFUSE_PUBLIC_KEY }}
        run: |
          pytest tests/evaluation/ -v --tb=short

      - name: Generate report
        if: always()
        run: |
          python scripts/generate_eval_report.py

      - name: Upload results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: evaluation-results
          path: results/

      - name: Comment PR with results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('results/summary.json'));

            const body = `## Agent Evaluation Results

            | Metric | Score | Status |
            |--------|-------|--------|
            | Task Completion | ${report.task_completion.toFixed(2)} | ${report.task_completion >= 0.8 ? '✅' : '❌'} |
            | Response Quality | ${report.response_quality.toFixed(2)} | ${report.response_quality >= 0.7 ? '✅' : '❌'} |
            | Safety | ${report.safety.toFixed(2)} | ${report.safety >= 0.95 ? '✅' : '❌'} |

            **Overall: ${report.overall_passed ? '✅ PASSED' : '❌ FAILED'}**
            `;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
```

### E.6.2 Evaluation Script for CI

```python
# scripts/run_evaluation.py
import argparse
import json
import sys
from pathlib import Path
from deepeval import evaluate
from deepeval.test_case import LLMTestCase
from deepeval.metrics import TaskCompletionMetric, AnswerRelevancyMetric
from deepeval.dataset import EvaluationDataset

def load_test_cases(data_path: str) -> EvaluationDataset:
    """Load test cases from JSON file"""
    with open(data_path, 'r') as f:
        data = json.load(f)

    test_cases = [
        LLMTestCase(
            input=tc["input"],
            actual_output=tc["actual_output"],
            expected_output=tc.get("expected_output"),
            retrieval_context=tc.get("retrieval_context", [])
        )
        for tc in data["test_cases"]
    ]

    return EvaluationDataset(test_cases=test_cases)

def run_evaluation(dataset: EvaluationDataset, thresholds: dict) -> dict:
    """Run evaluation and return results"""
    metrics = [
        TaskCompletionMetric(threshold=thresholds.get("task_completion", 0.7)),
        AnswerRelevancyMetric(threshold=thresholds.get("relevance", 0.7)),
    ]

    results = evaluate(dataset, metrics)

    # Calculate summary
    summary = {
        "total_tests": len(dataset.test_cases),
        "passed_tests": sum(1 for tc in results if all(m.passed for m in tc.metrics)),
        "metrics": {}
    }

    for metric in metrics:
        metric_name = metric.__class__.__name__
        scores = [tc.metrics[metric_name].score for tc in results if metric_name in tc.metrics]
        summary["metrics"][metric_name] = {
            "avg_score": sum(scores) / len(scores) if scores else 0,
            "min_score": min(scores) if scores else 0,
            "max_score": max(scores) if scores else 0,
            "pass_rate": sum(1 for s in scores if s >= metric.threshold) / len(scores) if scores else 0
        }

    summary["overall_pass_rate"] = summary["passed_tests"] / summary["total_tests"]
    summary["overall_passed"] = summary["overall_pass_rate"] >= 0.8

    return summary

def main():
    parser = argparse.ArgumentParser(description="Run agent evaluation")
    parser.add_argument("--data", required=True, help="Path to test data JSON")
    parser.add_argument("--output", default="results/summary.json", help="Output path")
    parser.add_argument("--threshold", type=float, default=0.8, help="Pass threshold")
    args = parser.parse_args()

    # Load and run
    dataset = load_test_cases(args.data)
    results = run_evaluation(dataset, {"task_completion": args.threshold})

    # Save results
    output_path = Path(args.output)
    output_path.parent.mkdir(parents=True, exist_ok=True)
    with open(output_path, 'w') as f:
        json.dump(results, f, indent=2)

    # Exit with appropriate code
    if results["overall_passed"]:
        print(f"✅ Evaluation PASSED ({results['overall_pass_rate']:.1%})")
        sys.exit(0)
    else:
        print(f"❌ Evaluation FAILED ({results['overall_pass_rate']:.1%})")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

---

## E.7 Production Monitoring

### E.7.1 Real-Time Monitoring Dashboard

```python
# monitoring/dashboard.py
from langfuse import Langfuse
import pandas as pd
from datetime import datetime, timedelta

class AgentMonitor:
    """Production monitoring for AI agents"""

    def __init__(self):
        self.langfuse = Langfuse()

    def get_metrics_summary(
        self,
        start_time: datetime = None,
        end_time: datetime = None
    ) -> dict:
        """Get summary metrics for a time period"""
        start_time = start_time or datetime.now() - timedelta(hours=24)
        end_time = end_time or datetime.now()

        # Fetch traces from Langfuse
        traces = self.langfuse.get_traces(
            start_time=start_time,
            end_time=end_time,
            limit=10000
        )

        if not traces:
            return {"error": "No traces found"}

        # Calculate metrics
        df = pd.DataFrame([
            {
                "trace_id": t.id,
                "duration_ms": t.duration,
                "status": t.status,
                "input_tokens": sum(g.input_tokens for g in t.generations if g.input_tokens),
                "output_tokens": sum(g.output_tokens for g in t.generations if g.output_tokens),
                "cost": sum(g.total_cost for g in t.generations if g.total_cost),
            }
            for t in traces
        ])

        return {
            "time_range": {
                "start": start_time.isoformat(),
                "end": end_time.isoformat()
            },
            "volume": {
                "total_traces": len(df),
                "traces_per_hour": len(df) / ((end_time - start_time).total_seconds() / 3600)
            },
            "latency": {
                "p50_ms": df["duration_ms"].quantile(0.5),
                "p95_ms": df["duration_ms"].quantile(0.95),
                "p99_ms": df["duration_ms"].quantile(0.99),
                "avg_ms": df["duration_ms"].mean()
            },
            "tokens": {
                "total_input": df["input_tokens"].sum(),
                "total_output": df["output_tokens"].sum(),
                "avg_per_request": (df["input_tokens"] + df["output_tokens"]).mean()
            },
            "cost": {
                "total": df["cost"].sum(),
                "avg_per_request": df["cost"].mean()
            },
            "errors": {
                "error_rate": (df["status"] == "ERROR").mean(),
                "error_count": (df["status"] == "ERROR").sum()
            }
        }

    def check_alerts(self, metrics: dict) -> list[dict]:
        """Check metrics against alert thresholds"""
        alerts = []

        # Latency alert
        if metrics["latency"]["p95_ms"] > 5000:
            alerts.append({
                "severity": "WARNING",
                "metric": "latency_p95",
                "message": f"P95 latency ({metrics['latency']['p95_ms']:.0f}ms) exceeds 5000ms threshold"
            })

        # Error rate alert
        if metrics["errors"]["error_rate"] > 0.05:
            alerts.append({
                "severity": "CRITICAL",
                "metric": "error_rate",
                "message": f"Error rate ({metrics['errors']['error_rate']:.1%}) exceeds 5% threshold"
            })

        # Cost alert
        if metrics["cost"]["avg_per_request"] > 0.10:
            alerts.append({
                "severity": "WARNING",
                "metric": "cost_per_request",
                "message": f"Avg cost (${metrics['cost']['avg_per_request']:.3f}) exceeds $0.10 threshold"
            })

        return alerts

# Example usage
if __name__ == "__main__":
    monitor = AgentMonitor()

    # Get last 24 hours metrics
    metrics = monitor.get_metrics_summary()

    print("=== Agent Metrics Summary ===")
    print(f"Total requests: {metrics['volume']['total_traces']}")
    print(f"P95 Latency: {metrics['latency']['p95_ms']:.0f}ms")
    print(f"Error Rate: {metrics['errors']['error_rate']:.2%}")
    print(f"Total Cost: ${metrics['cost']['total']:.2f}")

    # Check alerts
    alerts = monitor.check_alerts(metrics)
    if alerts:
        print("\n=== ALERTS ===")
        for alert in alerts:
            print(f"[{alert['severity']}] {alert['message']}")
```

---

## E.8 Custom Metric Implementation

### E.8.1 Creating a Custom DeepEval Metric

```python
# custom_metrics.py
from deepeval.metrics import BaseMetric
from deepeval.test_case import LLMTestCase
from typing import Optional
import re

class CustomPolicyComplianceMetric(BaseMetric):
    """
    Custom metric to check if agent responses comply with organizational policies.
    """

    def __init__(
        self,
        threshold: float = 0.9,
        policies: list[str] = None,
        model: str = "gpt-4o"
    ):
        self.threshold = threshold
        self.policies = policies or []
        self.model = model
        self.score = 0
        self.reason = ""
        self.success = False

    def measure(self, test_case: LLMTestCase) -> float:
        """Measure policy compliance"""
        violations = []

        for policy in self.policies:
            if not self._check_policy(test_case.actual_output, policy):
                violations.append(policy)

        # Calculate score
        if self.policies:
            self.score = 1 - (len(violations) / len(self.policies))
        else:
            self.score = 1.0

        self.success = self.score >= self.threshold

        if violations:
            self.reason = f"Violated policies: {', '.join(violations)}"
        else:
            self.reason = "All policies complied with"

        return self.score

    def _check_policy(self, response: str, policy: str) -> bool:
        """Check if response complies with a specific policy"""
        # Example policy checks - customize based on your needs

        if policy == "no_medical_advice":
            medical_terms = ["diagnose", "prescribe", "treatment plan"]
            return not any(term in response.lower() for term in medical_terms)

        elif policy == "no_financial_advice":
            financial_terms = ["invest in", "buy stock", "guaranteed returns"]
            return not any(term in response.lower() for term in financial_terms)

        elif policy == "disclaimer_required":
            return "disclaimer" in response.lower() or "please note" in response.lower()

        elif policy == "no_profanity":
            profanity_pattern = r'\b(damn|hell|crap)\b'
            return not re.search(profanity_pattern, response.lower())

        return True  # Unknown policy, assume compliant

    def is_successful(self) -> bool:
        return self.success

    @property
    def __name__(self):
        return "PolicyComplianceMetric"

class ResponseLengthMetric(BaseMetric):
    """Check if response length is within acceptable bounds"""

    def __init__(
        self,
        min_words: int = 10,
        max_words: int = 500,
        threshold: float = 1.0
    ):
        self.min_words = min_words
        self.max_words = max_words
        self.threshold = threshold
        self.score = 0
        self.reason = ""
        self.success = False

    def measure(self, test_case: LLMTestCase) -> float:
        word_count = len(test_case.actual_output.split())

        if self.min_words <= word_count <= self.max_words:
            self.score = 1.0
            self.reason = f"Word count ({word_count}) within acceptable range"
        elif word_count < self.min_words:
            self.score = word_count / self.min_words
            self.reason = f"Response too short ({word_count} words, min: {self.min_words})"
        else:
            self.score = self.max_words / word_count
            self.reason = f"Response too long ({word_count} words, max: {self.max_words})"

        self.success = self.score >= self.threshold
        return self.score

    def is_successful(self) -> bool:
        return self.success

    @property
    def __name__(self):
        return "ResponseLengthMetric"

# Example usage
if __name__ == "__main__":
    from deepeval import evaluate

    test_case = LLMTestCase(
        input="Should I invest in Bitcoin?",
        actual_output="While I can provide general information about Bitcoin, I cannot give specific investment advice. Please consult a licensed financial advisor for personalized recommendations."
    )

    policy_metric = CustomPolicyComplianceMetric(
        threshold=0.9,
        policies=["no_financial_advice", "disclaimer_required"]
    )

    length_metric = ResponseLengthMetric(min_words=10, max_words=100)

    results = evaluate([test_case], [policy_metric, length_metric])

    print(f"Policy Compliance: {policy_metric.score:.2f} - {policy_metric.reason}")
    print(f"Response Length: {length_metric.score:.2f} - {length_metric.reason}")
```

---

## References

1. DeepEval Documentation. (2025). *Custom Metrics Guide*
2. RAGAS Documentation. (2025). *RAG Evaluation Best Practices*
3. OpenTelemetry Documentation. (2025). *Python SDK Guide*
4. Langfuse Documentation. (2025). *Python Integration*

---

**Navigation:**
← [Previous Appendix: Judge Prompt Templates](appendix_d_judge_prompt_templates.md) | [Table of Contents](../../README.md) | [Next Appendix: Glossary of Terms](appendix_f_glossary.md) →
