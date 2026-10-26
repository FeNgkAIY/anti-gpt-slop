# anti-gpt-slop

**Make Luna Great Again.** 🌕

A [Pi](https://pi.dev) extension that catches a GPT model hedging, apologizing, and narrating its own caution instead of doing the work — and makes it fix the response before you ever see the slop.

> 项目的起因很简单：某次线上事故排查，`gpt-6-luna` 查到了真正的根因，结论写得很扎实，然后在结尾加了一句——"我不会把缺少日志记录当作某个操作没发生的证据。本次没有改动代码或买家数据。" 没人问它改没改东西，`Session Handoff` 里 `Modified Files: None` 已经写明了。这就是典型的 GPT 废话：好好的结论，非要裹一层自我辩护。**Make Luna Great Again** 不是反讽 Luna 本身，是不让它被这种口癖拖后腿。

[![npm](https://img.shields.io/npm/v/anti-gpt-slop)](https://www.npmjs.com/package/anti-gpt-slop)
[![license](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

---

## What it catches / 它抓什么

The built-in ruleset targets concrete GPT failure modes, not vague "be better" vibes:

- **Hedging & self-canceling disclaimers** that add zero decision-relevant information.
- **Epistemic-process narration** — stating *how carefully* it's reasoning instead of just stating the actual finding or gap.
- **Apology-as-deliverable** — "sorry", "you're right", "let me check" used as a substitute for actually investigating, implementing, or answering.
- **Unsupported project claims** — assertions about your codebase with no trace evidence (no file read, no test run, no log checked) behind them.
- **Invented history** — "you said X earlier" / "this corrects my previous answer" when no such prior claim exists.
- **Redundant restatement** of facts already covered by the mandatory Session Handoff block (e.g. announcing "nothing was changed" when `Modified Files: None` already says so).

中文版：

- **自我辩护式免责声明**：说了等于没说，没提供任何新信息。
- **自述推理过程**：不直接说结论或缺口，先给自己的"严谨态度"打广告。
- **用道歉代替交付**："抱歉""你说得对""我去查一下"——说完就结束，没有真正查、改、答。
- **编造的项目结论**：对代码库下断言，但轨迹里没有对应的文件读取、测试运行或日志检查。
- **编造的历史**："你之前说过 X"——翻遍记录根本没有这句话。
- **重复 Handoff 已有信息**：Handoff 已经写了 `Modified Files: None`，正文又用大段文字强调一遍没改东西。

## Scope: GPT models only / 只对 GPT 生效

This extension activates only when the active model's identity matches `gpt` (case-insensitive), or is explicitly listed in `PI_GPT_AUDIT_MODELS`. On any other model family it is a silent no-op by design — no rules injected, no audit call made. If you're running Claude or Gemini and nothing happens, that's expected, not broken.

只有当前模型身份匹配 `gpt`（或显式配置在 `PI_GPT_AUDIT_MODELS` 里）时才会生效。换成其他模型家族，它什么都不做——这是设计，不是故障。

## What it costs / 代价

Every GPT turn adds at least one intent-preflight model call plus one audit call against the proposed final response. A detected violation adds a consistency-check call and can trigger up to two bounded self-correction turns, each repeating the audit. Expect real extra latency and token cost per turn — this trades speed for not shipping slop.

每个 GPT 回合至少多一次意图预判调用和一次审核调用；一旦判定违规，还会再加一次一致性校验，并最多触发两轮限次自我修正，每轮都重新审核。这是用延迟和 token 换"不让废话过关"。

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

## Project-level rules / 项目级规则

Run `/anti-gpt-slop` in a session to add, edit, or remove rules that supplement the built-in ruleset for the current project only. They're stored under `.pi/anti-gpt-slop.rules.json` in the project root.

在会话里运行 `/anti-gpt-slop` 即可为当前项目增删改补充规则，只对该项目生效，存放在项目根目录的 `.pi/anti-gpt-slop.rules.json`。

## License

MIT
