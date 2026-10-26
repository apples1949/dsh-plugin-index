# PR 文案：把两个插件加进 awesome-dsh-plugin

目标仓库：<https://github.com/awesome-dsh-plugin/awesome-dsh-plugin>（主分支 `main`，条目就在 `README.md` 里，一个分类一个 `###` 段，段内按 `owner/repo` 字母序排）

## PR 标题

```
Add apples1949/dsh-thinking-tag-guard and apples1949/dsh-speed-meter
```

## 要改的地方（两行，各插在自己分类的字母序位置）

插入 `### Sessions & Messages` 段：

```markdown
- [apples1949/dsh-thinking-tag-guard](https://github.com/apples1949/dsh-thinking-tag-guard) - Moves reasoning that a gateway leaked into the assistant text channel back into the reasoning channel (thinking / summary / analysis in any spelling, bracketed or bare, including pairs mid-block), drops text that just replays the reasoning already streamed, strips gateway-private `<|DSML|…>` tokens from both channels even when a token is split across deltas, and keeps a tool-less step from closing the turn silently when its answer is missing or is leaked thinking.
```

插入 `### UI Enhancements` 段：

```markdown
- [apples1949/dsh-speed-meter](https://github.com/apples1949/dsh-speed-meter) - Live output speed (tok/s) shown as a small pill below the composer: rolls while the model streams and shows the previous turn's average when idle. No timers — it estimates from streaming deltas, self-calibrates against the server's `usage.outputTokens`, and refuses to display a number when the reply arrived in one burst.
```

## PR 正文（英文，直接粘贴）

```markdown
Adds two DeepSeek Harness plugins I wrote for my own daily use.

**dsh-thinking-tag-guard** (Sessions & Messages, Host plugin)
Some gateways return a model's reasoning inside the `content` channel instead of
`reasoning_content`, dressed in reasoning scaffolding: `<thinking>…</thinking>`,
a bare `thinking`, a lone `/thinking` at the end of a block, or a bracketed pair
anywhere in it. The inbound parser only reads the reasoning fields, so that text
is rendered as the assistant's answer. Worse, such a step usually calls no tool,
so the turn closes "completed" with the leaked thinking as its final answer.
The plugin hooks `llm/stream` and rewrites the leaked segments into reasoning
blocks (it only corrects the `block-end` payload, so nothing is re-emitted), and
hooks `agent/turn-stopping` to steer the turn onward when it is about to close on
a leaked or empty answer. It also strips the gateway's private `<|DSML|…>`
special tokens from both channels, including tokens split across deltas.

- `dsh plugin --profile web add github:apples1949/dsh-thinking-tag-guard`
- Declares `dsh.bundle.patch` (inserts loader id `thinking-tag-guard`).
- Tests: `test/run.mjs` drives real chunk streams through the plugin and judges
  the result with DSH's own `BlockAssembler`, plus a recorded corpus (42 real
  leaks must be reclassified, 11 real answers must pass untouched);
  `test/dsml.mjs` covers the token stripping (10 checks, no dependencies).

**dsh-speed-meter** (UI Enhancements, Web client plugin)
A pill under the composer showing live output speed in tok/s. It estimates tokens
from streaming deltas (CJK 0.6, other 0.25 per character), blends a 1.5 s sliding
window with the step-wide rate through an EMA, and calibrates the scale against
the server's real `usage.outputTokens`. No timers: it re-renders only when the
stream delivers something, and it degrades to "no number" rather than guess when
a reply arrives in one burst.

- `dsh plugin --profile web add github:apples1949/dsh-speed-meter`
- Declares `dsh.client.platform: "web"` with an `exports["./client"]` bundle.

Both are read-only about the session log, ship no telemetry, and depend on
nothing beyond DSH itself. They are published as-is from their repos — I keep
no release cadence — so listing them means "this installs and does what the line
says", not "this is maintained on a schedule".
```

## ⚠️ 提交前要注意的两件事

1. **收录标准里有一条 "is maintained"** —— 我们 README 里明说"仅按需更新、不保证更新"，可能被维护者挑出来。可以接受，但别惊讶；PR 正文里我已经写成 "I keep no release cadence" 而不是"不维护"，语气上留了余地。
2. **speed-meter 没有声明 `dsh.bundle`**。这个列表第 11 行的原话是：*"This list collects community plugins that are installable via `dsh plugin add` (each declares a `dsh.bundle` manifest)"*。speed-meter 只有 `dsh.client`，现在靠 dshmarket 的 *client-only shim* 挂载。两条路：
   - 直接提 PR，在正文里说明它是 Web 客户端插件、用 `dsh plugin add` 可装（可能被要求补 bundle）；
   - 或者先给它加一个 `cordis.patch.yml`（`insert` 一行 `name: '@apples1949/dsh-speed-meter'`）再提——**但这需要重装 + 重启宿主做实测**，没验证过不能声称能用，别急着加。
