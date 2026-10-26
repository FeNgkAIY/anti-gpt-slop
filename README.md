# anti-gpt-slop

**Make Luna Great Again.** 🌕

English | [中文](README.zh-CN.md)

A [Pi](https://pi.dev) extension that catches a GPT model hedging, apologizing, and narrating its own caution instead of doing the work — and makes it fix the response before you ever see the slop.

## Origin: a essay called "应试智能" (Exam-Oriented Intelligence)

This ruleset isn't an arbitrary style preference. It's a direct translation of the GPT failure modes catalogued in a widely-circulated Chinese essay, *应试智能* ("Exam-Oriented Intelligence" / "Teaching-to-the-Test Intelligence"). Its core claim: every GPT answer is written for an imagined grader — padding never loses points, being wrong does — so it would rather be verbose, hedge every sentence, and qualify itself into unfalsifiability than risk saying anything that could be held against it. The literal request gets satisfied; the actual intent does not.

The essay's named symptoms map directly onto this ruleset:

- **Responsibility evasion / defensive writing** — phrases like "这不代表 XXX" ("this doesn't mean XXX"), "不能说明 XXX" ("this doesn't prove XXX"). In code, the same instinct builds a defensive fortress for a problem nobody has. This maps to Section 0's Zero Conversational Wrapper and the Section 4 blacklist.
- **Habitual negation / strawman reframing** — the essay's pattern: the "not X but Y" transition usually negates a misreading the user never held, or swaps the user's real question for an easier one ("the real question isn't X, it's Y"). This is the Section 4 banned "不是……而是……" / "真正要考虑的不是 X，而是 Y" family.
- **Jargon and verbal tics** — the essay lists them verbatim: 稳稳接住 (steadily catch it), 收口 (seal it off), 落盘 (land it), 口径 (the framing), 底牌 (the hole card), 链路 (the chain), 兜底 (the safety net), 闭环 (closed loop). That list went almost unchanged into Section 4's Banned Jargon.
- **No situational awareness / literal-minded execution** — the essay's example: told to clean a room, GPT stuffs all the trash into a cabinet — technically satisfying the literal instruction while missing the point; and the "you're right" reflex that fixes the symptom of being challenged rather than the underlying problem. This is what the audit flags as apology-as-deliverable: apologies, admissions, and "let me check" cannot substitute for the actual investigation or fix.
- **Cooperative principle** — the essay invokes linguist H. Paul Grice's maxims (quantity, quality, relevance, manner) to argue that GPT's filler is elaborately-dressed, technically-true, information-free padding. That same citation is the direct source of this ruleset's Section 1, "Core Alignment: Gricean Cooperation Over Defensive Correctness."

What this extension does is take that essay's catalogue of symptoms and turn it into something a model can be audited against and asked to self-correct.

> A real case, for context: during an actual incident investigation, `gpt-6-luna` found the correct root cause and wrote a solid conclusion — then closed with "I won't treat the absence of a log as evidence this didn't happen. No code or buyer data was changed this time." Nobody asked whether anything was changed; the mandatory Session Handoff already said `Modified Files: None`. **Make Luna Great Again** means not letting a self-congratulatory disclaimer drag down an otherwise solid finding.

## What it catches

- **Self-canceling disclaimers** that add zero decision-relevant information.
- **Habitual negation / strawman transitions** — "not X but Y" against a misreading nobody held.
- **Epistemic-process narration** — stating *how carefully* it's reasoning instead of just stating the actual finding or gap.
- **Apology-as-deliverable** — "sorry", "you're right", "let me check" used as a substitute for actually investigating, implementing, or answering.
- **Unsupported project claims** — assertions about your codebase with no trace evidence (no file read, no test run, no log checked) behind them.
- **Invented history** — "you said X earlier" / "this corrects my previous answer" when no such prior claim exists.
- **Redundant restatement** of facts already covered by the mandatory Session Handoff block (e.g. announcing "nothing was changed" when `Modified Files: None` already says so).
- **Jargon and verbal tics** — the 稳稳接住 / 收口 / 落盘 / 口径 / 底牌 / 链路 / 兜底 / 闭环 family.

## Scope: GPT models only

This extension activates only when the active model's identity matches `gpt` (case-insensitive), or is explicitly listed in `PI_GPT_AUDIT_MODELS`. On any other model family it is a silent no-op by design — no rules injected, no audit call made. If you're running Claude or Gemini and nothing happens, that's expected, not broken.

## What it costs

Every GPT turn adds at least one intent-preflight model call plus one audit call against the proposed final response. A detected violation adds a consistency-check call and can trigger up to two bounded self-correction turns, each repeating the audit. Expect real extra latency and token cost per turn — this trades speed for not shipping slop.

## Install

```bash
# from GitHub
pi install git:github.com/FeNgkAIY/anti-gpt-slop@v0.1.0

# from npm
pi install npm:anti-gpt-slop

# try once without installing
pi -e ./anti-gpt-slop/extensions/anti-gpt-slop.ts
```

## Configuration

| Variable | Effect |
|---|---|
| `PI_GPT_AUDIT_MODELS` | Comma-separated extra model identities (`provider/id`) treated as GPT for audit purposes. |
| `PI_GPT_AUDIT_PROVIDER` / `PI_GPT_AUDIT_MODEL` | Run the audit itself through a specific provider/model instead of the session's active model. Both must be set together. |

## Project-level rules

Run `/anti-gpt-slop` in a session to add, edit, or remove rules that supplement the built-in ruleset for the current project only. They're stored under `.pi/anti-gpt-slop.rules.json` in the project root.

## License

MIT
