# Section 13: Evaluation-Driven Development

> **Part VI**: Testing & Evaluation Processes
> **Estimated Reading Time**: 18 minutes

## Overview

Traditional software development treats testing as a validation checkpoint—build the feature, then verify it works. This paradigm fails catastrophically for AI agents, where behavior is emergent, non-deterministic, and evolves continuously in production. Evaluation-Driven Development (EDD) represents a fundamental paradigm shift: embedding continuous, adaptive evaluation across the entire agent lifecycle, from initial design through production operation. This section explores the EDD framework, CI/CD integration patterns, metrics-as-code practices, and iterative improvement workflows that enable reliable agent development.

## Table of Contents

- [13.1 The EDD Paradigm Shift](#131-the-edd-paradigm-shift)
- [13.2 CI/CD Integration](#132-cicd-integration)
- [13.3 Metrics as Code](#133-metrics-as-code)
- [13.4 IDE-Integrated Evaluation Tools](#134-ide-integrated-evaluation-tools)
- [13.5 Iterative Improvement Workflows](#135-iterative-improvement-workflows)

---

## 13.1 The EDD Paradigm Shift

Evaluation-Driven Development emerged from the recognition that traditional testing methodologies—Test-Driven Development (TDD) and Behavior-Driven Development (BDD)—were designed for deterministic systems with static requirements. AI agents violate every assumption these methodologies rely upon:

| Aspect | Traditional (TDD/BDD) | Evaluation-Driven Development |
|--------|----------------------|------------------------------|
| **Scope** | Pre-deployment only | Entire lifecycle (pre + post deployment) |
| **Requirements** | Static, explicitly defined | Evolving, context-dependent |
| **Feedback Sources** | Developers, stakeholders | Real-time monitoring, users, AI evaluators |
| **Adaptability** | Low to moderate | High (self-adjusting, continuous refinement) |
| **Success Definition** | Binary pass/fail | Multi-dimensional quality scores |
| **Test Data** | Fixed test sets | Dynamic, production-reflecting datasets |

### The EDD Manifesto

Research synthesizing 134 academic sources and 27 industry frameworks identified a persistent evaluation gap: **93.28%** of academic evaluations occur pre-deployment, while only **4.48%** implement continuous evaluation [1]. This pre-deployment bias creates the evaluation-to-production performance gap (91% eval → 68% production) that plagues agent deployments.

EDD addresses this through four core principles:

**1. Evaluation as First-Class Concern**

Evaluation isn't a phase—it's a continuous activity woven into every development decision. Teams define evaluation criteria before writing agent code, just as TDD advocates writing tests before implementation.

```
Traditional Development:
Design → Implement → Test → Deploy → (occasionally monitor)

Evaluation-Driven Development:
Define Eval Criteria → Design for Testability → Implement with Instrumentation →
Continuous Eval (offline) → Deploy with Monitoring → Continuous Eval (online) →
Feedback → Improve → Repeat
```

**2. Production Reality as Ground Truth**

Evaluation sets must reflect production distribution, not idealized scenarios. The Distribution Health Loop continuously monitors alignment between test data and real queries, triggering automatic refresh when distributions diverge [2].

**3. Multi-Dimensional Quality Assessment**

Single metrics (accuracy, F1) incentivize gaming. EDD mandates balanced evaluation across multiple dimensions: task completion, reasoning quality, safety, efficiency, and user experience. No dimension should be optimized at the expense of others.

**4. Closed Feedback Loops**

Production failures must flow back into development as actionable improvements. The forensic loop converts every production incident into a regression test, creating a continuously strengthening evaluation suite.

### The Four-Step EDD Process Model

Research at the University of Melbourne developed a validated process model for implementing EDD [1]:

```
┌─────────────────────────────────────────────────────────────────────┐
│  STEP 1: Define Evaluation Plan                                      │
│                                                                      │
│  • Translate user goals into testable scenarios                      │
│  • Incorporate governance requirements (compliance, ethics)          │
│  • Analyze agent architecture for evaluation points                  │
│  • Risk-based prioritization (severity × domain sensitivity)         │
│  • Generate structured, adaptive evaluation plans                    │
│                                                                      │
│  Outputs: Evaluation scope, strategy, criteria, metrics              │
├─────────────────────────────────────────────────────────────────────┤
│  STEP 2: Develop Evaluation Test Cases                               │
│                                                                      │
│  • Identify standardized benchmarks and frameworks                   │
│  • Collect domain-specific knowledge with expert guidance            │
│  • Curate test data collaboratively with SMEs                        │
│  • Generate synthetic data for rare/high-risk scenarios              │
│                                                                      │
│  Outputs: Comprehensive test suites (standard + edge cases)          │
├─────────────────────────────────────────────────────────────────────┤
│  STEP 3: Conduct Offline and Online Evaluations                      │
│                                                                      │
│  • Offline: Controlled environment testing, baseline establishment   │
│  • Online: Production monitoring, user interaction capture           │
│  • Evaluate results AND intermediate artifacts (traces, plans)       │
│  • Integrate AI evaluators with human oversight for high-stakes      │
│                                                                      │
│  Outputs: Offline baselines, production metrics, actionable feedback │
├─────────────────────────────────────────────────────────────────────┤
│  STEP 4: Analyze and Improve                                         │
│                                                                      │
│  • Runtime improvements: Dynamic pipeline adjustments                │
│  • Redevelopment: Architecture refinements, component updates        │
│  • Update safety cases and evaluation criteria                       │
│  • Evolve test sets based on production learnings                    │
│                                                                      │
│  Outputs: Updated safety cases, refined architecture, new tests      │
└─────────────────────────────────────────────────────────────────────┘
```

### Critical Gaps EDD Addresses

The multivocal literature review identified six systematic gaps in current agent evaluation practice [1]:

| Gap | Prevalence | EDD Solution |
|-----|------------|--------------|
| Fragmented lifecycle coverage | 93% pre-deployment only | Continuous offline + online evaluation |
| Over-reliance on aggregated metrics | 66% use only high-level metrics | Multi-dimensional, trace-level evaluation |
| Model-level vs system-level | 66% model-only, 22% system-level | Full-stack evaluation (model → agent → system) |
| Static evaluation | 98% static test cases | Adaptive, production-reflecting datasets |
| Evaluation as checkpoint | 71% treat eval as one-time | Continuous improvement feedback loops |
| AI-only evaluation | 88% academic, 56% industry lack humans | Hybrid AI + human evaluation (80/20 split) |

---

## 13.2 CI/CD Integration

Evaluation must be automated and integrated into the continuous integration/continuous deployment pipeline to catch regressions before they reach users and ensure consistent quality gates.

### 13.2.1 Pre-Deployment Testing

Every code or prompt change triggers automated evaluation before deployment eligibility.

**CI Pipeline Architecture:**

```yaml
# .github/workflows/agent-evaluation.yml
name: Agent Evaluation Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  evaluation:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run Unit Evaluations
        run: |
          python -m pytest tests/unit_evals/ \
            --eval-config=configs/eval_config.yaml \
            --output=results/unit_evals.json

      - name: Run Integration Evaluations
        run: |
          python -m agent_eval.integration \
            --golden-dataset=datasets/golden_v2.3.yaml \
            --metrics=task_success,safety,tool_accuracy \
            --output=results/integration_evals.json

      - name: Run Regression Tests
        run: |
          python -m agent_eval.regression \
            --baseline=baselines/production_v1.2.json \
            --threshold-file=thresholds/production.yaml \
            --output=results/regression.json

      - name: Evaluate with LLM Judge
        run: |
          python -m agent_eval.llm_judge \
            --samples=100 \
            --aspects=reasoning,helpfulness,safety \
            --judge-config=configs/judge_v2.yaml \
            --output=results/llm_judge.json

      - name: Generate Report
        run: |
          python -m agent_eval.report \
            --results-dir=results/ \
            --format=markdown \
            --output=evaluation_report.md

      - name: Post Results to PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const report = fs.readFileSync('evaluation_report.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: report
            });

      - name: Quality Gate Check
        run: |
          python -m agent_eval.gate \
            --results-dir=results/ \
            --thresholds=thresholds/production.yaml \
            --fail-on-regression
```

**Braintrust GitHub Action Integration:**

Braintrust provides the most comprehensive CI/CD integration with dedicated GitHub Actions [3]:

```yaml
- name: Run Braintrust Evaluation
  uses: braintrustdata/github-action@v1
  with:
    api-key: ${{ secrets.BRAINTRUST_API_KEY }}
    project: my-agent-project
    experiment-name: pr-${{ github.event.pull_request.number }}
    dataset: golden_dataset_v2
    eval-script: evals/comprehensive.py
```

The action automatically posts detailed comparisons to pull requests, showing:
- Score breakdowns by evaluation dimension
- Cases that improved or regressed
- Links to full experiment details in Braintrust dashboard

### 13.2.2 Regression Detection

Regression testing ensures new changes don't degrade existing capabilities.

**Regression Detection Framework:**

```python
class RegressionDetector:
    """
    Detect quality regressions between agent versions.
    """

    def __init__(self, baseline_results, threshold_config):
        self.baseline = baseline_results
        self.thresholds = threshold_config

    def detect_regressions(self, current_results):
        """
        Compare current results against baseline, flag regressions.
        """
        regressions = []
        improvements = []

        for metric, baseline_value in self.baseline.items():
            current_value = current_results.get(metric)
            if current_value is None:
                continue

            threshold = self.thresholds.get(metric, {})
            regression_threshold = threshold.get('regression_delta', 0.05)
            improvement_threshold = threshold.get('improvement_delta', 0.05)

            delta = current_value - baseline_value
            relative_delta = delta / baseline_value if baseline_value != 0 else delta

            if delta < -regression_threshold:
                regressions.append({
                    'metric': metric,
                    'baseline': baseline_value,
                    'current': current_value,
                    'delta': delta,
                    'relative_delta': relative_delta,
                    'severity': self.classify_severity(metric, relative_delta)
                })
            elif delta > improvement_threshold:
                improvements.append({
                    'metric': metric,
                    'baseline': baseline_value,
                    'current': current_value,
                    'delta': delta
                })

        return {
            'regressions': regressions,
            'improvements': improvements,
            'verdict': 'FAIL' if any(r['severity'] == 'critical' for r in regressions) else 'PASS',
            'requires_review': len(regressions) > 0
        }

    def classify_severity(self, metric, relative_delta):
        """Classify regression severity based on metric importance and magnitude."""
        critical_metrics = self.thresholds.get('critical_metrics', ['safety_score', 'task_success'])

        if metric in critical_metrics:
            if relative_delta < -0.10:
                return 'critical'
            elif relative_delta < -0.05:
                return 'high'
        else:
            if relative_delta < -0.20:
                return 'high'
            elif relative_delta < -0.10:
                return 'medium'

        return 'low'
```

**Threshold Configuration:**

```yaml
# thresholds/production.yaml
metrics:
  task_success_rate:
    baseline_minimum: 0.85
    regression_delta: 0.03
    improvement_delta: 0.02

  safety_score:
    baseline_minimum: 0.99
    regression_delta: 0.01  # Very tight threshold for safety
    improvement_delta: 0.005

  tool_accuracy:
    baseline_minimum: 0.92
    regression_delta: 0.05
    improvement_delta: 0.03

  response_latency_p95:
    baseline_maximum: 3000  # milliseconds
    regression_delta: 500   # Allow some variance
    improvement_delta: 200

  helpfulness_score:
    baseline_minimum: 4.0   # on 5-point scale
    regression_delta: 0.3
    improvement_delta: 0.2

critical_metrics:
  - safety_score
  - task_success_rate

blocking_rules:
  - metric: safety_score
    condition: "current < 0.95"
    action: block_deployment

  - metric: task_success_rate
    condition: "relative_regression > 0.10"
    action: require_manual_review
```

### 13.2.3 Performance Comparison

Compare candidate agent versions systematically using A/B experiment infrastructure.

**Experiment Design:**

```python
class AgentExperiment:
    """
    Run controlled experiments comparing agent versions.
    """

    def __init__(self, baseline_agent, candidate_agent, dataset):
        self.baseline = baseline_agent
        self.candidate = candidate_agent
        self.dataset = dataset

    def run_comparison(self, n_runs_per_case=3, statistical_threshold=0.05):
        """
        Run both agents on the same dataset, compute statistical comparison.
        """
        results = {
            'baseline': [],
            'candidate': []
        }

        for test_case in self.dataset:
            # Run multiple times to handle non-determinism
            for _ in range(n_runs_per_case):
                baseline_result = self.baseline.execute(test_case['input'])
                candidate_result = self.candidate.execute(test_case['input'])

                baseline_scores = self.evaluate(baseline_result, test_case)
                candidate_scores = self.evaluate(candidate_result, test_case)

                results['baseline'].append(baseline_scores)
                results['candidate'].append(candidate_scores)

        # Statistical analysis
        comparison = self.statistical_comparison(
            results['baseline'],
            results['candidate'],
            threshold=statistical_threshold
        )

        return comparison

    def statistical_comparison(self, baseline_scores, candidate_scores, threshold):
        """
        Perform statistical tests to determine significant differences.
        """
        from scipy import stats
        import numpy as np

        metrics = baseline_scores[0].keys()
        comparisons = {}

        for metric in metrics:
            baseline_values = [s[metric] for s in baseline_scores]
            candidate_values = [s[metric] for s in candidate_scores]

            # Paired t-test for normally distributed data
            t_stat, p_value = stats.ttest_rel(candidate_values, baseline_values)

            # Effect size (Cohen's d)
            diff = np.array(candidate_values) - np.array(baseline_values)
            effect_size = np.mean(diff) / np.std(diff)

            comparisons[metric] = {
                'baseline_mean': np.mean(baseline_values),
                'candidate_mean': np.mean(candidate_values),
                'difference': np.mean(candidate_values) - np.mean(baseline_values),
                'p_value': p_value,
                'significant': p_value < threshold,
                'effect_size': effect_size,
                'effect_interpretation': self.interpret_effect_size(effect_size),
                'recommendation': self.recommend_action(p_value, effect_size, metric)
            }

        return comparisons

    def interpret_effect_size(self, d):
        """Interpret Cohen's d effect size."""
        if abs(d) < 0.2:
            return 'negligible'
        elif abs(d) < 0.5:
            return 'small'
        elif abs(d) < 0.8:
            return 'medium'
        else:
            return 'large'
```

### 13.2.4 Pipeline Integration (Azure DevOps, GitHub Actions)

**Azure DevOps Pipeline:**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Evaluation
    displayName: 'Agent Evaluation'
    jobs:
      - job: RunEvaluations
        displayName: 'Run Evaluation Suite'
        steps:
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '3.11'

          - script: |
              pip install -r requirements.txt
              pip install azure-ai-evaluation
            displayName: 'Install dependencies'

          - script: |
              python -m agent_eval.run \
                --config=azure_eval_config.yaml \
                --output=$(Build.ArtifactStagingDirectory)/results
            displayName: 'Run evaluations'
            env:
              AZURE_AI_PROJECT: $(AZURE_AI_PROJECT)
              AZURE_CREDENTIAL: $(AZURE_CREDENTIAL)

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)/results'
              artifactName: 'EvaluationResults'

          - script: |
              python -m agent_eval.gate \
                --results=$(Build.ArtifactStagingDirectory)/results \
                --policy=policies/deployment_gate.yaml
            displayName: 'Quality Gate Check'

  - stage: Deploy
    displayName: 'Deploy Agent'
    dependsOn: Evaluation
    condition: succeeded()
    jobs:
      - deployment: DeployAgent
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploying agent..."
```

**Azure AI Foundry Evaluation SDK Integration:**

Azure provides native agent evaluation capabilities through the Azure AI Evaluation SDK [4]:

```python
from azure.ai.evaluation import evaluate, ToolCallAccuracyEvaluator, TaskAdherenceEvaluator

# Configure evaluators
evaluators = {
    "tool_accuracy": ToolCallAccuracyEvaluator(),
    "task_adherence": TaskAdherenceEvaluator(),
    "intent_resolution": IntentResolutionEvaluator(),
}

# Run evaluation
results = evaluate(
    data="datasets/golden_dataset.jsonl",
    evaluators=evaluators,
    azure_ai_project={
        "subscription_id": os.environ["AZURE_SUBSCRIPTION_ID"],
        "resource_group": os.environ["AZURE_RESOURCE_GROUP"],
        "project_name": os.environ["AZURE_PROJECT_NAME"],
    },
)

# Check results against thresholds
for metric, value in results.metrics.items():
    threshold = thresholds.get(metric)
    if threshold and value < threshold:
        raise Exception(f"Metric {metric} ({value}) below threshold ({threshold})")
```

---

## 13.3 Metrics as Code

Treating evaluation metrics as version-controlled code ensures reproducibility, enables collaboration, and creates an audit trail for quality decisions.

### 13.3.1 Version Control for Evaluations

Store evaluation configurations, prompts, and rubrics alongside agent code.

**Repository Structure:**

```
agent-project/
├── src/
│   └── agent/              # Agent implementation
├── evals/
│   ├── configs/
│   │   ├── production.yaml
│   │   ├── staging.yaml
│   │   └── development.yaml
│   ├── metrics/
│   │   ├── task_success.py
│   │   ├── tool_accuracy.py
│   │   ├── safety.py
│   │   └── custom_domain.py
│   ├── prompts/
│   │   ├── judge_prompts/
│   │   │   ├── reasoning_v2.txt
│   │   │   ├── helpfulness_v3.txt
│   │   │   └── safety_v1.txt
│   │   └── rubrics/
│   │       ├── helpfulness_rubric.md
│   │       └── safety_rubric.md
│   ├── datasets/
│   │   ├── golden_v2.3.yaml
│   │   ├── regression_v1.yaml
│   │   └── adversarial_v1.yaml
│   └── baselines/
│       ├── production_v1.2.json
│       └── baseline_history.json
├── tests/
│   └── unit_evals/         # Fast unit-level evaluations
└── .github/
    └── workflows/
        └── evaluation.yml
```

**Benefits:**
- **Reproducibility:** Any historical evaluation can be re-run exactly
- **Code Review:** Metric changes go through PR review
- **Audit Trail:** Git history shows who changed what and why
- **Branching:** Experiment with new metrics on branches
- **Rollback:** Revert problematic metric changes easily

### 13.3.2 Centralized Metric Libraries

Build reusable metric libraries that encapsulate evaluation logic and can be shared across projects.

**Metric Library Architecture:**

```python
# evals/metrics/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Any, Dict, Optional

@dataclass
class MetricResult:
    """Standardized metric result structure."""
    metric_name: str
    score: float
    confidence: Optional[float] = None
    reasoning: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None

class BaseMetric(ABC):
    """Base class for all evaluation metrics."""

    def __init__(self, config: Optional[Dict] = None):
        self.config = config or {}
        self.name = self.__class__.__name__

    @abstractmethod
    def evaluate(self, agent_output: Dict, reference: Optional[Dict] = None) -> MetricResult:
        """
        Evaluate agent output and return standardized result.

        Args:
            agent_output: The agent's response and trace
            reference: Optional reference/expected output

        Returns:
            MetricResult with score and metadata
        """
        pass

    def validate_input(self, agent_output: Dict) -> bool:
        """Validate that required fields are present."""
        required_fields = self.config.get('required_fields', [])
        return all(field in agent_output for field in required_fields)


# evals/metrics/task_success.py
class TaskSuccessMetric(BaseMetric):
    """
    Evaluate whether the agent successfully completed the user's task.
    """

    def __init__(self, config: Optional[Dict] = None):
        super().__init__(config)
        self.success_criteria = self.config.get('success_criteria', 'output_match')

    def evaluate(self, agent_output: Dict, reference: Optional[Dict] = None) -> MetricResult:
        if self.success_criteria == 'output_match':
            score = self._evaluate_output_match(agent_output, reference)
        elif self.success_criteria == 'tool_sequence':
            score = self._evaluate_tool_sequence(agent_output, reference)
        elif self.success_criteria == 'llm_judge':
            score = self._evaluate_with_llm_judge(agent_output, reference)
        else:
            raise ValueError(f"Unknown success criteria: {self.success_criteria}")

        return MetricResult(
            metric_name=self.name,
            score=score,
            reasoning=f"Evaluated using {self.success_criteria} criteria"
        )

    def _evaluate_output_match(self, agent_output, reference):
        # Implementation for output matching
        pass

    def _evaluate_tool_sequence(self, agent_output, reference):
        # Implementation for tool sequence checking
        pass

    def _evaluate_with_llm_judge(self, agent_output, reference):
        # Implementation using LLM judge
        pass


# evals/metrics/registry.py
class MetricRegistry:
    """Central registry for all available metrics."""

    _metrics: Dict[str, type] = {}

    @classmethod
    def register(cls, name: str):
        """Decorator to register a metric class."""
        def decorator(metric_class):
            cls._metrics[name] = metric_class
            return metric_class
        return decorator

    @classmethod
    def get(cls, name: str, config: Optional[Dict] = None) -> BaseMetric:
        """Get an instance of a registered metric."""
        if name not in cls._metrics:
            raise ValueError(f"Unknown metric: {name}. Available: {list(cls._metrics.keys())}")
        return cls._metrics[name](config)

    @classmethod
    def list_available(cls) -> List[str]:
        """List all registered metrics."""
        return list(cls._metrics.keys())


# Usage
@MetricRegistry.register('task_success')
class TaskSuccessMetric(BaseMetric):
    ...

@MetricRegistry.register('tool_accuracy')
class ToolAccuracyMetric(BaseMetric):
    ...
```

### 13.3.3 Evaluation Pipelines

Compose metrics into configurable evaluation pipelines that can be run consistently.

**Pipeline Configuration:**

```yaml
# evals/configs/production.yaml
pipeline:
  name: production_evaluation
  version: "2.3.0"

stages:
  - name: fast_checks
    parallel: true
    metrics:
      - name: format_compliance
        config:
          schema: schemas/output_schema.json
      - name: latency
        config:
          threshold_ms: 3000

  - name: quality_evaluation
    parallel: true
    metrics:
      - name: task_success
        config:
          success_criteria: llm_judge
          judge_prompt: prompts/judge_prompts/task_success_v2.txt
          weight: 0.3
      - name: tool_accuracy
        config:
          strict_parameter_check: true
          weight: 0.25
      - name: reasoning_quality
        config:
          judge_prompt: prompts/judge_prompts/reasoning_v2.txt
          weight: 0.2
      - name: helpfulness
        config:
          rubric: prompts/rubrics/helpfulness_rubric.md
          weight: 0.25

  - name: safety_checks
    parallel: false  # Safety checks run sequentially for thoroughness
    fail_fast: true  # Stop pipeline on safety failure
    metrics:
      - name: safety_compliance
        config:
          policies: policies/safety_v1.yaml
          threshold: 0.99
      - name: pii_detection
        config:
          patterns: patterns/pii_patterns.yaml
          threshold: 1.0  # Zero tolerance

aggregation:
  method: weighted_average
  weights_from: metric_configs
  minimum_passing_score: 0.75

reporting:
  format: [json, markdown]
  include_trace_samples: true
  sample_size: 10
```

**Pipeline Runner:**

```python
class EvaluationPipeline:
    """
    Execute configured evaluation pipelines.
    """

    def __init__(self, config_path: str):
        self.config = self.load_config(config_path)
        self.metrics = self.initialize_metrics()

    def run(self, dataset: List[Dict], parallel: bool = True) -> Dict:
        """
        Run the full evaluation pipeline on a dataset.
        """
        results = {
            'pipeline_name': self.config['pipeline']['name'],
            'pipeline_version': self.config['pipeline']['version'],
            'timestamp': datetime.now().isoformat(),
            'dataset_size': len(dataset),
            'stage_results': {},
            'aggregate_score': None
        }

        for stage in self.config['stages']:
            stage_name = stage['name']
            print(f"Running stage: {stage_name}")

            stage_results = self.run_stage(stage, dataset)
            results['stage_results'][stage_name] = stage_results

            # Check for fail_fast
            if stage.get('fail_fast') and not stage_results['passed']:
                results['early_termination'] = {
                    'stage': stage_name,
                    'reason': 'fail_fast triggered'
                }
                break

        # Calculate aggregate
        results['aggregate_score'] = self.aggregate_results(results['stage_results'])
        results['verdict'] = 'PASS' if results['aggregate_score'] >= self.config['aggregation']['minimum_passing_score'] else 'FAIL'

        return results

    def run_stage(self, stage_config: Dict, dataset: List[Dict]) -> Dict:
        """Run a single evaluation stage."""
        metrics_to_run = stage_config['metrics']

        if stage_config.get('parallel', False):
            # Run metrics in parallel
            with ThreadPoolExecutor(max_workers=len(metrics_to_run)) as executor:
                futures = {
                    executor.submit(self.run_metric, m, dataset): m['name']
                    for m in metrics_to_run
                }
                metric_results = {
                    futures[f]: f.result() for f in as_completed(futures)
                }
        else:
            # Run sequentially
            metric_results = {}
            for metric_config in metrics_to_run:
                result = self.run_metric(metric_config, dataset)
                metric_results[metric_config['name']] = result

                # Check for early exit on failure
                if stage_config.get('fail_fast') and result['score'] < metric_config.get('threshold', 0):
                    break

        return {
            'metrics': metric_results,
            'passed': all(r['passed'] for r in metric_results.values())
        }
```

---

## 13.4 IDE-Integrated Evaluation Tools

Modern development workflows benefit from evaluation tools integrated directly into the development environment, enabling rapid iteration without context switching.

### IDE Extensions

**VS Code Extension Features:**

Azure AI Foundry provides a VS Code extension for agent evaluation [4]:

```
┌─────────────────────────────────────────────────────────────────┐
│  VS Code Agent Evaluation Features                               │
├─────────────────────────────────────────────────────────────────┤
│  • Run evaluations directly from editor                         │
│  • View results in side panel                                    │
│  • Quick-compare against baselines                               │
│  • Trace visualization                                           │
│  • Inline metric annotations                                     │
│  • One-click test case generation from failures                  │
└─────────────────────────────────────────────────────────────────┘
```

**Galileo IDE Integration:**

Galileo AI provides IDE-integrated tools that enable developers to [5]:
- Generate test data directly from the IDE
- Run evaluations on code changes before committing
- Define custom domain-specific metrics in natural language
- View evaluation results inline with code

### Notebook-Based Evaluation

For exploratory development, Jupyter notebooks provide an interactive evaluation environment:

```python
# evaluation_notebook.ipynb

# Cell 1: Setup
from agent_eval import EvaluationPipeline, visualize_results
import pandas as pd

pipeline = EvaluationPipeline('configs/development.yaml')
dataset = load_dataset('datasets/dev_sample.yaml')

# Cell 2: Run evaluation
results = pipeline.run(dataset)

# Cell 3: Visualize results
visualize_results(results)

# Cell 4: Drill into failures
failures = [r for r in results['detailed'] if not r['passed']]
for failure in failures[:5]:
    display_trace(failure['trace'])
    display_evaluation(failure['evaluation'])

# Cell 5: Generate new test cases from failures
new_tests = generate_test_cases_from_failures(failures)
save_to_dataset(new_tests, 'datasets/new_regression_tests.yaml')
```

---

## 13.5 Iterative Improvement Workflows

EDD is fundamentally iterative—each evaluation cycle informs the next improvement. Establishing structured workflows ensures continuous quality improvement.

### The Forensic Loop

The forensic loop converts production failures into development improvements:

```
┌─────────────────────────────────────────────────────────────────┐
│  PRODUCTION                                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Agent serving users, full trace logging enabled          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Anomaly/failure detected (metric drop, user complaint,   │    │
│  │ safety violation, error spike)                           │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                      │
├───────────────────────────┼──────────────────────────────────────┤
│  ANALYSIS                 ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Pull full trace, analyze root cause:                     │    │
│  │ - Where did reasoning go wrong?                          │    │
│  │ - Was it a tool error?                                   │    │
│  │ - Policy gap?                                            │    │
│  │ - Edge case in prompt?                                   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                      │
├───────────────────────────┼──────────────────────────────────────┤
│  TEST GENERATION          ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Convert failure to regression test:                      │    │
│  │ - Exact input/context preserved                          │    │
│  │ - Expected behavior defined (opposite of failure)        │    │
│  │ - Added to regression suite                              │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                      │
├───────────────────────────┼──────────────────────────────────────┤
│  FIX & VERIFY             ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Implement fix (prompt, code, policy)                     │    │
│  │ Run regression suite (must pass new test)                │    │
│  │ Run full evaluation (no regressions elsewhere)           │    │
│  │ Deploy via CI/CD pipeline                                │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│                    Back to Production                            │
└─────────────────────────────────────────────────────────────────┘
```

### Feedback Integration Architecture

**MLflow Feedback Tracking:**

MLflow provides native support for tracking human feedback, enabling closed-loop improvement [6]:

```python
import mlflow

# Log agent execution with trace
with mlflow.start_run():
    # Execute agent
    result = agent.execute(user_query)

    # Log trace
    mlflow.log_trace(result.trace)

    # Later: collect feedback
    mlflow.log_feedback(
        run_id=mlflow.active_run().info.run_id,
        feedback={
            'human_rating': 4,
            'feedback_text': 'Good response but could be more concise',
            'feedback_type': 'quality_review',
            'reviewer_id': 'expert-1'
        }
    )

# Query feedback for improvement analysis
feedback_data = mlflow.search_feedback(
    filter_string="feedback_type = 'quality_review' AND human_rating < 3",
    order_by=["timestamp DESC"]
)

# Use feedback to generate improvement insights
low_rated_traces = [get_trace(f.run_id) for f in feedback_data]
improvement_suggestions = analyze_failure_patterns(low_rated_traces)
```

### Continuous Improvement Cycles

**Weekly Improvement Rhythm:**

```yaml
improvement_cycle:
  cadence: weekly

  monday:
    - Review production metrics dashboard
    - Identify top 3 failure patterns from past week
    - Prioritize based on user impact

  tuesday_wednesday:
    - Root cause analysis on prioritized failures
    - Generate test cases for each failure pattern
    - Propose fixes (prompt changes, code updates)

  thursday:
    - Run candidate fixes through evaluation pipeline
    - Compare against baseline
    - Code review for approved changes

  friday:
    - Deploy validated improvements
    - Update baseline metrics
    - Document learnings
    - Plan next week's priorities

  metrics_to_track:
    - Failure rate trend (should decrease)
    - Regression test suite size (should grow)
    - Time from failure detection to fix deployment
    - Test coverage by intent category
```

### DevOps Paradigm Shift (2026 Perspective)

The 2026 landscape is witnessing a fundamental shift from automation to autonomy in DevOps [7]:

```
Traditional DevOps (Automation):
"Do what I say" - Human-defined pipelines execute predetermined steps

EDD-Enabled DevOps (2026):
"Decide what's best" - Evaluation-informed systems make deployment decisions

Emerging Patterns:
├── AI agents write code, generate tests, remediate incidents
├── Continuous experimentation seamlessly integrated
├── Evaluation metrics drive automatic rollback decisions
├── Progressive delivery based on real-time quality signals
└── Adaptive scaling informed by evaluation outcomes
```

This shift raises important questions that EDD helps answer:
- What does "velocity" mean when machines do the work? **Answer:** Measure by evaluation improvement rate, not deployment frequency
- How do we maintain reliability with learning systems? **Answer:** Continuous evaluation with automatic rollback triggers
- How do we ensure accountability? **Answer:** Audit trails through metrics-as-code and evaluation logging

---

## Key Takeaways

1. **EDD is a paradigm shift, not a process tweak**: Moving from evaluation-as-checkpoint to evaluation-as-continuous-activity requires cultural and tooling changes across the organization

2. **93% of academic evaluations are pre-deployment only**: This creates the evaluation-production gap that EDD addresses through continuous offline + online evaluation

3. **CI/CD integration is non-negotiable**: Every code/prompt change must trigger automated evaluation with regression detection and quality gates before deployment eligibility

4. **Metrics-as-code enables reproducibility and collaboration**: Version-control evaluation configurations, prompts, and rubrics alongside agent code for audit trails and team coordination

5. **The forensic loop closes the improvement cycle**: Every production failure should automatically generate a regression test, creating a continuously strengthening evaluation suite

6. **Hybrid evaluation (80/20 AI/human) is the industry standard**: Pure automation misses nuance; pure human review doesn't scale—the combination achieves both quality and coverage

7. **The DevOps paradigm is shifting from automation to autonomy**: Evaluation metrics increasingly drive deployment decisions, rollbacks, and scaling—not just inform human reviewers

---

## References

1. Akram, B. et al. (2024). *Evaluation-Driven Development of LLM Agents: A Process Model and Reference Architecture*. arXiv. [https://arxiv.org/html/2411.13768v2](https://arxiv.org/html/2411.13768v2)

2. Bhatia, N. (2025). *Part 2 — Solving the Distribution Mismatch: Make Your Eval Actually Predict Production*. Medium. [https://medium.com/@nitikab23/part-2-evaluating-agentic-ai-systems-the-complete-framework-55ceaf86c406](https://medium.com/@nitikab23/part-2-evaluating-agentic-ai-systems-the-complete-framework-55ceaf86c406)

3. Braintrust. (2025). *Best AI Evals Tools for CI/CD in 2025*. [https://www.braintrust.dev/articles/best-ai-evals-tools-cicd-2025](https://www.braintrust.dev/articles/best-ai-evals-tools-cicd-2025)

4. Microsoft. (2026). *Agent Evaluation with Azure AI Foundry SDK*. Azure Documentation. [https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/agent-evaluate-sdk](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/agent-evaluate-sdk)

5. Galileo AI. (2025). *Four New Agent Evaluation Metrics*. [https://galileo.ai/blog/four-new-agent-evaluation-metrics](https://galileo.ai/blog/four-new-agent-evaluation-metrics)

6. MLflow. (2025). *MLflow Tracing for LLM Observability*. [https://mlflow.org/docs/latest/genai/tracing/](https://mlflow.org/docs/latest/genai/tracing/)

7. DevOps.com. (2026). *Predict 2026: Why AI Will Force DevOps to Reinvent Itself*. [https://devops.com/predict-2026-why-ai-will-force-devops-to-reinvent-itself/](https://devops.com/predict-2026-why-ai-will-force-devops-to-reinvent-itself/)

8. DEV Community. (2025). *A Practical Guide to Integrating AI Evals into Your CI/CD Pipeline*. [https://dev.to/kuldeep_paul/a-practical-guide-to-integrating-ai-evals-into-your-cicd-pipeline-3mlb](https://dev.to/kuldeep_paul/a-practical-guide-to-integrating-ai-evals-into-your-cicd-pipeline-3mlb)

9. Master of Code. (2025). *AI Agent Evaluation Metrics 2025*. [https://masterofcode.com/blog/ai-agent-evaluation](https://masterofcode.com/blog/ai-agent-evaluation)

10. LangChain. (2025). *State of AI Agents*. [https://www.langchain.com/state-of-agent-engineering](https://www.langchain.com/state-of-agent-engineering)

11. ODSC. (2025). *Mastering AI Evaluation in 2025: Lessons from Ian Cairns*. [https://opendatascience.com/mastering-ai-evaluation-in-2025-lessons-from-ian-cairns-on-building-reliable-systems/](https://opendatascience.com/mastering-ai-evaluation-in-2025-lessons-from-ian-cairns-on-building-reliable-systems/)

12. testRigor. (2025). *Different Evals for Agentic AI: Methods, Metrics & Best Practices*. [https://testrigor.com/blog/different-evals-for-agentic-ai/](https://testrigor.com/blog/different-evals-for-agentic-ai/)

---

**Navigation:**
← [Previous Section: LLM-as-a-Judge Evaluation](12_llm_as_a_judge_evaluation.md) | [Table of Contents](../README.md) | [Next Section: Observability and Tracing Platforms](14_observability_and_tracing_platforms.md) →
