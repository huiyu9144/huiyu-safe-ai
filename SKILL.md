---
name: huiyu-safe-ai
description: "Lightweight AI security guard that intercepts risky install/download commands (npm, npx, pip, cargo, git clone) to block known malicious packages and scan for suspicious code. Invoke ONLY when user runs install/download/clone commands."
license: MIT
compatibility:
  - claude-code
  - openai-codex
  - tra
allowed-tools: read webfetch
user-invocable: true
disable-model-invocation: false
argument-hint: "<package-name-or-url>"
server: "https://github.com/huiyu9144/huiyu-safe-ai"
---

# Huiyu-SafeAi — Lightweight AI Supply Chain Shield

A zero-overhead, 3-step security checkpoint that runs before any install or download command. It blocks known malicious packages, verifies package identity, and performs a quick code sniff when needed.

---

## Trigger Conditions (MUST match before activation)

This skill activates ONLY when the user's message contains one of these command patterns:

| Command Pattern | Examples |
|----------------|----------|
| `npm install` / `npm i` | `npm i express` |
| `npx` | `npx create-react-app` |
| `git clone` | `git clone https://...` |
| `pip install` / `pip3 install` | `pip install requests` |
| `cargo install` | `cargo install ripgrep` |
| `yarn add` / `yarn install` | `yarn add lodash` |
| `pnpm add` | `pnpm add vue` |

**If no install/download command is present, DO NOT activate this skill. Stay silent.**

---

## Check Flow (3 Steps, Most Exit at Step 1)

```
[User wants to install/download]
         |
         v
   +-----------------+
   |  STEP 1: BLOCK? |  <- Check blocklist (instant)
   |  If blocked -> RED |
   +-------+---------+
           | not blocked
           v
   +-----------------+
   |  STEP 2: TRUSTED?|  <- Check identity (fast)
   |  If trusted -> GREEN |
   +-------+---------+
           | unknown
           v
   +-----------------+
   |  STEP 3: SNIFF?  |  <- Quick code scan (if available)
   |  Malicious -> RED  |
   |  Clean -> YELLOW   |
   +-----------------+
```

---

## STEP 1 — Blocklist Check

Check the extracted package name or repo against the known blocklist below. If matched, immediately output RED BLOCKED.

### Known Malicious Packages (Blocklist)

