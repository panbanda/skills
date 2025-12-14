---
name: ai-evaluations
description: Use when implementing quality assessment for LLM/AI outputs, creating evaluators, comparing model performance, or setting up automated testing for generated content - provides evaluator patterns, dataset management, and CI/CD integration guidance
---

# AI Evaluations

Systematic quality assessment for LLM applications using automated evaluators, datasets, and metrics.

## Evaluation Types

**Inference-based**: Run inputs through your flow, assess outputs. Use for most testing scenarios.

**Raw evaluation**: Assess pre-collected data without inference. Use when you have production traces or external data.

## Evaluator Selection

| Need | Evaluator Type | Cost |
|------|---------------|------|
| Format validation, length checks, regex matching | Heuristic | Free |
| JSON structure, field presence | Heuristic | Free |
| Semantic quality, tone, accuracy | LLM-based | Billed |
| Maliciousness, harmful content | LLM-based | Billed |
| Faithfulness to retrieved context | LLM-based | Billed |

**Start with heuristic evaluators.** Only use LLM-based when semantic judgment is required.

## What to Test

### Test Case Categories

Build datasets covering three categories:

1. **Happy path** - Common, expected inputs reflecting typical usage
2. **Edge cases** - Ambiguous, complex, or unusual inputs
3. **Adversarial** - Malicious inputs testing safety (prompt injection, jailbreaks)

### Quality Dimensions

| Dimension | What It Measures | Method |
|-----------|------------------|--------|
| **Correctness** | Factual accuracy against ground truth | Reference-based comparison |
| **Hallucination** | Fabricated or unsupported claims | LLM-as-judge or NLI models |
| **Relevance** | Response addresses the input appropriately | LLM-as-judge |
| **Faithfulness** | Output aligns with retrieved context (RAG) | LLM-as-judge |
| **Completeness** | Fully answers the question | LLM-as-judge or checklist |
| **Toxicity** | Offensive or harmful content | LLM-as-judge or classifier |
| **Bias** | Unfair treatment across demographics | LLM-as-judge |
| **Format** | Correct structure (JSON, required fields) | Heuristic validation |

### Prioritization

1. Start with metrics matching observed failures
2. Add safety metrics for user-facing applications
3. Include faithfulness for RAG systems
4. Expand coverage gradually based on real issues

### Dataset Design

- **Use real data** - Actual user queries, support logs, production traces
- **Ensure diversity** - Varying complexity, topics, input lengths
- **Include failures** - Cases where the system previously failed
- **Keep stable** - Don't change datasets mid-experiment
- **Start small** - 20-50 cases initially, expand based on findings

## Dataset Requirements

```json
{
  "testCaseId": "unique-id",
  "input": "test input (required)",
  "output": "actual output (optional for inference-based)",
  "reference": "expected result (optional)",
  "context": ["retrieved context (optional)"]
}
```

**Critical**: Evaluations being compared MUST use the same dataset. File-source evaluations cannot be compared.

## Workflow

1. **Create dataset** - Dev UI or JSON file with diverse test cases including edge cases
2. **Select evaluators** - Start heuristic, add LLM-based as needed
3. **Run evaluation** - `genkit eval:flow <flow> --input <dataset>`
4. **Review results** - Dev UI at `localhost:4000/evaluate`
5. **Compare runs** - Use same dataset, check metric highlighting (green=improvement, red=regression)

## CLI Commands

```bash
# Inference-based evaluation
genkit eval:flow myFlow --input dataset.json

# Run specific evaluators
genkit eval:flow myFlow --input dataset.json --evaluators=custom/myEval

# Raw evaluation (no inference)
genkit eval:run dataset.json

# Extract data from traces for evaluation
genkit eval:extractData myFlow --label myRun --output evalData.json

# Batch processing for large datasets
genkit eval:flow myFlow --input dataset.json --batchSize=10
```

## Creating Evaluators

For code patterns and examples, see [references/evaluator-patterns.md](references/evaluator-patterns.md).

**Key points:**
- Use namespace format: `custom/evaluatorName`
- Set `IsBilled: true` for LLM-based evaluators (UI will prompt confirmation)
- Validate input fields exist before processing
- Return scores as 0.0-1.0 for comparison compatibility

## Best Practices

1. **Cost awareness** - Monitor billed evaluator usage; start with heuristics
2. **Dataset diversity** - Cover typical inputs, edge cases, and failure modes
3. **Version control** - Commit datasets and evaluation results
4. **CI/CD integration** - Run evaluations on each commit for regression detection
5. **Baseline first** - Establish baseline evaluation before making changes
6. **Same dataset rule** - Always use identical datasets when comparing runs
