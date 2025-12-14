# Evaluator Patterns

Code examples for creating custom evaluators in Go and TypeScript.

## Heuristic Evaluator (Go)

Non-billed, deterministic evaluation:

```go
func NewLengthEvaluator(g *genkit.Genkit) ai.Evaluator {
    return genkit.DefineEvaluator(g, api.NewName("custom", "lengthCheck"),
        &ai.EvaluatorOptions{
            DisplayName: "Response Length",
            Definition:  "Validates response meets minimum length.",
            IsBilled:    false,
        },
        func(ctx context.Context, req *ai.EvaluatorCallbackRequest) (*ai.EvaluatorCallbackResponse, error) {
            // Validate required fields
            if req.Input.Output == nil {
                return nil, errors.New("output is required")
            }
            output, ok := req.Input.Output.(string)
            if !ok {
                return nil, errors.New("output must be a string")
            }

            // Scoring logic
            score := 1.0
            if len(output) < 50 {
                score = 0.0
            }

            return &ai.EvaluatorCallbackResponse{
                TestCaseId: req.Input.TestCaseId,
                Evaluation: []ai.Score{{Score: score}},
            }, nil
        },
    )
}
```

## LLM-Based Evaluator (Go)

Billed evaluator using LLM as judge:

```go
func NewSemanticEvaluator(g *genkit.Genkit) ai.Evaluator {
    return genkit.DefineEvaluator(g, api.NewName("custom", "semanticQuality"),
        &ai.EvaluatorOptions{
            DisplayName: "Semantic Quality",
            Definition:  "Assesses response quality using LLM judgment.",
            IsBilled:    true,  // Important: triggers cost confirmation
        },
        func(ctx context.Context, req *ai.EvaluatorCallbackRequest) (*ai.EvaluatorCallbackResponse, error) {
            if req.Input.Output == nil {
                return nil, errors.New("output is required")
            }
            output := req.Input.Output.(string)
            input := req.Input.Input.(string)

            resp, err := genkit.Generate(ctx, g,
                ai.WithModelName("googleai/gemini-2.0-flash"),
                ai.WithPrompt(fmt.Sprintf(
                    `Rate this response 0-1 for quality.
                    Question: %s
                    Response: %s
                    Return only a number.`, input, output)),
            )
            if err != nil {
                return nil, fmt.Errorf("evaluation failed: %w", err)
            }

            score, _ := strconv.ParseFloat(strings.TrimSpace(resp.Text()), 64)
            return &ai.EvaluatorCallbackResponse{
                TestCaseId: req.Input.TestCaseId,
                Evaluation: []ai.Score{{Score: score}},
            }, nil
        },
    )
}
```

## Heuristic Evaluator (TypeScript)

```typescript
export const jsonStructureEvaluator = ai.defineEvaluator(
    {
        name: 'custom/jsonStructure',
        displayName: 'JSON Structure',
        definition: 'Validates output contains required JSON fields.',
    },
    async (datapoint: BaseEvalDataPoint) => {
        if (!datapoint.output) {
            throw new Error('output is required');
        }

        let score = 0;
        try {
            const parsed = JSON.parse(datapoint.output as string);
            const requiredFields = ['id', 'name', 'status'];
            const hasAll = requiredFields.every(f => f in parsed);
            score = hasAll ? 1 : 0;
        } catch {
            score = 0;
        }

        return {
            testCaseId: datapoint.testCaseId,
            evaluation: { score },
        };
    }
);
```

## LLM-Based Evaluator (TypeScript)

```typescript
export const faithfulnessEvaluator = ai.defineEvaluator(
    {
        name: 'custom/faithfulness',
        displayName: 'Faithfulness',
        definition: 'Checks if response is faithful to provided context.',
        isBilled: true,
    },
    async (datapoint: BaseEvalDataPoint) => {
        if (!datapoint.output || !datapoint.context) {
            throw new Error('output and context are required');
        }

        const result = await ai.generate({
            model: 'googleai/gemini-2.0-flash',
            prompt: `Rate 0-1 how faithful this response is to the context.
                Context: ${JSON.stringify(datapoint.context)}
                Response: ${datapoint.output}
                Return only a number.`,
        });

        return {
            testCaseId: datapoint.testCaseId,
            evaluation: { score: parseFloat(result.text()) },
        };
    }
);
```

## Initializing Evaluators

### Go

```go
import "github.com/firebase/genkit/go/plugins/evaluators"

metrics := []evaluators.MetricConfig{
    {MetricType: evaluators.EvaluatorDeepEqual},
    {MetricType: evaluators.EvaluatorRegex},
}

g := genkit.Init(ctx,
    genkit.WithPlugins(&evaluators.GenkitEval{Metrics: metrics}),
)

// Register custom evaluators
NewLengthEvaluator(g)
NewSemanticEvaluator(g)
```

### TypeScript

```typescript
import { genkitEval, GenkitMetric } from '@genkit-ai/evaluator';

export const ai = genkit({
    plugins: [
        genkitEval({
            judge: googleAI.model('gemini-2.0-flash'),
            metrics: [GenkitMetric.FAITHFULNESS, GenkitMetric.MALICIOUSNESS],
        }),
    ],
});
```

## Built-in Evaluators

| Evaluator | Description |
|-----------|-------------|
| `EvaluatorDeepEqual` | Exact match against reference |
| `EvaluatorRegex` | Pattern match against reference regex |
| `EvaluatorJsonata` | JSONata expression match |
| `GenkitMetric.FAITHFULNESS` | Factual consistency with context |
| `GenkitMetric.ANSWER_RELEVANCY` | Response relevance to prompt |
| `GenkitMetric.MALICIOUSNESS` | Harmful content detection |
