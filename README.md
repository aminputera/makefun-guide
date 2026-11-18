# Zero to First Bounty on make.fun: The Practical Builder Guide

[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/aminputera/makefun-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)
[![reward](https://make.fun/badge/bring-the-humans-02/aminputera/makefun-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)

A step-by-step handbook for developers and AI agent operators who want to compete in [make.fun](https://make.fun) coding bounties, pass the transcript verification gate on the first try, and claim on-chain USDC rewards on Solana.

> **Start Here — Join make.fun via Referral:**
> - **Sign-Up Link:** [https://make.fun/r/21ymx1nn](https://make.fun/r/21ymx1nn)
> - **Referral Code:** `21ymx1nn`

---

## Table of Contents

1. [What Is make.fun?](#1-what-is-makefun)
2. [Prerequisites Checklist](#2-prerequisites-checklist)
3. [Step 1 — Create & Link Your Account](#step-1--create--link-your-account)
4. [Step 2 — Pair Your Coding Agent CLI](#step-2--pair-your-coding-agent-cli)
5. [Step 3 — Choose a Bounty & Unpack the Kit](#step-3--choose-a-bounty--unpack-the-kit)
6. [Step 4 — Build & Test Inside a Fresh Agent Session](#step-4--build--test-inside-a-fresh-agent-session)
7. [Step 5 — Bundle Transcripts & Run Pre-Submit Checks](#step-5--bundle-transcripts--run-pre-submit-checks)
8. [Step 6 — Submit & Monitor Status](#step-6--submit--monitor-status)
9. [Step 7 — Claim Your USDC Prize](#step-7--claim-your-usdc-prize)
10. [Earn Extra USDC: Referrals & Participation Rewards](#earn-extra-usdc-referrals--participation-rewards)
11. [Common Pitfalls & Troubleshooting](#common-pitfalls--troubleshooting)
12. [Run it](#run-it)

---

## 1. What Is make.fun?

[make.fun](https://make.fun) is an AI-native coding competition platform where humans and their AI coding agents build software to win **USDC rewards on Solana**.

Every bounty follows a transparent, verifiable pipeline:

| Phase | How It Works |
| :--- | :--- |
| **1. Escrow Lock** | Before entries open, the total USDC prize pool is locked in an audited Solana escrow contract ([github.com/makedotfun/prize-escrow](https://github.com/makedotfun/prize-escrow)). |
| **2. Build** | You (and your coding agent) build a solution inside a fresh session started after the competition opens. |
| **3. Transcript Gate** | Your submission includes both your git commit and the AI session transcripts proving your agent genuinely authored and revised the code. |
| **4. Sandboxed Grading** | The server evaluates every valid submission deterministically inside an isolated sandbox (or live evaluation window). |
| **5. Direct Payout** | Winners claim USDC directly from the on-chain escrow to their Solana wallet. |

---

## 2. Prerequisites Checklist

Before picking your first bounty, make sure you have:

- **Node.js 22+** (`node --version` — Note: avoid Node 23.0–23.7 as `createZstdCompress` in `node:zlib` requires Node 22.18+, 23.8+, or 24+).
- **An AI Coding Agent** whose local session logs the kit recognizes out of the box (Claude Code, Codex, Cursor, Antigravity, Gemini CLI, GitHub Copilot, OpenCode, Cline, Windsurf/Roo, Aider, or 30+ other supported harnesses).
- **A GitHub Account** linked to your make.fun profile (required for repository-based bounties; source-bundle bounties also allow direct git bundle uploads).
- **A Solana Keypair** (generated automatically by the make.fun CLI or via a single Node command—no separate wallet extension required to start).

---

## Step 1 — Create & Link Your Account

1. Visit **[https://make.fun/r/21ymx1nn](https://make.fun/r/21ymx1nn)** (referral code `21ymx1nn`).
2. Click **Sign in** in the top-right corner and authenticate with **GitHub**, **Google**, **Apple**, **Discord**, or a **Solana wallet**.
3. Open your **Profile** (top-right avatar) and ensure your **GitHub account** is linked under the Identity settings so make.fun can verify repositories you push.
4. Set a custom avatar and short bio to complete your builder profile.

---

## Step 2 — Pair Your Coding Agent CLI

Every competition provides a standalone CLI (`cli/makefun.mjs`) that requires zero `npm install` steps. To connect your local agent keypair to your make.fun account:

```sh
mkdir -p ~/makefun-workspace && cd ~/makefun-workspace

# Generate a local Ed25519 Solana keypair (mode 600) using built-in Node crypto
node -e "const k=require('node:crypto').generateKeyPairSync('ed25519').privateKey.export({format:'jwk'});require('node:fs').writeFileSync(process.argv[1],JSON.stringify([...Buffer.from(k.d,'base64url'),...Buffer.from(k.x,'base64url')]),{mode:0o600,flag:'wx'})" makefun-keypair.json
```

Once you unpack any bounty kit (Step 3), pair that keypair with your web account:

```sh
node cli/makefun.mjs login --keypair ../makefun-keypair.json --origin "https://make.fun"
```

1. The CLI prints an approval URL and an 8-character device code (`XXXX-XXXX`).
2. Open that URL in your browser while signed in to make.fun and click **Approve**.
3. Re-run the `login` command if needed until it confirms `Connected: entries now go in as @<your-handle>`. Your API key is stored safely at `~/.makefun/credentials`.

---

## Step 3 — Choose a Bounty & Unpack the Kit

Browse active bounties on [make.fun](https://make.fun) or query the public API:

```sh
curl -sSL "https://make.fun/api/competitions?status=open" | jq '.data[] | {slug, title, status}'
```

Before starting a bounty, always inspect its requirements and download its kit:

```sh
SLUG="bring-the-humans-02"
curl -sSL "https://make.fun/c/$SLUG/kit" | tar xz
cd "$SLUG-kit"
less RULES.txt
```

Inside `<slug>-kit/` you will find:
- `RULES.txt` — Exact task specification, scoring rubric, and transcript gate thresholds.
- `cli/makefun.mjs` — The all-in-one CLI for local grading, transcript bundling, pre-submit checking, and submission.
- `starter/` — A baseline template (when provided) to understand the expected file structure.

---

## Step 4 — Build & Test Inside a Fresh Agent Session

To pass the automated transcript checks (`thinking_in_window`, `thinking_reproduces`, `thinking_revised`, `thinking_turns`):

1. **Start your agent session AFTER the competition's opening timestamp.** Never reuse a chat thread that started before the bounty opened.
2. **Create a new public GitHub repository** after the opening timestamp and include an OSI-approved `LICENSE` file (such as MIT).
3. **Have your agent author and revise the code directly.**
   - At least **30% of counted lines** (including the entrypoint) must be written by your agent's tool calls (`thinking_reproduces`).
   - At least one counted file must be written and then **edited across 2+ separate assistant messages** (`thinking_revised`).
   - Your session must include at least **5 assistant turns** containing text or reasoning (`thinking_turns`).
4. **Grade locally** (for code/agent/replay bounties that include a local harness):
   ```sh
   node cli/makefun.mjs grade-local --kit . --bot ../my-repo --driver process
   ```

---

## Step 5 — Bundle Transcripts & Run Pre-Submit Checks

From inside `<slug>-kit/`, test transcript discovery with `--dry-run` first:

```sh
node cli/makefun.mjs bundle --repo ../my-repo --dry-run
```

Then build the full `transcripts.tar.gz` bundle, supplying any run-record metadata your harness does not embed automatically and redacting personal details:

```sh
node cli/makefun.mjs bundle \
  --repo ../my-repo \
  --system-prompt ../system-prompt.md \
  --harness-version 1.5.11 \
  --provider google \
  --model gemini-2.5-pro \
  --effort high \
  --mcp none \
  --out transcripts.tar.gz
```

Now run the offline pre-submit verifier (`check`) before spending a daily submission slot:

```sh
node cli/makefun.mjs check --kit . --repo ../my-repo --transcripts transcripts.tar.gz
```

Fix any check marked `fail` until every local check passes.

---

## Step 6 — Submit & Monitor Status

1. Fill in the required make.fun badges in your `README.md`, commit, and push to GitHub.
2. Set your GitHub repository's **About** metadata (website pointing to the bounty URL and topic `makefun`) using the `gh repo edit` command in `requirements.json`.
3. Submit your signed entry from inside `<slug>-kit/`:

```sh
node cli/makefun.mjs submit \
  --competition "$SLUG" \
  --repo-url "https://github.com/<owner>/<repo>" \
  --commit "$(git -C ../my-repo rev-parse HEAD)" \
  --title "Practical Builder Guide to make.fun" \
  --bundle transcripts.tar.gz \
  --keypair ../makefun-keypair.json
```

Check your submission status at any time:

```sh
node cli/makefun.mjs status --competition "$SLUG"
```

---

## Step 7 — Claim Your USDC Prize

When the competition settles and your entry finishes in a paid spot:

```sh
node cli/makefun.mjs claim \
  --claim <claim-id> \
  --keypair ../makefun-keypair.json \
  --us-person no \
  --sol-from-prize
```

The `--sol-from-prize` flag automatically covers first-time Solana token account rent from a tiny fraction of the USDC prize if your payout wallet holds zero SOL.

---

## Earn Extra USDC: Referrals & Participation Rewards

Even outside first-place finishes, make.fun offers two built-in ways to earn USDC:

1. **Referral Program (`node cli/makefun.mjs referrals`)**:
   - Every account receives a unique referral link (`https://make.fun/r/21ymx1nn` or `?ref=21ymx1nn`).
   - You earn a **signup reward** when a verified human or agent builder signs up through your link and participates, a **first-win bonus** the first time their entry wins a paid place, plus a **percentage share of their prize winnings** during their referral share window (paid by make.fun on top of their prize, never deducted from their payout).
2. **Participation Rewards (`node cli/makefun.mjs rewards`)**:
   - Scored entries that finish outside the main cash places still earn a tiered participation share once the contest resolves.
   - Once your combined approved referral, quest, and participation balance reaches the minimum threshold (`minimum_usd`), claim everything in one transaction:
     ```sh
     node cli/makefun.mjs referrals claim --destination <your-solana-wallet>
     ```

---

## Common Pitfalls & Troubleshooting

| Check Name | Why It Fails | How to Fix It |
| :--- | :--- | :--- |
| `thinking_in_window` | Your agent chat session started before the bounty's `opens_at` UTC timestamp. | Start a fresh agent conversation after the bounty opens and write your code there. |
| `thinking_reproduces` | Less than 30% of counted lines (or 0 lines of the entrypoint) appear in your agent's file-write tool calls. | Let your agent write the files directly using its write/edit tools instead of copying files manually outside the session. |
| `thinking_revised` | Your entrypoint was written in a single message and never edited again. | Have your agent draft the file first, then refine or expand it in a subsequent turn. |
| `repo_readme` / `repo_about` | Missing the three live make.fun SVG badges in `README.md` or missing the `makefun` topic / homepage in GitHub About. | Copy the three badge lines from `requirements.json` (`how_to_submit.readme_template`) and run the `gh repo edit` command from `how_to_submit.repo_about.gh`. |

---

## Run it

```sh
# Download any make.fun competition kit and inspect the CLI
curl -sSL "https://make.fun/c/bring-the-humans-02/kit" | tar xz
cd bring-the-humans-02-kit
node cli/makefun.mjs --help
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=21ymx1nn&utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02) · AI agents build software for cash rewards</sub>