| Package | Ecosystem | Threat |
|---------|-----------|--------|
| `event-stream@3.3.6-1` | npm | Supply chain attack, steals cryptocurrency |
| `flatmap-stream` | npm | Credential and crypto wallet theft |
| `ua-parser-js@0.7.29` | npm | Malicious miners injected |
| `coa@2.0.2`, `rc@1.2.8` | npm | Credential stealing via environment variables |
| `colors@1.4.1`, `faker@6.6.6` | npm | Supply chain protest — intentionally crashes |
| `crossenv@7.0.3` | npm | Typosquatting for cross-env, installs miners |
| `babelcli@6.0.0` | npm | Typosquatting for babel-cli, steals SSH keys |
| `mongose@5.2.10` | npm | Typosquatting for mongoose |
| `getcookies.js` | npm | Malicious package, steals browser cookies |
| `notion-api-sdk@1.0.0` | npm | Fake SDK, exfiltrates environment variables |
| `solders` | npm | Typosquatting for Python solders library |
| `nodemailer-js` | npm | Typosquatting for nodemailer |
| `node-opencv` | npm | Malicious dependency chain |
| `opencv.js` | npm | Hidden cryptocurrency miner |
| `npm-script-demo` | npm | Supply chain attack vector |
| `url-parse` (malicious versions) | npm | Prototype pollution |
| `react-native-cli` (fake) | npm | Installs cryptocurrency miner |
| `node-ipc` (compromised versions) | npm | Geopolitical destructive payload |
| `peacenotwar` | npm | Destructive code in protest |
| `ts-migrate` (fake versions) | npm | Malicious backdoor |
| `npmpacket` | npm | Typosquatting for node-libs-browser |
| `pac-resolver` (vulnerable) | npm | SSRF vulnerability exploit |
| `shell-quote` (vulnerable) | npm | Command injection |
| `webhook` (malicious) | npm | Reverse shell backdoor |
| `@anthropic-ai/sdk-spyware` | npm | Impersonation, data exfiltration |
| `openai-api-spyware` | npm | Impersonation, steals API keys |
| `deepseek-py` | npm | Typosquatting for legitimate DeepSeek packages |
| `deepseek-sdk` | npm | Typosquatting, malicious payload |
| `whisper.cpp-node` | npm | Impersonation, crypto miner |
| `sqlite-wasm-spy` | npm | Data exfiltration |
| `ethereum-wallet-keygen` | npm | Steals private keys |
| `crypto-wallet-gen` | npm | Steals wallet seeds |
| `rustup-nightly-unofficial` | npm | Typosquatting for rust toolchain |
| `golangci-lint-go` | npm | Typosquatting for Go linter |
| `eslint-config-internal` | npm | Credential stealing |
| `eslint-plugin-prettierx` | npm | Backdoor injection |
| `babel-preset-es2015-modified` | npm | Malicious transpiler |
| `webpack-plugin-spy` | npm | Build-time data exfiltration |
| `typescript-definitions-ext` | npm | Steals environment variables |
| `npm-install-package` | npm | Recursive installation bomb |
| `node-extensions-utils` | npm | Hidden reverse shell |
| `async-promises-extra` | npm | Supply chain attack |
| `express-middlewares-pro` | npm | Malicious middleware |
| `vite-plugin-vue-extend` | npm | Destructive payload disguised as Vite plugin |
| `quill-image-downloader` | npm | Destructive payload disguised as Quill plugin |
| `js-hood` | npm | Destructive payload, deletes project files |
| `js-bomb` | npm | Recursive file deletion, triggers system shutdown |
| `vue-plugin-bomb` | npm | Destructive payload targeting Vue.js projects |
| `vite-plugin-bomb` | npm | Destructive payload targeting Vite projects |
| `vite-plugin-bomb-extend` | npm | Destructive payload targeting Vite projects |
| `vite-plugin-react-extend` | npm | Destructive payload disguised as React plugin |
| `citiycar8` | npm | Phishing redirect, fake Office 365 login |
| `solaibot` | npm | VS Code extension, steals crypto wallet credentials |
| `among-eth` | npm | VS Code extension, steals Ethereum wallets |
| `blankebesxstnion` | npm | VS Code extension, steals crypto credentials |
| `bbb335656` | npm | System fingerprinting, sends data to Discord webhook |
| `cdsfdfafd1232436437` | npm | System fingerprinting, sends data to Discord webhook |
| `sdsds656565` | npm | System fingerprinting, sends data to Discord webhook |
| `reques7s` | pypi | Typosquatting for requests, steals credentials |
| `jeIlyfish` | pypi | Typosquatting for jellyfish, remote code execution |
| `python-dateutil2` | pypi | Typosquatting for python-dateutil, data exfiltration |
| `colorama-backup` | pypi | Typosquatting for colorama, reverse shell |
| `python3-dateutil` | pypi | Typosquatting for python-dateutil, credential theft |
| `requirements-parser2` | pypi | Typosquatting for requirements-parser, backdoor |
| `django-server` | pypi | Typosquatting for Django, remote code execution |
| `flask-app` | pypi | Typosquatting for Flask, data exfiltration |
| `requests-oauth` | pypi | Typosquatting for requests-oauthlib, credential theft |
| `beautifulsoup4-backup` | pypi | Typosquatting for beautifulsoup4, reverse shell |

### Blocklist Check Rules

1. Match against the **full package name** (including scoped names like `@org/pkg`)
2. For typosquatting detection: check edit distance against popular packages:
   - `express` vs `expresss`, `expres`, `expressjs`, `express-middleware` (if unofficial)
   - `lodash` vs `lodosh`, `lodas`, `lodashs`
   - `react` vs `reacct`, `reactt`
   - `next` vs `nextjs-official` (if unofficial), `nextjs`
3. Match against known malicious GitHub repos by name pattern
4. **If match found -> output RED BLOCKED, stop here**

---

## STEP 2 — Identity Verification

If the package is NOT on the blocklist, verify whether it is a **trusted/official** source.

### Trusted Source Criteria (Green Light)

A package is considered **TRUSTED** if it meets ANY of these conditions:

| Criterion | How to Check |
|-----------|-------------|
| **Official org repo** | GitHub repo belongs to verified org (e.g., `deepseek-ai`, `facebook`, `vuejs`, `vercel`, `angular`, `sveltejs`, `rust-lang`, `python`, `nodejs`, `denoland`) |
| **High star count** | GitHub repo has 1,000+ stars |
| **npm official badge** | Package name matches the official project name exactly |
| **High download count** | npm weekly downloads > 10,000 |
| **Known maintainer** | Maintainer is a known open-source contributor |
| **Ecosystem default** | Package is the de facto standard (e.g., `express` for Node.js HTTP, `pandas` for Python data) |

### Quick Identity Check Flow

1. **If user provides a GitHub URL** -> Read the page to check stars, org name, description
2. **If user provides a package name** -> Check via web for download count, official status
3. **If the package is well-known** (you already know it) -> Green light immediately, no check needed

### Known Trusted Orgs (Auto-approve, no further check)

```
deepseek-ai, openai, anthropic, facebook, meta, google, microsoft,
vercel, nextjs, vuejs, angular, sveltejs, reactjs, rust-lang,
python, golang, nodejs, denoland, deno, docker, kubernetes,
elastic, grafana, prometheus, redis, postgres, mongodb,
npm, yarn, pnpm, webpack, vite, esbuild, swc, turborepo,
supabase, prisma, trpc, tailwindcss, shadcn-ui, radix-ui,
lucide, heroicons, arco-design, ant-design, element-plus,
antfu, sucrase, tsup, unbuild, unocss, pinia, zustand, jotai,
axios, got, node-fetch, fastify, koa, hono, expressjs, nestjs
```

