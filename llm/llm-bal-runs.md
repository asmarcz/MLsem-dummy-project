# LLM inference of `bal` (llm.mlsem), 2026-11-11

Native Anthropic API, each model at its default effort, from the "MLsem" output
channel (18:57–19:01). The file's own typecheck timed out (10 s), which the
prompt told the model.

## Runs

| Model | Tries | Result | Wall time | Prompt tokens | Cache read | Cache write | Uncached | Completion tokens | Thinking | Est. cost |
|---|---|---|---|---|---|---|---|---|---|---|
| claude-haiku-5-5 | 2 | checks | 8.3 s | 3,864 | 1,775 | 2,081 | 8 | 620 | 560 | ≈ $0.0006 |
| claude-sonnet-5-5 | 1 | checks | 2.7 s | 1,779 | 0 | 1,775 | 4 | 31 | 0 | ≈ $0.0048 |
| claude-opus-5-5 | 1 | checks | 3.1 s | 1,779 | 0 | 1,775 | 4 | 31 | 0 | ≈ $0.0095 |

All three ended with `val bal : t('a) -> Key -> 'a -> t('a) -> t('a)`.

## Tries

| Model | Try | Answer | Result | LLM time | Check time | Completion | Thinking |
|---|---|---|---|---|---|---|---|
| claude-haiku-5-5 | 1 | `val bal : (t('a), Key, 'a, t('a)) -> t('a)` | rejected: untypeable cast at `l` (expected `empty`, given the tuple) | 3.0 s | 0.07 s | 241 | 212 |
| claude-haiku-5-5 | 2 | `val bal : t('a) -> Key -> 'a -> t('a) -> t('a)` | checks | 3.8 s | 1.4 s | 379 | 348 |
| claude-sonnet-5-5 | 1 | `val bal : t('a) -> Key -> 'a -> t('a) -> t('a)` | checks | 1.3 s | 1.4 s | 31 | 0 |
| claude-opus-5-5 | 1 | `val bal : t('a) -> Key -> 'a -> t('a) -> t('a)` | checks | 1.7 s | 1.4 s | 31 | 0 |

## Notes

- **Cost.** Estimated from the prices in Claude's API reference cached on
  2026-10-06 (per million tokens, input / output: Haiku 5.5 $0.10 / $0.50,
  Sonnet 5.5 $2 / $10, Opus 5.5 $4 / $20), with cache writes at 1.25× and cache
  reads at 0.1× the input price. Check the current pricing page before relying
  on them.
- **Thinking.** Billed as output. Sonnet and Opus did not think at all (0 thinking
  tokens, no thinking block); Haiku thought on both tries, and its thinking was
  90 % of its output (560 of 620).
- **Why Haiku needed two tries.** Its first-try thinking says it matched "the
  tuple-style `val` form used in the file" (`val (+) : (int, int) -> int`), but
  `bal l x d r` takes its arguments one at a time. The error it got back
  (`expected: empty`) does not say so directly; it worked it out on try 2.
- **Caching cost more than it saved on one-try runs.** The first request writes
  the prompt to the cache at 1.25× the input price, which only pays off when a
  retry reads it at 0.1×. Sonnet and Opus never retried, so they paid about 25 %
  more for their input than without caching (Sonnet ≈ $0.0039, Opus ≈ $0.0078
  uncached). Haiku's retry read 1,775 tokens back, and the run still came out
  slightly cheaper than uncached (≈ $0.0007).
- **Same counts for Sonnet and Opus.** Same prompt and tokenizer, same one-line
  answer and no thinking, so the same token counts; only the price per token
  differs.
