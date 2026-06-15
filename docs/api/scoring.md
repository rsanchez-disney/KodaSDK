# Scoring API

Evaluate prompt quality before sending to an agent. Returns a composite score with dimensional breakdown.

## `koda.score(prompt)`

```typescript
async score(prompt: string): Promise<ScoreResult>
```

## ScoreResult

```typescript
interface ScoreResult {
  total: number;              // 0-100 composite score
  estimatedTokens: number;
  dimensions: Record<string, { score: number; reasons: string[] }>;
}
```

## Usage

```typescript
const result = await koda.score('Explain the velocity trend for team Bolt');

console.log(result.total);           // 78
console.log(result.estimatedTokens); // 1200

for (const [dim, info] of Object.entries(result.dimensions)) {
  console.log(`${dim}: ${info.score} — ${info.reasons.join(', ')}`);
}
```

## Configuration

The scoring endpoint defaults to `http://localhost:8000/score`. Override via SDK options:

```typescript
const koda = new KodaSDK({
  appName: 'my-app',
  scoringUrl: 'http://scoring.internal/score',
});
```

!!! note
    Scoring is a local evaluation that doesn't consume agent tokens. Use it to pre-validate prompts or provide quality feedback to users before submission.