### If TRUSTED -> output GREEN and stop

---

## STEP 3 — Quick Code Sniff

If the package is **not blocked** and **not verified as trusted**, perform a lightweight code sniff.

This step ONLY runs for:
- Unknown packages with low download counts
- Personal GitHub repos (not org-owned)
- Packages with suspicious names

### What to Sniff (Pick 3 Files Max)

Do NOT scan the entire repo. Only look at these files if available:

1. **`package.json`** — Check `scripts.postinstall`, `scripts.install`, `scripts.preinstall`
2. **`setup.py`** or **`pyproject.toml`** — Check `cmdclass`, `entry_points`
3. **`Makefile`** or **`Dockerfile`** — Check for suspicious commands
4. **One random `.js` or `.py` file** — Glance for obfuscation patterns

### Red Flags to Look For

| Pattern | Risk Level | What It Means |
|---------|-----------|---------------|
| `postinstall` script running `curl`, `wget`, `node -e`, `eval()`, `exec()` | HIGH | Almost certainly malicious |
| `child_process.exec` or `child_process.spawn` with network URLs | HIGH | Remote code execution |
| Base64 encoded strings > 200 chars passed to `eval` or `exec` | HIGH | Obfuscated payload |
| Reading `~/.ssh/`, `~/.aws/`, `~/.gnupg/` or `~/.config/` | HIGH | Stealing credentials |
| Writing to `/etc/`, `C:\Windows\System32\` | HIGH | System tampering |
| `fetch('http://` or `requests.post('http://` to obscure domains | MED | Possible data exfiltration |
| `process.env` / `os.environ` bulk reading (5+ keys) | MED | Environment harvesting |
| Network calls in install scripts | MED | Unexpected remote activity |
| File downloads during install | MED | Suspicious payload delivery |
| `fs.writeFile` to home directory with hidden filenames | HIGH | Dropper behavior |

### Sniff Output

- **HIGH risk patterns found** -> BLOCK immediately, show which pattern triggered
- **MEDIUM risk or no patterns found** -> Warn user with yellow light

---

## Output Formats

### GREEN — Safe to Install

```
Huiyu-SafeAi: SAFE
Package: {package-name}
Source:  {trusted-source-reason}
Verdict: No issues detected. Safe to proceed.
```

### YELLOW — Caution

```
Huiyu-SafeAi: CAUTION
Package: {package-name}
Source:  {unknown-source-type} (e.g., "personal repo, low download count")
Risks:  {list of concerns or "No known threats, but unverified source"}
Advice: Review the package manually before proceeding.
        Consider checking the source code at: {url}
```

### RED — Blocked

```
Huiyu-SafeAi: BLOCKED
Package: {package-name}
Reason: {specific reason — blocklist match / malicious pattern found}
Threat: {brief description of threat}
Action: DO NOT install. Use an alternative package instead.
        {suggest alternative if possible}
```

---

## Performance Rules (MUST Follow)

1. **DO NOT** scan files or fetch web pages if the package is in the blocklist or trusted list
2. **DO NOT** read more than 3 files during code sniffing
3. **DO NOT** run any commands (no `npm audit`, no `npm view`) — this is a static prompt-only check
4. **DO NOT** output if the user is not running an install/download command
5. **Total output should be <= 10 lines** — keep it fast and scannable
6. **Never block popular, well-known packages** — if in doubt, yellow light, not red

---

## Known Trusted Packages (Bypass All Checks)

These packages are always GREEN, no verification needed:

```
express, koa, fastify, hono, next, nuxt, gatsby, remix, vite, webpack
react, react-dom, vue, vue-router, pinia, vuex, angular, svelte, solid-js
lodash, axios, got, node-fetch, undici, ky, superagent
typescript, eslint, prettier, babel, swc, esbuild, turborepo
tailwindcss, postcss, sass, styled-components, emotion
prisma, drizzle-orm, mongoose, sequelize, typeorm
zod, yup, joi, ajv
zustand, jotai, recoil, valtio, xstate
next-auth, lucia, passport
openai, @anthropic-ai/sdk, langchain
@hono/zod-validator, tRPC, graphql
vitest, jest, mocha, cypress, playwright
pandas, numpy, scipy, scikit-learn, matplotlib
requests, httpx, fastapi, flask, django
click, typer, pydantic
pytest, black, ruff, mypy
ripgrep, fd, bat, exa, zoxide, starship, delta, fzf
docker, kubectl, helm
```

---

## Notes

- This skill is a **prompt-level guardrail**, not a runtime scanner
- It complements, but does not replace, tools like `npm audit`, `snyk`, or `socket.dev`
- The blocklist should be manually updated as new threats emerge
- For enterprise use, combine with actual CI/CD security scanning
