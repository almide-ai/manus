# manus

> Claude's hand on your Mac — a computer-use agent written in [Almide](https://github.com/almide/almide).

`manus` (Latin: *hand*) gives Claude eyes and hands on a **real macOS
desktop**. It screenshots the screen, sends it to the Anthropic Messages
API `computer` tool, and executes the actions Claude returns
(clicks, typing, key presses) with [`cliclick`](https://github.com/BlueM/cliclick).
Screenshot → think → act, in a loop, until the task is done.

No Docker, no sandbox VM — it drives *your* machine. That is powerful and
risky; every action asks for confirmation unless you opt out.

## Requirements

- **macOS** (Apple Silicon or Intel)
- `ANTHROPIC_API_KEY` in the environment
- [`cliclick`](https://github.com/BlueM/cliclick) on `PATH` — `brew install cliclick`
- **Screen Recording** + **Accessibility** permission granted to the
  terminal you run it from (System Settings → Privacy & Security)

## Build & run

```sh
almide build src/main.almd -o manus     # compile to a native binary
export ANTHROPIC_API_KEY=sk-ant-...
./manus "open Safari and search for the Almide language"
```

Or run straight from source:

```sh
almide run src/main.almd -- "take a screenshot and describe what you see"
```

### Environment

| Var | Meaning |
|---|---|
| `ANTHROPIC_API_KEY` | **required** — your Anthropic API key |
| `MANUS_MODEL` | model id (default `claude-sonnet-4-6`) |
| `MANUS_YOLO=1` | skip the per-action confirmation prompt |
| `MANUS_TASK` | task text, if not passed as an argument |

## Safety

`manus` controls your real machine. Mis-clicks and mistyped text can do
real damage (delete files, send messages). Defaults are conservative:

- **Every action prompts `[y/N]`** before it runs. `MANUS_YOLO=1` disables this.
- Watch it work — don't walk away.
- Prefer running it in a **dedicated macOS user account** for anything risky.
- There is a hard cap of 40 steps per run.

## How it works

1. Read the logical screen size via `osascript` (Finder desktop bounds).
2. `screencapture` the screen, downscale to logical points with `sips`
   (so Claude's coordinates map 1:1 onto `cliclick`'s), base64-encode.
3. Call `POST /v1/messages` with the `computer_20250124` tool and the
   `computer-use-2025-01-24` beta header (built directly with the `http`
   and `json` stdlib modules).
4. Parse the returned `tool_use` block, execute the action via `cliclick`,
   and reply with a fresh screenshot as a `tool_result`.
5. Loop until Claude stops without requesting a tool.

Supported actions: `screenshot`, `left_click`, `right_click`,
`double_click`, `middle_click`, `mouse_move`, `type`, `key`, `wait`,
`cursor_position`. (`scroll` and `left_click_drag` return a "not
supported" result in v0.1.)

## Status

Early (v0.1). Written to dogfood Almide on a real, effect-heavy program:
subprocess control, HTTP, JSON, an agent loop. Known upstream issue found
while building it: `almide test` (WASM target) trips a Perceus
reference-counting verification bug in the compiler on this module,
though `almide build` (Rust target) is clean — see [CLAUDE.md](CLAUDE.md).

## License

MIT
