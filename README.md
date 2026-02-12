# Chrome Test Runner Plugin

A Claude Code plugin that turns Claude into a QA testing agent. It drives your real Chrome browser, executes test scenarios, and produces unified diagnostic logs.

**No custom MCP server needed** — this plugin uses the [Claude in Chrome](https://code.claude.com/docs/en/chrome) extension that's already built into Claude Code.

## What it produces

A unified markdown log that interleaves three layers for every test step:

- **🧠 Reasoning** — what Claude is thinking, what it expects, what it's watching for
- **🎯 Actions** — the browser interactions (click, type, navigate, scroll)
- **📡 Telemetry** — network requests, console logs, JS errors, screenshots

Plus a bug report and summary statistics.

## Setup

### Prerequisites

1. Claude Code installed and up to date
2. [Claude in Chrome extension](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn) installed in Chrome

### Install the plugin

**From local directory (development):**

```bash
claude --plugin-dir /path/to/chrome-test-runner-plugin-v2
```

**From GitHub:**

```bash
# Inside Claude Code:
/plugin add github:your-username/chrome-test-runner-plugin
```

### Start Claude Code with Chrome

```bash
claude --chrome
```

Or enable it permanently: run `/chrome` inside Claude Code and select "Enabled by default".

## Usage

```
/chrome-test-runner:test https://myapp.vercel.app — stress test the chat, send rapid messages, create child chats
```

Or just describe what you want in natural language — Claude will auto-delegate to the test-runner agent:

```
Can you click around my app at localhost:3000 and find bugs?
Send several messages, try concurrent operations, and capture everything.
```

## What's in the box

```
chrome-test-runner-plugin-v2/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── agents/
│   └── test-runner.md       # The QA test execution agent
├── skills/
│   └── chrome-testing/
│       └── SKILL.md         # Testing knowledge base (auto-loaded)
├── commands/
│   └── test.md              # /chrome-test-runner:test slash command
└── README.md
```

| Component | Purpose |
|-----------|---------|
| **test-runner agent** | Plans test scenarios, executes them with the flush→act→capture cycle, writes reasoning, flags bugs, compiles the unified log |
| **chrome-testing skill** | Auto-loaded knowledge about bug patterns, telemetry tools, console message triage, JS inspection snippets |
| **test command** | `/chrome-test-runner:test` shortcut to trigger the agent |

## How it works

The test-runner agent uses Claude in Chrome's built-in tools:

| Tool | Purpose |
|------|---------|
| `navigate` | Go to URLs |
| `computer` | Click, type, scroll, screenshot |
| `find` | Locate elements by description |
| `form_input` | Fill form fields |
| `read_network_requests` | Capture HTTP traffic |
| `read_console_messages` | Capture console output |
| `javascript_tool` | Run JS for deep inspection |
| `gif_creator` | Record interaction GIFs |

The key pattern is **flush → act → capture**: before each action, clear the telemetry buffers; after each action, read fresh telemetry. This gives you clean per-step data instead of one giant dump at the end.

## Example output

See [EXAMPLE-LOG.md](./EXAMPLE-LOG.md) for a full sample of what the unified log looks like.
