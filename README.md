# Huiyu-SafeAi

Lightweight AI security guard for install/download commands. Blocks known malicious packages, verifies package identity, and scans for suspicious code — all in under 1 second.

## What It Does

Every time you run `npm install`, `npx`, `git clone`, `pip install`, or `cargo install`, Huiyu-SafeAi automatically:

1. **Checks a blocklist** of 40+ known malicious packages (instant)
2. **Verifies identity** — is this from an official org or a personal repo?
3. **Sniffs code** — only for unknown packages, only 3 files max

Output: GREEN (safe), YELLOW (caution), or RED (blocked).

## Supported Platforms

| Platform | Install Path |
|----------|-------------|
| Claude Code | `~/.claude/skills/huiyu-safe-ai/` |
| OpenAI Codex CLI | `~/.codex/skills/huiyu-safe-ai/` |
| Trae | `.trae/skills/huiyu-safe-ai/` |
| Any SKILL.md compatible tool | Copy the folder |

## Quick Install

### Claude Code

```bash
git clone https://github.com/huiyu9144/huiyu-safe-ai.git ~/.claude/skills/huiyu-safe-ai
```

### OpenAI Codex CLI

```bash
git clone https://github.com/huiyu9144/huiyu-safe-ai.git ~/.codex/skills/huiyu-safe-ai
```

### Manual

```bash
git clone https://github.com/huiyu9144/huiyu-safe-ai.git
# Copy the huiyu-safe-ai folder to your skills directory
```

## Features

- **40+ known malicious packages** in blocklist (event-stream, flatmap-stream, deepseek-py, etc.)
- **60+ trusted organizations** auto-approved (deepseek-ai, openai, anthropic, facebook, etc.)
- **70+ trusted packages** bypass all checks (express, react, pandas, etc.)
- **Typosquatting detection** — catches name-similar malicious packages
- **Code sniffing** — detects postinstall exploits, credential theft, obfuscated payloads
- **Zero overhead** — no network calls, no commands executed, prompt-only check

## How It Works

```
User runs: npm install <package>
         |
   STEP 1: Blocklist?  --> YES --> BLOCKED (RED)
         | NO
   STEP 2: Trusted?    --> YES --> SAFE (GREEN)
         | UNKNOWN
   STEP 3: Code Sniff  --> Malicious --> BLOCKED (RED)
         | Clean
         v
      CAUTION (YELLOW)
```

## Example Output

### Safe Package

```
Huiyu-SafeAi: SAFE
Package: express
Source:  Official npm package, 30M+ weekly downloads
Verdict: No issues detected. Safe to proceed.
```

### Blocked Package

```
Huiyu-SafeAi: BLOCKED
Package: deepseek-py
Reason: Blocklist match — known typosquatting package
Threat: Malicious payload, credential/data exfiltration
Action: DO NOT install. Use official DeepSeek API instead.
```

### Unknown Package

```
Huiyu-SafeAi: CAUTION
Package: some-random-lib
Source:  Personal repo, low download count
Risks:  No known threats, but unverified source
Advice: Review the package manually before proceeding.
```

## Updating the Blocklist

The blocklist is embedded in `SKILL.md`. To add new malicious packages:

1. Edit the "Known Malicious Packages" table in `SKILL.md`
2. Add the package name, ecosystem, and threat description
3. Commit and push

## Contributing

Contributions welcome! Areas where help is needed:

- Expanding the blocklist with newly discovered malicious packages
- Adding more trusted organizations
- Improving typosquatting detection rules
- Adding support for more ecosystems (Ruby gems, Go modules, etc.)

## License

MIT

## Acknowledgments

Built as a prompt-level security guardrail. Complements, but does not replace, tools like `npm audit`, `snyk`, or `socket.dev`.
