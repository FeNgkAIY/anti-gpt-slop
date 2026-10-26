# anti-gpt-slop

A [Pi](https://pi.dev) extension that audits a GPT model's response against a built-in ruleset before it reaches the user, and asks the model to self-correct when it finds a grounded violation.

## What it catches

The built-in ruleset targets common GPT failure modes rather than generic style preferences:

- Hedging, self-canceling qualifications, and unprompted disclaimers that add no decision-relevant information.
- Narrating evidentiary caution instead of stating the actual unresolved question (e.g. "I won't treat the absence of a log as evidence this didn't happen").
- Apologies, status narration, or promises to investigate used as a substitute for completing the requested analysis, investigation, or implementation.
- Claims about the current project that are not backed by trace evidence (source, tests, logs, or tool results).
- Misattributed "you said X" / "this corrects the earlier statement" claims not backed by an actual prior message.
- Restating facts already carried by the mandatory Session Handoff block (e.g. announcing "no changes were made" when `Modified Files: None` already says so).

## Scope: GPT models only

This extension only activates when the active model's identity matches `gpt` (case-insensitive, by provider/model id), or is listed in `PI_GPT_AUDIT_MODELS`. On any other model it is a no-op by design: no rules are injected, no audit runs. If you install this and the active model isn't GPT, that is expected, not a bug.

## What it costs

Every GPT turn adds at least one extra model call (an intent-preflight classification) plus one audit call against the proposed final response. If the audit finds a violation, it adds one more consistency-check call and can trigger up to two bounded self-correction turns, each of which repeats the audit. Expect noticeably higher token usage and latency per turn compared to running GPT without this extension.

## Install

From a local checkout:

```bash
pi install ./anti-gpt-slop
```

From GitHub:

```bash
pi install git:github.com/FeNgkAIY/anti-gpt-slop@v0.1.0
```

From npm (once published):

```bash
pi install npm:anti-gpt-slop
```

To try it for one invocation without adding it to settings:

```bash
pi -e ./anti-gpt-slop/extensions/anti-gpt-slop.ts
```

## Configuration

| Variable | Effect |
|---|---|
| `PI_GPT_AUDIT_MODELS` | Comma-separated extra model identities (`provider/id`) treated as GPT for audit purposes. |
| `PI_GPT_AUDIT_PROVIDER` / `PI_GPT_AUDIT_MODEL` | Run the audit itself through a specific provider/model instead of the session's active model. Both must be set together. |

## Project-level rules

Run `/anti-gpt-slop` in a session to add, edit, or remove project-specific rules that supplement the built-in ruleset for the current project. They are stored under `.pi/anti-gpt-slop.rules.json` in the project root and apply only to that project.

## License

MIT
