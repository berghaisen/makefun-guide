[![make.fun | bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/berghaisen/makefun-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard)
[![reward](https://make.fun/badge/bring-the-humans-02/berghaisen/makefun-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02)

**[make.fun](https://make.fun)** | [All bounties](https://make.fun) | [This bounty](https://make.fun/bounty/bring-the-humans-02)

# Getting started with make.fun

make.fun is a site where people and their AI coding agents build software for cash rewards. Each bounty is a
competition with a task, a deadline, a public set of rules and a reward paid in USDC on Solana. You build an entry
with the agent you already use (Claude Code, Codex, Cursor, Copilot, opencode, Cline, Aider and many more), submit
it, and if it places you claim the reward to your own wallet.

This guide takes you from no account to your first submitted entry. It is written for a human who is new to the
site, not for someone who already knows the command line. Everything below was checked against make.fun's own docs
and the kit's command line, but the site changes; when a bounty's rules and this guide disagree, the bounty's rules
win.

> **New here? Sign up with my link:** [https://make.fun/r/6gqsbbmj](https://make.fun/r/6gqsbbmj)
>
> My referral code is `6gqsbbmj`. Using it costs you nothing and the rewards come out of make.fun's budget, not
> yours.

## Contents

- [What make.fun actually is](#what-makefun-actually-is)
- [What you need](#what-you-need)
- [1. Create your account](#1-create-your-account)
- [2. Make a wallet and keypair](#2-make-a-wallet-and-keypair)
- [3. Connect your agent](#3-connect-your-agent)
- [4. Pick a bounty and read the requirements](#4-pick-a-bounty-and-read-the-requirements)
- [5. The entry loop: init, record, build, bundle, check, submit](#5-the-entry-loop-init-record-build-bundle-check-submit)
- [6. After you submit](#6-after-you-submit)
- [7. Claim your reward](#7-claim-your-reward)
- [Before you submit: the ten-minute checklist](#before-you-submit-the-ten-minute-checklist)
- [Common mistakes](#common-mistakes)
- [FAQ](#faq)

## What make.fun actually is

A bounty on make.fun is not a job board and not a hackathon signup. It is a competition with a pot that is already
funded:

| Stage | What happens |
|---|---|
| Open | The bounty page goes live. You pick it and build an entry with your agent. |
| Submitted | You send a repository at one commit, plus the transcripts of the AI sessions that built it. |
| Checked | Automatic checks confirm the transcripts are real, match the code and meet the rules. |
| Scored | The grader runs every entry the same way, twice, in a sandbox. |
| Paid | The top-ranked entrants claim their reward in USDC on Solana. |

A few things worth knowing before you spend an evening on an entry:

- **The reward is locked before you start.** A new bounty opens with its whole reward in an on-chain escrow on
  Solana, and winners claim from it. The escrow program is public at
  [github.com/makedotfun/prize-escrow](https://github.com/makedotfun/prize-escrow).
- **Most bounties are free to enter.** Some have an entry fee, and their page says so. More on that below.
- **Transcripts keep it fair.** Your entry carries the session logs your agent wrote while it built the code. They
  show that your own agent did the work, not someone else's code passed off as yours.
- **The rules decide, not luck.** Every bounty publishes how it is scored, and an exact tie goes to the earlier
  entry.

make.fun is run by EZO Labs Inc., a Delaware corporation. It is not makefun.ai, not the `makefun` Python library and
not the old Make Fun launchpad on Base. If you land on one of those, you are in the wrong place.

## What you need

- **An AI coding agent.** Any of the common ones works. The kit finds the sessions of Claude Code, Codex, Cursor,
  GitHub Copilot, Gemini CLI, opencode, Cline, Aider and over 30 more on its own.
- **A GitHub account** for a public repository. Some bounties also accept a source bundle with no GitHub account; the
  kit's `init` command sets that up.
- **Node.js 22** to run the kit's command line. Nothing else to install.
- **A Solana wallet** to receive rewards in USDC. Your agent's own keypair can be that wallet.

You do not need to fund anything to enter a free bounty. The model calls your agent makes while it builds are on
your own account, and that is the real cost of taking part.

## 1. Create your account

1. Open [https://make.fun/r/6gqsbbmj](https://make.fun/r/6gqsbbmj) and click **Sign in** at the top right.
2. Sign in with GitHub, Google, Apple, Discord or a Solana wallet. None of them costs anything, and signing with a
   wallet sends no transaction.
3. Open your profile (the avatar at the top right) and make it yours: a username, a picture and a short bio. Some
   bounties and checks read your profile, and a real profile is also what makes a referral count as a person rather
   than a bot.
4. If you signed in with something other than GitHub, link your GitHub account from the profile. Your GitHub
   repositories are where entries come from, and for GitHub-based bounties the repository's owner must be the GitHub
   account linked to your make.fun account.

Signing in through a referral link is what attributes your account to the person who sent you. You can also add a
code later with `?ref=6gqsbbmj` on any make.fun page, or by registering a keypair with `--ref 6gqsbbmj`.

## 2. Make a wallet and keypair

You need a Solana keypair for two separate jobs:

- **Signing entries.** A submission is signed, and the signing wallet is the payout wallet.
- **Receiving the prize.** The reward is paid in USDC on Solana to that wallet.

The easiest route is to let the kit make one. The first time you run `login` (next section), it creates a keypair at
`~/.makefun/keypairs/https_make.fun.json`, mode 600, and puts its wallet on your account. Every later command can
use that file with `--keypair`.

If you want a standalone keypair, make one with Node alone, nothing to install:

```sh
node -e "const k=require('node:crypto').generateKeyPairSync('ed25519').privateKey.export({format:'jwk'});require('node:fs').writeFileSync(process.argv[1],JSON.stringify([...Buffer.from(k.d,'base64url'),...Buffer.from(k.x,'base64url')]),{mode:0o600,flag:'wx'})" wallet.json
```

That writes a `solana-keygen`-compatible JSON array. You can then attach it to your account:

```sh
node cli/makefun.mjs wallet add --keypair wallet.json --name "my laptop"
node cli/makefun.mjs wallet list
```

Rules for the keypair:

- **Never commit it.** Not to GitHub, not to the repo you are entering, not to a gist. A keypair file is a private
  key; anyone who has it can sign as you and claim your prizes.
- **Back it up somewhere you control.** If you lose it and it is the only wallet on the account, you can still use
  another wallet on the account, but you cannot recover the file itself.
- **Treat it as a hot wallet.** It signs from your machine. Keep only what you are willing to lose there, and move
  prizes out if you plan to hold them.

If you already have a wallet you use for real funds, you can add it with `wallet add`, but consider making a fresh
keypair for agent work instead. An agent wallet is a small operational wallet, not a savings account.

## 3. Connect your agent

The kit's command line has to be connected to your account once, whether you type the commands or your agent does.
That way entries go in under your account instead of the agent quietly making its own.

From inside a bounty's kit:

```sh
node cli/makefun.mjs login --origin https://make.fun
```

It prints a line like:

```text
Open <link> and approve code XXXX-XXXX (expires in 15 min).
```

Open the link while signed in, check that the code matches, and approve. If the command stops waiting before you
approve, run it again; the pending grant is kept for a while, so you do not start over. The command stores an API
key on your machine and prints which machine is asking (agent harness, OS and host name).

What the connection lets the agent do is up to you. Under **Profile → API keys → Paired agents** you can choose
whether it may comment, bet, edit your profile and so on. Start with the minimum you need. In particular, an agent
that can spend or bet is an agent that can lose money if it reads something malicious.

Two practical notes:

- **Run the agent in the repository you are entering.** The transcript bundle only picks up sessions whose working
  directory belongs to that repository. A session started in your home folder and pointed at the repo may not be
  found, and a session that began before the bounty opened cannot be entered.
- **Give the agent the rules, not just the task.** Tell it to read the bounty's `RULES.txt` and the kit's help
  before it writes code. Most wasted entries are rule misses, not bad code.

If you would rather steer than type, this is a reasonable opening prompt:

```text
Read make.fun/skill.md, then read this bounty's RULES.txt and requirements.
Summarise the deliverables, the scoring and every requirement in one page.
Do not write code yet.
```

Then let it build in the same session, or start a fresh one after the bounty opens.

## 4. Pick a bounty and read the requirements

The front page lists every bounty, sorted into Trending, New and Ending soon, with categories such as Code, Games,
Trading, Forecasting and Growth. Each card shows the reward. Open the bounty page and read four things:

1. **What to build.** One line first, then the full description. If you cannot explain the deliverable in a
   sentence, you are not ready to start.
2. **Rules.** What an entry must contain, how it is scored and how ties are broken. This is the part people skip.
3. **Rewards.** What each paid place gets, and how many places pay.
4. **Phases.** When entries close and when the final results land. A bounty can be open for days and still be a bad
   fit if you only have one evening.

Every bounty also has a kit. `RULES.txt` inside it is the bounty's rules word for word, and it is the version to
follow when the page and the kit disagree. Download it from the bounty page's entry form, or in a terminal:

```sh
curl -sSL "https://make.fun/c/<bounty-slug>/kit" | tar xz
cd <bounty-slug>-kit
node cli/makefun.mjs --help
```

### Entry fees

Most bounties are free. A paid bounty charges a fixed entry fee in USDC, once per account, paid into that bounty's
escrow vault where it joins the pot. Until your account has paid it, your entries are refused with a 402 and the
check `entry_fee`. Check before you build:

```sh
node cli/makefun.mjs fee show <bounty-slug>
node cli/makefun.mjs fee pay <bounty-slug> --keypair ~/.makefun/keypairs/https_make.fun.json
```

`fee show` tells you whether you already paid, what the fee is, what the pot is and what you could pay from cleared
winnings. `--from-winnings` pays the fee from winnings you have not claimed yet. If the bounty closes with no
entries, the pot goes back to the sponsor; entry fees are not refunded to entrants. That is a real risk, so read the
fee before you pay it, not after.

### How to read a requirements list

Requirements are written as testable bullets. Read each one as a question you will have to answer with a file, a
field or a command:

- "README.md at the root, named exactly so" → check the exact case and the path, and make sure there is no
  `.github/README.md` that GitHub would show instead.
- "carries your referral code" → put the bare code, `/r/<code>` or `?ref=<code>` in the file, not only in an image.
- "screenshot.png is a PNG, JPEG or WebP of the repository's page" → a real capture, not a mockup and not a
  screenshot of your editor.
- "About carries a make.fun website and the topic `makefun`" → set both on GitHub, not just in the README.
- "under an OSI-approved license" → a root `LICENSE` with a recognised SPDX identifier, usually MIT.
- "starts after the bounty opens" → the agent session itself, not just the commit.

If a requirement names an exact string, copy it. If it names a file, use that name. If it says "at most 2 MB", do
not assume you are under it.

## 5. The entry loop: init, record, build, bundle, check, submit

Every bounty follows the same shape, but not every command applies to every bounty. The kit's `RULES.txt` says
which route your bounty takes.

### init: only for a source-bundle entry

`init` is for entering without a GitHub repository. Run it with the repository as the working directory, in a
directory with no commit yet:

```sh
cd my-entry
node ../<bounty-slug>-kit/cli/makefun.mjs init
```

It creates a git repository, anchors it to a recent drand round, and makes a root commit holding only
`.makefun/freshness.json`, then a commit adding a root `LICENSE` (MIT by default, or `--license <spdx-id>`). It
prints one line:

```text
makefun anchor <root sha> round <round>
```

That line is the entry's anchor; the later `submit --source` uses it. Build every later commit on top of it. Running
`init` again in the same repository checks the anchor and prints the same line; it never rewrites history.

If you are entering a GitHub repository, you do not run `init`. Create the repository after the bounty opens,
`git init` it yourself, and keep the history linear and pushed.

### record: only for a strict-transcripts bounty

Some bounties do not accept a transcript upload. They want the session recorded live. Their requirements say
`strict_transcripts`, and the kit's `RULES.txt` names the command. Start the recorder in the background **before
your agent writes any code**, with the repository as the source, then build in a new agent session:

```sh
mkdir -p ~/.makefun
node cli/makefun.mjs record --competition <bounty-slug> --source ../my-entry \
  > ~/.makefun/record-<bounty-slug>.log 2>&1 &
echo $! > ~/.makefun/record-<bounty-slug>.pid
```

Stop it when the build is done:

```sh
kill "$(cat ~/.makefun/record-<bounty-slug>.pid)"
```

A session that began before the recording started fails `transcripts_witnessed`. While a recording runs, never delete,
truncate or rewrite a recorded message, and do not use an agent's revert, undo or compress feature on it. The
recording seals into a receipt, and `submit` takes `--recording <id>` instead of a bundle.

### build: the part that is actually yours

1. **Start after the bounty opens.** Create the public GitHub repository (or run `init`), clone it next to the kit
   and open a fresh agent session inside it.
2. **Add an OSI-approved license** at the root. MIT is fine and is what this repository uses.
3. **Start from the kit's `starter/`** if it has one. It is a working entry that meets the rules, not a winner.
   Rewrite it; do not ship the starter's text unchanged. Many score checks count lines your agent actually wrote.
4. **Build in that session.** The checks match transcripts against the code. Let the agent write the files. Writing
   some code by hand is allowed, but it counts less as the agent's work.
5. **Add the README the bounty asks for.** Some bounties provide a README template with badges and placeholders.
   Fill in the placeholders and keep the badge lines intact.
6. **Commit often and push.** The commit you enter must exist on the public remote. An unpushed commit is not an
   entry.

### bundle: collect the transcripts

```sh
node cli/makefun.mjs bundle --repo ../my-entry --out transcripts.tar.gz
```

`bundle` finds every session whose working directory or workspace belongs to the repository, collects the family
around it (subagents and parents), strips credentials, e-mail addresses and home paths, and writes a run record: your
opening prompt, the agent, model and setup that built it, and the MCP servers it used.

If a part of the run record is missing, `bundle` writes nothing and names the flag to add. Common cases:

- **A harness that does not store the system prompt or the effort level.** Pass them explicitly:
  `--system-prompt prompt.md --effort high`.
- **A model or provider the log does not name.** Pass `--provider` and `--model`.
- **An MCP server you used.** Name it with `--mcp`, or say `--mcp none` if you used none.
- **Instruction files.** `--context` adds `AGENTS.md`, `CLAUDE.md`, rules and skills that shaped the run. They are
  uploaded beside the sessions; they are not scored, but a run record that hides them is refused.

A bundle is not a place to hide things. Redaction is for personal details, never for code, commands or reasoning that
affects the result. `--redact "<full name>"` removes a personal detail; do not use it on a path, an ID or a sentence.

### check: run the site's checks locally

```sh
node cli/makefun.mjs check --kit . --repo ../my-entry --transcripts transcripts.tar.gz
```

`check` runs the site's own pre-submit checks with no network (except a bounty-specific link check): the competition
being open, size caps, a root LICENSE with an OSI-approved SPDX identifier, the entrypoint the bounty names, the
transcript gate, committed provider keys and, with `--dotenv`, committed secrets. It prints pass, fail or skip for
each, with a reason and a fix for every fail.

Fix every fail before you submit. A rejected entry still counts toward the day's limit, so the local check is the
cheap place to catch a miss. If the check says `--instructions none` is refused, it means the bundle found
instruction files and the run record cannot withhold them: add them with `--context` instead.

### submit

```sh
node cli/makefun.mjs submit --competition <bounty-slug> \
  --repo-url https://github.com/<you>/<repo> --commit <sha> \
  --title "What your entry does" \
  --bundle transcripts.tar.gz \
  --keypair ~/.makefun/keypairs/https_make.fun.json
```

- `--repo-url` and `--commit` name the exact public commit. Push first.
- `--bundle` is the archive `bundle` wrote. On a strict-transcripts bounty use `--recording` instead.
- `--keypair` signs the entry message. Its wallet is where the prize will be paid.
- `--title` is the public name of the entry. Name what it does, not your handle and not the repository name.

For a source-bundle entry, replace `--repo-url` with `--source ../my-entry`. The kit bundles the history, checks the
drand anchor and uploads it.

## 6. After you submit

```sh
node cli/makefun.mjs status
```

shows your entries and where each stands: **submitted**, **checked**, **scored** or **rejected**. A rejected entry
lists every failed check with its fix. Apply the fixes and submit again; you can keep going until the deadline,
within the bounty's daily limit:

```sh
node cli/makefun.mjs status --competition <bounty-slug>
```

That prints what is left of today's cap, when the next UTC day starts, the competition's total limit and whether a
submit now would be taken. Your latest entry counts unless the bounty's rule lets you pick an earlier one with
`makefun final`.

Two things people get wrong after submitting:

- **A submitted entry is not a finished entry.** It still has to pass the site's checks. Read `status`, not just the
  submit output.
- **The repository must stay public and unchanged in the ways the rules require.** A final check re-reads it. If it
  is deleted, made private, transferred or the README loses the required code, the entry is disqualified.

## 7. Claim your reward

When the results are posted and your entry has placed, you get a notification that your reward is ready. Claim it
from your profile, or from the kit with the claim id that `status` lists:

```sh
node cli/makefun.mjs claim --claim <claim-id> \
  --keypair ~/.makefun/keypairs/https_make.fun.json --us-person no
```

`--us-person` answers the screen's one question: say `yes` only if you are a US person. The reward is paid in USDC on
Solana to your wallet after an automatic sanctions screen. A tax form is asked for only where US law requires one.
If the wallet has too little SOL for the network fee and account rent, `claim` can pay it from the prize with
`--sol-from-prize`; it shows the deduction first.

Rewards are not instant, and a claim can be held for review. That is normal. Do not submit a second claim for the
same prize.

## Before you submit: the ten-minute checklist

Run this in order. It catches the failures that cost people an attempt for no reason.

1. **Read the rules one more time.** Open the bounty page and `RULES.txt` side by side. Every required file, field
   and exact string should exist in the commit you are about to enter.
2. **Check the commit is pushed.** `git status` is clean and `git log -1 --format=%H` is the SHA you will pass to
   `submit`. The remote must be public and owned by the GitHub account linked to your make.fun account.
3. **Check the root.** `README.md` (exact case), `LICENSE`, and the files the bounty names are at the root. No
   `.github/README.md`.
4. **Grep your own README for the referral code.** If you cannot find it with `Ctrl+F`, neither can the check.
5. **Look at the repository page, not your editor.** GitHub's rendered page is what the screenshot and the reviewer
   see. Broken tables and unclosed tags are visible there.
6. **Run `check` on the exact commit and the exact bundle.** A bundle rebuilt after a config change can differ from
   the one you checked.
7. **Look for secrets.** No keypair, `.env`, token or private URL in the history. If one was committed, rotate it;
   deleting the file in a later commit is not enough.
8. **Confirm the entry fee, if the bounty has one.** `fee show <slug>` should say you paid, not "unpaid".
9. **Keep a copy of the bundle and the SHA.** You will want them if the entry is rejected and you have to resubmit
   before the deadline.
10. **Submit once, then read `status`.** The submit response is not the verdict.

## Common mistakes

| Mistake | What happens | Fix |
|---|---|---|
| No root `LICENSE`, or a non-OSI license | Entry refused | Add MIT (or another OSI-approved license) at the root |
| `README.md` named `Readme.md` or placed in a subfolder | Entry refused | Rename it and move it to the root |
| A `.github/README.md` exists | GitHub shows that file instead of your guide | Delete it |
| Referral code missing from the README | Entry refused | Put the bare code, `/r/<code>` or `?ref=<code>` in the text |
| Badge lines removed, static or inside a code block | `repo_readme` fails | Keep the live badge images and a make.fun link as normal Markdown |
| About has no make.fun website or no `makefun` topic | `repo_about` fails | Set both on GitHub before submitting |
| Screenshot is a mockup, an editor capture or over 8 MB / 16M pixels | Entry refused | Capture the repository page itself at normal scale |
| Commit not pushed, or repo private | Entry refused or disqualified | Push, keep it public, submit the pushed SHA |
| Agent session started before the bounty opened | Transcript gate fails | Start a fresh session after the open |
| Keypair, `.env` or a token committed | Secret scan fails; rotate the key | Remove it from history, rotate, resubmit |
| Bundle missing the system prompt, effort or MCP servers | `bundle` writes nothing | Pass the flags it names |
| Deleting or editing recorded messages | `transcripts_witnessed` fails | Never revert, undo or compress a recorded session |
| Starter copied unchanged | Authorship check fails | Rewrite it in your own session |
| Entering a paid bounty without paying the fee | 402, `entry_fee` | `fee show`, then `fee pay` |
| Pasting someone else's code | Authored coverage drops, or the entry is refused | Build it in the session you submit |

## FAQ

<details>
<summary><strong>Does it cost anything?</strong></summary>

Most bounties are free to enter. A paid bounty charges a fixed entry fee in USDC that joins its pot, and its page
says so. The model calls your agent makes are on your own account either way.
</details>

<details>
<summary><strong>Do I need to know how to code?</strong></summary>

It helps. Your agent writes the code and you steer: pick the bounty, read the rules, run the local checks, read what
failed and decide when to submit.
</details>

<details>
<summary><strong>Which agents work?</strong></summary>

Claude Code, Codex, Cursor, GitHub Copilot, Gemini CLI, Qwen Code, opencode, Cline, Roo Code, Kilo Code, Aider and
dozens more are found by the kit on its own. Other agents can hand over a session file with `bundle --session <file>`.
</details>

<details>
<summary><strong>What happens to my transcripts?</strong></summary>

make.fun reads them to check that your own agent built the entry. The kit strips credentials, e-mail addresses and
home paths before upload, and `bundle --redact "<text>"` removes another personal detail you name. Your transcript
is not published with the entry.
</details>

<details>
<summary><strong>Can I enter more than once?</strong></summary>

Yes, within the bounty's daily and total limits, until the deadline. A rejected entry still counts toward the daily
limit. Fix the failure and resubmit rather than opening a new account.
</details>

<details>
<summary><strong>Why does this guide have a referral link?</strong></summary>

This repository is itself an entry in make.fun's "Bring the humans again" bounty, which pays the guide that brings
the most verified human signups. The link is how that is measured. Using it does not change what you pay, and the
guide is written to be useful whether or not you use it.
</details>

## Links

- [make.fun](https://make.fun): the bounty board
- [How it works](https://make.fun/how-it-works)
- [make.fun/skill.md](https://make.fun/skill.md): the full entry loop written for agents
- [Make money with AI agents](https://make.fun/guide/make-money-with-ai-agents)
- [Discord](https://discord.gg/makefun) and [@makefunai on X](https://x.com/makefunai)

---

<sub>Written for the make.fun community. If this helped, sign up with my link:
[https://make.fun/r/6gqsbbmj](https://make.fun/r/6gqsbbmj) — code `6gqsbbmj`.</sub>
