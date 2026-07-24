# 🧪 AI Agent Workflow Lab

> A checkpoint-based guide for building a safe, testable issue → branch → implementation → review → pull-request workflow before applying it to a real project.

## Target outcome

By the end of this guide, you will be able to:

1. Define one shared set of repository instructions that both Codex and GitHub Copilot can use.
2. Convert a small idea into a documented specification and GitHub issue.
3. Give a bounded issue to an AI coding agent.
4. Have the agent create an isolated branch, implement the change, run tests, and open a pull request.
5. Review and merge the pull request yourself.
6. Run the same workflow through Sandcastle in an isolated sandbox.
7. Start or supervise approved work remotely from a phone through Codex Remote or Hermes.

This guide deliberately starts with a tiny TypeScript repository. Do not use a client project until the lab has completed at least ten clean issue-to-PR runs.

---

# 0. Recommended architecture

Use the tools as separate layers instead of forcing one product to do everything.

```text
Repository knowledge
AGENTS.md + CONTEXT.md + specs + tests
        ↓
Reusable procedures
Agent Skills
        ↓
Interactive development
VS Code Copilot or Codex
        ↓
Durable work queue
GitHub Issues
        ↓
Isolated autonomous execution
Sandcastle + Docker + Codex
        ↓
Review boundary
GitHub Pull Request + CI + human approval
        ↓
Optional remote control
Codex mobile OR Hermes + Discord
        ↓
Optional private networking
Tailscale
```

## The most important compatibility rule

Use **`AGENTS.md` at the repository root as the canonical project instruction file**.

- Codex reads `AGENTS.md` automatically.
- VS Code Copilot also supports `AGENTS.md`.
- `.github/copilot-instructions.md` is Copilot-specific. Do not assume Codex will load it.
- `.github/agents/*.agent.md` defines VS Code Copilot custom agents. Codex does not automatically use those agents.
- `.agents/skills/*/SKILL.md` is the best location for portable Agent Skills when the installer supports it.
- Install Matt Pocock's skills for both Codex and Copilot when the installer asks which agents should receive them.

Keep shared rules in one place. Add provider-specific files only when a provider genuinely needs extra behavior.

---

# 1. Cost and time summary

## Required software cost

| Item | Cost | Required now? |
|---|---:|---:|
| VS Code | $0 | Yes |
| Git and GitHub CLI | $0 | Yes |
| GitHub Free private repository | $0 | Yes |
| Node.js, pnpm, TypeScript, Vitest | $0 | Yes |
| Matt Pocock Skills | $0 | Milestone 5 |
| Sandcastle | $0, open source | Milestone 8 |
| Docker Desktop | $0 for personal use and qualifying small businesses | Milestone 8 |
| Tailscale Personal | $0 | Optional |
| Hermes Agent | $0, open source | Optional |
| ChatGPT Plus with Codex | $20/month, already owned | Recommended |
| GitHub Copilot | Existing plan | Recommended for VS Code custom-agent testing |

Current individual Copilot list prices:

- Copilot Pro: $10/month
- Copilot Pro+: $39/month
- Copilot Max: $100/month

## Optional hosting cost

| Host choice | Expected cost | Best use |
|---|---:|---|
| Your current PC | $0 additional | First lab and supervised runs |
| Existing spare PC | $0 additional | Always-on local worker |
| Mac mini | Starts around $799 new | Dedicated local worker; not needed for this lab |
| Small VPS | Roughly $5–$24/month | Always-on Linux worker without buying hardware |
| Managed cloud sandbox | Usage-based | Disposable jobs and stronger isolation |

Model usage can consume subscription limits or purchased credits. The software may be free while the model inference is not.

## Estimated setup time

| Milestone | Time |
|---|---:|
| 1. Verify prerequisites | 30–60 minutes |
| 2. Create the lab repository | 20–30 minutes |
| 3. Add the project brain | 20–30 minutes |
| 4. Verify Codex and Copilot separately | 30–45 minutes |
| 5. Install and test Agent Skills | 45–90 minutes |
| 6. Run the manual issue-to-PR workflow | 30–45 minutes |
| 7. Create the VS Code baby loop | 45–90 minutes |
| 8. Add Sandcastle and Docker | 60–120 minutes |
| 9. Add a bounded autonomous loop | 60–90 minutes |
| 10–13. Add AFK control, hosting, and CI | 1–4 hours, depending on option |

Expected time to reach the first safe automated pull request: **about 5–8 focused hours**.

---

# Milestone 1 — Verify the workstation

## Purpose

Confirm that your existing Windows, WSL, VS Code, Git, Node, GitHub, Codex, and Copilot setup is healthy before adding orchestration. This avoids debugging five tools at once later.

## Recommended environment choice

Use this split:

- **VS Code on Windows** as the interface.
- **WSL 2 Ubuntu** as the development environment.
- Store the lab repository inside the WSL filesystem, such as `~/dev/agent-workflow-lab`.
- Use Docker Desktop's WSL integration later.
- Keep Hermes on Windows only for the first remote-control test, because it is already configured there.

Avoid storing the Sandcastle lab under `/mnt/c/...`; Linux tooling, file watching, permissions, and Docker volume access are generally cleaner under `~/dev/...`.

## Required versions

- Windows 11 with WSL 2
- VS Code Stable, current version
- Git 2.x
- Node.js **24 LTS**
- pnpm **10.x**
- TypeScript **^6.0.3** inside the project
- GitHub CLI current version
- Docker Desktop current version, added later

## Official links

- VS Code: https://code.visualstudio.com/
- WSL: https://learn.microsoft.com/windows/wsl/install
- Git: https://git-scm.com/download/win
- Node.js: https://nodejs.org/en/download
- pnpm: https://pnpm.io/installation
- GitHub CLI: https://cli.github.com/
- Codex: https://developers.openai.com/codex/
- GitHub Copilot in VS Code: https://code.visualstudio.com/docs/copilot/overview

## Step 1. Check WSL

Run from PowerShell or Windows Terminal:

```powershell
wsl --status
wsl --list --verbose
```

Expected result: Ubuntu is listed with `VERSION 2`.

When WSL is missing:

```powershell
wsl --install -d Ubuntu
```

Restart Windows if requested.

## Step 2. Open Ubuntu and verify the core tools

```bash
node --version
npm --version
pnpm --version
git --version
gh --version
code --version
```

Expected minimum:

```text
node: v24.x
pnpm: 10.x
git: 2.x
gh: installed
code: installed
```

## Step 3. Install or correct Node and pnpm only when needed

The cleanest option in WSL is a Node version manager. Since you already work with Node, keep your existing version manager if it is functioning.

When Node is missing, install Node 24 using your preferred version manager. Then install pnpm 10:

```bash
npm install --global pnpm@10
```

Verify again:

```bash
node --version
pnpm --version
```

## Step 4. Authenticate GitHub CLI

```bash
gh auth login
```

Choose:

1. `GitHub.com`
2. `HTTPS`
3. Authenticate through the browser
4. Allow GitHub CLI to authenticate Git operations when asked

Verify:

```bash
gh auth status
```

Do not store a personal access token inside the repository.

## Step 5. Verify the Codex CLI or IDE extension

The Codex IDE extension is enough for interactive testing. Install the CLI too because Sandcastle and Hermes can use it later:

```bash
npm install --global @openai/codex
codex --version
codex
```

Complete the ChatGPT device-login flow. Your ChatGPT Plus plan includes the Codex model family and IDE/CLI access, subject to plan usage limits.

## Step 6. Verify VS Code extensions

Inside VS Code, confirm:

- GitHub Copilot is signed in.
- GitHub Copilot Chat opens in Agent mode.
- Codex is signed in with your ChatGPT account.
- The WSL extension is installed.

Open the WSL workspace with:

```bash
code .
```

## Completion check

Do not continue until all commands succeed:

```bash
node --version
pnpm --version
git --version
gh auth status
codex --version
```

**Checkpoint:** You can open a WSL folder in VS Code and use both Copilot Chat and Codex.

---

# Milestone 2 — Create the isolated test repository

## Purpose

Create the smallest useful TypeScript project that supports implementation, tests, Git branches, GitHub issues, and pull requests. The code is intentionally trivial so failures belong to the workflow rather than the application.

## Repository behavior

The repository will expose one function:

```ts
incrementCounter(0); // 1
```

Later issues will add one documentation file, one feature, and one bug fix.

## Step 1. Create the repository locally

Run inside WSL:

```bash
mkdir --parents ~/dev/agent-workflow-lab
cd ~/dev/agent-workflow-lab
git init --initial-branch=main
pnpm init
```

## Step 2. Install project dependencies

```bash
pnpm add --save-dev \
  typescript@^6.0.3 \
  vitest@^3.2.0 \
  @types/node@^24.0.0
```

Record the exact pnpm version in `package.json`:

```bash
pnpm pkg set "packageManager=pnpm@$(pnpm --version)"
pnpm pkg set "type=module"
pnpm pkg set "private=true" --json
pnpm pkg set "scripts.test=vitest run"
pnpm pkg set "scripts.typecheck=tsc --noEmit"
pnpm pkg set "scripts.check=pnpm typecheck && pnpm test"
```

## Step 3. Create `.gitignore`

```bash
cat > .gitignore <<'EOF'
node_modules/
coverage/
dist/
.env
.env.*
!.env.example
.sandcastle/.env
*.log
EOF
```

## Step 4. Create `tsconfig.json`

```bash
cat > tsconfig.json <<'EOF'
{
  "compilerOptions": {
    "target": "ES2024",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "skipLibCheck": true,
    "types": ["node", "vitest/globals"]
  },
  "include": ["src/**/*.ts"]
}
EOF
```

## Step 5. Create the initial source and test

```bash
mkdir --parents src

cat > src/counter.ts <<'EOF'
export function incrementCounter(value: number): number {
  return value + 1;
}
EOF

cat > src/counter.test.ts <<'EOF'
import { describe, expect, it } from "vitest";

import { incrementCounter } from "./counter";

describe("incrementCounter", () => {
  it("increments an integer by one", () => {
    expect(incrementCounter(0)).toBe(1);
  });
});
EOF
```

## Step 6. Verify the project

```bash
pnpm check
```

Expected result:

- TypeScript passes.
- One Vitest test passes.

## Step 7. Create the initial commit

```bash
git add .
git commit -m "chore: initialize agent workflow lab"
```

## Step 8. Create a private GitHub repository

```bash
gh repo create agent-workflow-lab \
  --private \
  --source=. \
  --remote=origin \
  --push
```

Verify:

```bash
git status
gh repo view --web
```

Expected Git status:

```text
nothing to commit, working tree clean
```

## Completion check

**Checkpoint:** The private GitHub repository exists, `main` is pushed, and `pnpm check` passes.

---

# Milestone 3 — Create the shared project brain

## Purpose

Give every compatible agent the same rules, vocabulary, commands, boundaries, and definition of done. This is the foundation that prevents Sol from improvising a large architecture for a tiny task.

## File responsibilities

```text
AGENTS.md       Shared operating rules for Codex and Copilot
CONTEXT.md      Domain meaning and project intent
docs/specs/     Approved behavior and acceptance criteria
docs/adr/       Important architecture decisions
```

Do not duplicate all rules in both `AGENTS.md` and `.github/copilot-instructions.md`.

## Step 1. Create `AGENTS.md`

```bash
cat > AGENTS.md <<'EOF'
# Agent Instructions

## Project

This repository is an isolated TypeScript lab for testing AI issue-to-PR workflows.
Keep every change intentionally small.

## Commands

- Install dependencies: `pnpm install`
- Run all required checks: `pnpm check`
- Run tests: `pnpm test`
- Run type checking: `pnpm typecheck`

## Coding rules

- Use TypeScript with strict types.
- Keep each function focused on one responsibility.
- Reuse existing helpers before adding new ones.
- Use `for...of` when iteration is necessary.
- Do not use `Array.prototype.forEach`.
- Do not use classic `for (let i = 0; ...)` loops.
- Do not add a dependency without human approval.
- Do not modify unrelated files.
- Preserve working logic unless the issue explicitly requires a change.

## Git rules

- Work on one issue per branch.
- Use `feature/issue-<number>-<slug>` for features.
- Use `fix/issue-<number>-<slug>` for bug fixes.
- Use Conventional Commit messages.
- Never merge into `main`.
- Never force-push.
- Open a pull request against `main`.
- A human must approve and merge every pull request.

## Definition of done

A task is complete only when:

1. The issue acceptance criteria are satisfied.
2. Focused tests cover changed behavior.
3. `pnpm check` passes.
4. The diff contains no unrelated changes.
5. The branch is pushed.
6. A pull request links the issue.
7. The final report lists changed files and commands run.

## Stop conditions

Stop and request human input when:

- Requirements conflict or are ambiguous.
- A new dependency appears necessary.
- A destructive command appears necessary.
- Tests fail for reasons outside the issue scope.
- More than three implementation attempts fail.
- The requested work would modify authentication, secrets, deployment, billing, or production data.
EOF
```

## Step 2. Create `CONTEXT.md`

```bash
cat > CONTEXT.md <<'EOF'
# Project Context

## Purpose

This repository exists only to verify an AI-assisted development workflow.
It is not a real product and must remain intentionally small.

## Domain vocabulary

- Counter: an integer value.
- Increment: add one to a counter.
- Decrement: subtract one from a counter.
- Valid counter input: a finite integer.

## Product boundaries

- No UI.
- No database.
- No network requests.
- No runtime dependencies.
- No framework.
- No deployment.

## Quality target

Prefer the smallest implementation that clearly satisfies the approved issue.
EOF
```

## Step 3. Create documentation directories

```bash
mkdir --parents docs/specs docs/adr

touch docs/specs/.gitkeep
touch docs/adr/.gitkeep
```

## Step 4. Commit the project brain

```bash
git add AGENTS.md CONTEXT.md docs
git commit -m "docs: add shared agent instructions and context"
git push
```

## Optional Copilot-specific file

Do not add this until you find behavior that is genuinely specific to Copilot. When needed, keep it tiny:

```md
# Copilot Integration Notes

Follow the repository's root `AGENTS.md` as the canonical instruction source.
When completing a task, report changed files, checks run, and the issue or pull-request URL.
```

Store it at:

```text
.github/copilot-instructions.md
```

## Completion check

Ask both Codex and Copilot this read-only question:

```text
Without editing anything, summarize the repository's required test command,
branch naming rules, forbidden loop styles, and merge restriction.
```

Both should answer from `AGENTS.md`.

**Checkpoint:** Codex and Copilot independently recognize the same project rules.

---

# Milestone 4 — Test Codex and Copilot separately

## Purpose

Confirm each harness can read instructions, edit files, run checks, and respect Git boundaries before trying orchestration. Do not compare model intelligence yet; verify mechanics.

## Model recommendation

For this lab:

- **Codex:** start with GPT-5.6 Sol using Medium or High reasoning, Fast mode off.
- **Copilot:** use its automatic model or Sol when available.
- Later, use Terra for routine implementation and Sol for planning, difficult work, or final review.
- Do not use Ultra or unlimited subagents in this lab.

## Test A — Codex read-only check

Open Codex in the repository and ask:

```text
Read AGENTS.md and CONTEXT.md. Do not edit files or run destructive commands.
Explain what this repository is for, the command that proves a change is valid,
and which actions require human approval.
```

Pass condition: it identifies `pnpm check` and human-only merging.

## Test B — Copilot read-only check

Open Copilot Chat in Ask or Plan mode and submit the same prompt.

Pass condition: the answer matches the shared rules.

## Test C — Controlled local edit with Codex

Create a temporary branch yourself:

```bash
git switch -c test/codex-readme
```

Prompt Codex:

```text
Create README.md with a short title, purpose, installation command, and test command.
Do not commit or push. Run pnpm check after editing.
```

Inspect:

```bash
git diff
git status
pnpm check
```

When correct:

```bash
git add README.md
git commit -m "docs: add lab readme"
git switch main
git merge --ff-only test/codex-readme
git branch --delete test/codex-readme
git push
```

## Test D — Controlled local edit with Copilot

Create another temporary branch:

```bash
git switch -c test/copilot-note
```

Prompt Copilot Agent:

```text
Create docs/WORKFLOW.md containing a five-line summary of the intended
issue-to-branch-to-PR workflow. Do not commit or push. Run pnpm check.
```

Review the Chat Keep/Undo controls, choose **Keep**, inspect the diff, then commit manually:

```bash
git add docs/WORKFLOW.md
git commit -m "docs: summarize lab workflow"
git switch main
git merge --ff-only test/copilot-note
git branch --delete test/copilot-note
git push
```

## Completion check

**Checkpoint:** Both Codex and Copilot can make a bounded change and pass `pnpm check` without violating the merge rule.

---

# Milestone 5 — Install Matt Pocock's Agent Skills

## Purpose

Add reusable procedures for questioning, specifications, ticket creation, implementation, test-driven development, and review. Skills define repeatable work; they do not replace `AGENTS.md`.

## Official repository

https://github.com/mattpocock/skills

## Step 1. Install the skills

Run from the repository root:

```bash
npx skills@latest add mattpocock/skills
```

When prompted:

1. Select both Codex and GitHub Copilot/VS Code when available.
2. Install `/setup-matt-pocock-skills`.
3. Also select these initial skills:
   - `/grill-with-docs`
   - `/to-spec`
   - `/to-tickets`
   - `/implement`
   - `/tdd`
   - `/code-review`
   - `/diagnosing-bugs`
   - `/wayfinder`

Do not install everything merely because it exists. Start with the workflow above.

## Step 2. Run the setup skill

Inside a compatible agent, run:

```text
/setup-matt-pocock-skills
```

Choose:

- Issue tracker: GitHub Issues
- Documentation directory: `docs`
- Triage label: `agent-ready`
- Human clarification label: `needs-human`

If a slash command does not appear, tell the agent explicitly:

```text
Use the setup-matt-pocock-skills Agent Skill installed in this repository.
```

## Step 3. Inspect what was installed

Look for skill directories such as:

```text
.agents/skills/
.github/skills/
.claude/skills/
```

Each skill should contain `SKILL.md`. Keep the location generated by the installer; do not manually copy the same skill into every supported folder.

## Step 4. Run a tiny grilling exercise

Use `/grill-with-docs` with this idea:

```text
I want to add decrementCounter(value), which subtracts one from a valid integer.
The repository must remain dependency-free at runtime.
```

Answer its questions. Require it to save the result under:

```text
docs/specs/decrement-counter.md
```

The approved specification should include:

- Input and output behavior
- Integer requirement
- Error behavior for invalid values
- Tests
- Explicit non-goals

## Step 5. Convert the spec into tickets

Run:

```text
/to-tickets docs/specs/decrement-counter.md
```

For this lab, require one vertical ticket rather than unnecessary decomposition.

## Completion check

Verify:

```bash
gh issue list
find .agents .github -name SKILL.md 2>/dev/null
```

**Checkpoint:** A documented feature specification exists and at least one GitHub issue was created from it.

---

# Milestone 6 — Run the supervised issue-to-PR workflow

## Purpose

Prove the complete Git workflow manually before automating it. The agent may implement and open a PR, but you control when it starts and whether it merges.

## Step 1. Create workflow labels once

```bash
gh label create agent-ready \
  --color 0E8A16 \
  --description "Approved for autonomous implementation" \
  --force

gh label create needs-human \
  --color FBCA04 \
  --description "Blocked on a human decision" \
  --force

gh label create agent-done \
  --color 5319E7 \
  --description "Agent completed work and opened a PR" \
  --force

gh label create feature \
  --color A2EEEF \
  --description "New behavior" \
  --force

gh label create bug \
  --color D73A4A \
  --description "Incorrect behavior" \
  --force
```

## Step 2. Create three deliberately small issues

### Issue 1 — Trivial documentation

```bash
gh issue create \
  --title "docs: add the word one" \
  --label "agent-ready,feature" \
  --body $'Create `docs/one.md`.\n\nAcceptance criteria:\n- The file contains exactly `one` followed by one newline.\n- No other file changes.\n- `pnpm check` passes.\n- Open a PR that closes this issue.'
```

### Issue 2 — Feature

```bash
gh issue create \
  --title "feat: add decrementCounter" \
  --label "agent-ready,feature" \
  --body $'Implement `decrementCounter(value)` in `src/counter.ts`.\n\nAcceptance criteria:\n- Returns the input minus one.\n- Add focused Vitest coverage.\n- Do not add dependencies.\n- `pnpm check` passes.\n- Open a PR that closes this issue.'
```

### Issue 3 — Bug fix

```bash
gh issue create \
  --title "fix: reject non-integer counter values" \
  --label "bug" \
  --body $'Counter functions currently accept non-integer values.\n\nAcceptance criteria:\n- `incrementCounter` and `decrementCounter` throw `TypeError` for non-integer input.\n- The error message is `value must be an integer`.\n- Add regression tests.\n- Do not begin until issue 2 is merged.\n- `pnpm check` passes.'
```

Leave issue 3 without `agent-ready` until issue 2 has merged.

## Step 3. Give issue 1 to Codex

In Codex, use this prompt and replace `<number>`:

```text
Implement GitHub issue #<number> in this repository.

Rules:
- Read AGENTS.md and CONTEXT.md first.
- Inspect the issue with gh.
- Create the required feature branch.
- Make only the issue's requested change.
- Run pnpm check.
- Review git diff before committing.
- Commit using Conventional Commits.
- Push the branch.
- Open a pull request against main that closes the issue.
- Do not merge.
- Stop after returning the PR URL, changed files, and checks run.
```

## Step 4. Review the PR yourself

```bash
gh pr list
gh pr view <pr-number> --web
```

Verify:

- Correct branch name
- Correct linked issue
- No unrelated files
- Checks passed
- PR does not auto-merge

Then merge manually:

```bash
gh pr merge <pr-number> --squash --delete-branch
```

## Step 5. Repeat for issue 2

Use the same prompt. After the feature PR merges, label issue 3:

```bash
gh issue edit <issue-3-number> --add-label agent-ready
```

## Completion check

**Checkpoint:** Two agent-created PRs were reviewed and merged by you, while issue 3 remained blocked until its dependency was satisfied.

---

# Milestone 7 — Build a VS Code “baby loop” with custom agents

## Purpose

Separate planning, implementation, and review into focused contexts without yet creating a fully autonomous background service. This is the safest place to learn orchestration.

## Important boundary

These `.agent.md` files are **VS Code Copilot custom agents**. They do not automatically configure Codex. Shared rules still come from `AGENTS.md`.

## Official docs

- Custom agents: https://code.visualstudio.com/docs/agent-customization/custom-agents
- Subagents: https://code.visualstudio.com/docs/agents/subagents
- Agent Skills: https://code.visualstudio.com/docs/agent-customization/agent-skills

## Step 1. Create custom agents through VS Code

1. Open the Command Palette.
2. Run `Chat: New Custom Agent`.
3. Save each workspace agent under `.github/agents/`.
4. Create:
   - `planner.agent.md`
   - `implementer.agent.md`
   - `reviewer.agent.md`
   - `coordinator.agent.md`

Use the VS Code UI to select current tool IDs rather than copying stale tool names from a tutorial.

## Step 2. Configure the planner

Purpose:

```text
Investigate one approved issue, ask questions when needed, and produce a bounded plan.
Never edit source files, commit, push, or merge.
```

Allowed capabilities:

- Read files
- Search the workspace
- Read GitHub issues
- Run non-mutating inspection commands

Model:

- Sol when architecture or ambiguity matters
- Terra for ordinary tickets

## Step 3. Configure the implementer

Purpose:

```text
Implement an approved plan on the issue branch, add tests, and run pnpm check.
Do not merge.
```

Allowed capabilities:

- Read and edit workspace files
- Terminal commands
- Tests and type checking
- Git commit and push
- Create a PR

Model:

- Terra for the normal default
- Sol for difficult tickets

## Step 4. Configure the reviewer

Purpose:

```text
Review the issue, specification, diff, and tests adversarially.
Return concrete findings with file locations and severity.
Do not edit files or merge.
```

Allowed capabilities:

- Read files and diffs
- Run tests
- Inspect the issue and PR

Model:

- Sol for the lab
- Fable later only when you intentionally add a Claude provider and accept its cost

## Step 5. Configure the coordinator

Purpose:

```text
Coordinate one approved GitHub issue from planning through PR readiness.
Delegate planning, implementation, and review to the appropriate custom agents.
Permit no more than two correction cycles.
Never merge.
```

Allowed capabilities:

- Invoke subagents
- Read issue and PR state
- Run Git/GitHub status commands
- Ask for human input

The coordinator should enforce this flow:

```text
read issue
→ planner
→ human clarification when needed
→ implementer
→ reviewer
→ at most two correction cycles
→ open or update PR
→ stop for human review
```

## Step 6. Test the coordinator with issue 3

Prompt:

```text
Coordinate approved issue #<issue-3-number>.
Use the planner, implementer, and reviewer custom agents.
Allow no more than two implementation-review cycles.
Stop if requirements are ambiguous or pnpm check cannot pass.
Do not merge. Return the pull-request URL and a concise audit trail.
```

## Completion check

**Checkpoint:** VS Code shows separate planning, implementation, and review work, and the final result is a PR requiring your approval.

---

# Milestone 8 — Install Docker and Sandcastle

## Purpose

Move execution out of the active working tree and into an isolated sandbox. Sandcastle is the programmable execution engine; GitHub remains the queue and review surface.

## Official links

- Sandcastle: https://github.com/mattpocock/sandcastle
- Docker Desktop for Windows: https://docs.docker.com/desktop/setup/install/windows-install/

## Step 1. Install Docker Desktop

Install Docker Desktop and enable:

- WSL 2 backend
- Integration with your Ubuntu distribution

Restart Docker Desktop and WSL when requested.

Verify inside WSL:

```bash
docker version
docker run --rm hello-world
```

Do not continue until both commands succeed.

## Step 2. Create a Sandcastle setup branch

```bash
cd ~/dev/agent-workflow-lab
git switch main
git pull --ff-only
git switch -c chore/add-sandcastle
```

## Step 3. Install Sandcastle

Use pnpm for the project package:

```bash
pnpm add --save-dev @ai-hero/sandcastle@latest tsx@^4.21.0
```

Run the initializer with pnpm:

```bash
pnpm dlx @ai-hero/sandcastle@latest init
```

The official npm equivalent is:

```bash
npx @ai-hero/sandcastle init
```

When the initializer presents choices, select:

- Agent: Codex
- Sandbox: Docker
- Template: simple loop or the smallest available starter
- Issue tracker: GitHub Issues when offered
- Branch strategy: one isolated branch per task
- Automatic merge: disabled

## Step 4. Configure authentication safely

Prefer Codex ChatGPT OAuth rather than placing an API key in Git.

1. Confirm the host Codex CLI is authenticated.
2. Inspect the generated `.sandcastle/.env.example`.
3. Copy it locally:

```bash
cp .sandcastle/.env.example .sandcastle/.env
```

4. Fill only the variables required by the generated Codex template.
5. Keep `.sandcastle/.env` ignored by Git.

Do not manually invent credential mount paths. Use the current generated template because Sandcastle authentication details can change between releases.

## Step 5. Set the first Sandcastle prompt

Create or replace `.sandcastle/prompt.md`:

```md
Read AGENTS.md and CONTEXT.md.

Create `docs/sandcastle.md` containing exactly:

sandcastle works

Then run `pnpm check`.
Commit the change on the sandbox branch.
Do not open or merge a pull request.
Stop after reporting the commit hash and check results.
```

## Step 6. Set strict limits in the generated runner

Use these initial controls in `.sandcastle/main.ts` or the generated equivalent:

- One agent
- One sandbox
- One branch
- Maximum 2 iterations
- No automatic merge
- No parallel workers
- No production credentials
- Completion requires `pnpm check`

Do not hard-code an undocumented model identifier when the generated provider lets Codex use its selected default. When a model must be explicit:

1. Prefer `gpt-5.6-sol` for planning or difficult work.
2. Prefer `gpt-5.6-terra` for routine implementation.
3. If the CLI rejects a name, select the exact identifier shown by the current Codex model picker rather than guessing.

## Step 7. Run Sandcastle

Use the command generated by the initializer. It will commonly resemble:

```bash
pnpm exec tsx .sandcastle/main.ts
```

Watch the logs. Then inspect:

```bash
git status
git branch --all
git log --oneline --all --decorate --max-count=20
```

## Step 8. Commit the Sandcastle configuration

Never commit `.sandcastle/.env`.

```bash
git add package.json pnpm-lock.yaml .sandcastle .gitignore
git status
git commit -m "chore: add bounded Sandcastle runner"
git push --set-upstream origin chore/add-sandcastle
gh pr create --fill --base main
```

Review and merge manually.

## Completion check

**Checkpoint:** Sandcastle ran a Codex agent in Docker, created the requested file on an isolated branch, and did not merge anything automatically.

---

# Milestone 9 — Build the bounded issue-to-PR loop

## Purpose

Convert Sandcastle from a one-off prompt runner into a controlled worker that handles exactly one approved issue and stops at a pull request.

## Required state machine

```text
agent-ready issue
→ claim issue
→ isolated branch
→ inspect AGENTS.md, CONTEXT.md, spec, and issue
→ implement
→ pnpm check
→ self-review
→ maximum two corrections
→ push
→ PR
→ label agent-done
→ stop for human
```

## Hard limits

Set these before any AFK use:

| Control | Initial value |
|---|---:|
| Concurrent issues | 1 |
| Implementation attempts | 3 |
| Review correction cycles | 2 |
| Subagents | 2 maximum |
| Wall-clock limit | 45 minutes |
| Automatic merge | Off |
| Production credentials | None |
| Daily purchased-credit budget | $10–$25 |

A loop without limits is not automation; it is an uncontrolled bill and code generator.

## Step 1. Create a dedicated test issue

```bash
gh issue create \
  --title "feat: add addCounter" \
  --label "agent-ready,feature" \
  --body $'Add `addCounter(value, step)`.\n\nAcceptance criteria:\n- Both arguments must be finite integers.\n- Return `value + step`.\n- Invalid input throws `TypeError` with a clear argument-specific message.\n- Add focused tests.\n- Do not add dependencies.\n- `pnpm check` passes.\n- Open a PR and stop without merging.'
```

## Step 2. Make the worker accept one issue number

The runner should require an explicit issue number from one of these sources:

- CLI argument, preferred for the lab
- Environment variable
- A Discord command later

Example interface:

```bash
pnpm sandcastle:issue --issue 4
```

Never begin by polling every open issue indefinitely.

## Step 3. Require an eligibility check

Before starting, the runner must verify:

- Issue exists
- Issue is open
- Issue has `agent-ready`
- Issue does not have `needs-human`
- No open PR already closes it
- Repository working tree is clean
- No other worker currently owns it

When any check fails, stop without invoking a model.

## Step 4. Require a completion contract

The worker succeeds only when all are true:

- Branch exists remotely
- Commit exists
- `pnpm check` passed after the final edit
- PR is open against `main`
- PR body contains `Closes #<issue-number>`
- No merge occurred

## Step 5. Require a failure contract

When blocked:

1. Add `needs-human`.
2. Post one concise issue comment describing the blocker.
3. Preserve the branch and logs.
4. Stop.

Do not let the model invent a requirement to keep the loop moving.

## Step 6. Test the bounded worker

Run the issue by number and watch the first full execution.

Afterward:

```bash
gh issue view <issue-number>
gh pr list
gh pr checks <pr-number>
gh pr diff <pr-number>
```

Review and merge manually only when the result is correct.

## Completion check

Run this process successfully three times before adding remote control.

**Checkpoint:** One explicit approved issue reliably becomes one isolated PR, with bounded retries and no automatic merge.

---

# Milestone 10 — Choose the AFK control path

## Purpose

Choose how you will start, inspect, stop, and approve work away from the keyboard. These are alternatives, not prerequisites that must all be stacked together.

## Option A — Codex mobile remote control

### Best for

- Steering active Codex sessions from your phone
- Reviewing diffs and command approvals
- Avoiding an extra bot and public messaging gateway

### Cost

- Included with your ChatGPT plan, subject to Codex usage limits
- No VPS required

### Time

- About 15–30 minutes when the feature is available for your Windows setup

### Steps

1. Update ChatGPT on Android.
2. Update the Codex app or extension on the host machine.
3. Sign into the same ChatGPT account.
4. Open Codex in ChatGPT mobile.
5. Connect to the trusted machine when offered.
6. Start a lab thread.
7. Confirm that you can see terminal output, diffs, tests, and approvals.

### Current Windows caveat

OpenAI's May 2026 announcement says mobile Codex is in preview, while direct phone connection to the Codex app on Windows is still rolling out. Use this option when it appears for your account; do not block the rest of the lab on it.

## Option B — Hermes on your current Windows computer

### Best for

- Discord commands
- General computer workflows beyond coding
- Reusing the Hermes setup you already tested

### Cost

- $0 software cost
- Uses your selected model subscription or API provider

### Time

- 30–90 minutes to harden the existing setup

### Main risk

Hermes can act with the permissions of your Windows user. Your daily machine contains browser profiles, documents, SSH keys, and client work. This is acceptable for a supervised experiment, not as the final unattended architecture.

## Option C — Dedicated local worker

### Best for

- Always-on local execution
- Keeping agent work away from your daily workstation
- Docker and multiple repositories

### Choices

- Existing spare PC: cheapest
- Low-cost mini PC: practical
- Mac mini: polished, but unnecessary for the first lab

### Cost

- Existing hardware: $0
- New Mac mini: starts around $799 at the time of this guide
- Electricity and storage are additional

### Time

- 2–4 hours for OS updates, Git, Node, Docker, Codex, Hermes, and Tailscale

## Option D — VPS worker

### Best for

- Always-on availability
- Strong separation from personal files
- Easy snapshots and deletion

### Cost

- Entry Linux VPS plans commonly start around $5–$6/month
- A comfortable 4–8 GB RAM development worker is commonly around $12–$24/month
- Model usage remains separate

### Time

- 1–3 hours

### Requirements

- Ubuntu LTS
- Dedicated non-root `agent` user
- Git, Node 24, pnpm 10, Docker, Codex, Sandcastle
- Tailscale
- No public SSH after Tailscale is verified, when practical

## Recommendation

Use this order:

1. Current PC for the isolated lab.
2. Codex mobile when available for simple remote steering.
3. Hermes on the current PC only with strict allowlists.
4. Move unattended Sandcastle/Hermes execution to a dedicated Linux worker or VPS.
5. Do not buy a Mac mini until the workflow has proven useful enough to justify dedicated hardware.

---

# Milestone 11 — Restrict Hermes to a control plane

## Purpose

Use Hermes to submit approved jobs and retrieve status, not to provide unrestricted remote shell access to your personal computer.

## Official links

- Docs: https://hermes-agent.nousresearch.com/docs/
- Quickstart: https://hermes-agent.nousresearch.com/docs/getting-started/quickstart
- Providers: https://hermes-agent.nousresearch.com/docs/integrations/providers
- Discord: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/discord

## Step 1. Update and verify Hermes

Use the updater provided by your existing installation:

```bash
hermes update
hermes --version
```

When Hermes is not installed in a given environment, follow the current official installer rather than copying an old command.

## Step 2. Select the OpenAI Codex provider

```bash
hermes model
```

Choose OpenAI Codex and complete device authentication. Hermes can use ChatGPT OAuth and can import existing Codex CLI credentials when present.

## Step 3. Verify Discord authorization

Run:

```bash
hermes gateway setup
```

Configure:

- Your Discord bot token
- `DISCORD_ALLOWED_USERS` containing only your numeric Discord user ID
- Direct messages or a private server channel
- Mention-required behavior in server channels

Never use an allow-all user configuration while terminal or filesystem tools are enabled.

## Step 4. Expose only bounded commands

The target interface should be:

```text
/status
/start-ticket <number>
/stop-ticket <number>
/show-ticket <number>
/show-pr <number>
/show-run-log <number>
```

Each command should call a fixed script with validated arguments. It should not concatenate raw Discord text into a shell command.

Example boundary:

```text
/start-ticket 42
        ↓
validate integer 42
        ↓
verify GitHub issue 42 has agent-ready
        ↓
start one Sandcastle process
        ↓
return run ID
```

## Step 5. Validate every argument

The start command must reject:

- Missing values
- Non-integers
- Extra shell characters
- Closed or absent issues
- Issues lacking `agent-ready`
- A second job while one is active

## Step 6. Disable consequential actions

Hermes must not be able to:

- Merge PRs
- Push directly to `main`
- Deploy
- Access production secrets
- Delete repositories
- Change GitHub permissions
- Install arbitrary software without approval
- Read the whole Windows home directory

## Step 7. Test from your phone

From Discord:

```text
/status
/start-ticket <lab-issue-number>
/show-pr <lab-issue-number>
```

The expected final response is a PR URL, not a merged change.

## Completion check

**Checkpoint:** Your phone can start exactly one approved lab issue and receive its PR URL, while invalid commands are rejected.

---

# Milestone 12 — Add Tailscale when a second machine exists

## Purpose

Create private connectivity between your phone, daily computer, and dedicated worker without exposing SSH or dashboards directly to the public internet.

## Official links

- Install: https://tailscale.com/docs/install
- Pricing: https://tailscale.com/pricing
- Access controls: https://tailscale.com/docs/features/access-control

## Cost

Tailscale Personal is free for a small personal network. Check the current plan limits before adding multiple people.

## Current-PC-only note

You do not need Tailscale merely to use a Discord bot. Add it when you have a dedicated machine, VPS, private dashboard, or SSH target.

## Windows + WSL choice

Choose one installation pattern and test it carefully:

- Install Tailscale on the Windows host when you want access to the whole Windows machine.
- Install it inside WSL only when the WSL distribution itself must be a distinct tailnet node.
- Avoid casually running competing host and WSL network paths without understanding the routing implications.

For the final dedicated worker, install Tailscale directly on its Linux OS.

## Linux worker setup

On the worker:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Confirm:

```bash
tailscale status
tailscale ip -4
```

From your daily computer:

```bash
ssh agent@<tailscale-hostname>
```

## Access-control target

Permit only:

- Justin's devices → worker SSH port
- Justin's devices → private dashboard port, when one exists
- Worker → GitHub and required model providers

Deny unnecessary lateral access.

## Completion check

**Checkpoint:** The worker is reachable over its Tailscale name, and the same service is not exposed through a public inbound firewall rule.

---

# Milestone 13 — Add CI as objective truth

## Purpose

Make GitHub independently verify the pull request. Agent self-review is useful, but deterministic checks are the acceptance gate.

## Step 1. Create the workflow

```bash
mkdir --parents .github/workflows

cat > .github/workflows/check.yml <<'EOF'
name: Check

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up pnpm
        uses: pnpm/action-setup@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: pnpm

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Run checks
        run: pnpm check
EOF
```

## Step 2. Commit through a PR

```bash
git switch -c ci/add-check-workflow
git add .github/workflows/check.yml
git commit -m "ci: verify pull requests"
git push --set-upstream origin ci/add-check-workflow
gh pr create --fill --base main
```

Review and merge manually.

## Step 3. Require green CI operationally

Even when branch protection is unavailable or not configured, establish this rule:

```text
No PR merges until GitHub Actions Check is green.
```

Later, enable a GitHub ruleset for `main` requiring:

- Pull request before merge
- Required `check` status
- No force push
- No deletion

## Completion check

**Checkpoint:** Every PR runs `pnpm check` independently on GitHub.

---

# Milestone 14 — Graduate from the lab

## Purpose

Decide whether the workflow is reliable enough for one non-critical real repository. Do not scale based on one impressive run.

## Required scorecard

Complete at least ten lab issues and record:

| Metric | Target |
|---|---:|
| PRs opened successfully | 10/10 |
| PRs merged without code correction | At least 7/10 |
| Unrelated file changes | 0 |
| Unauthorized merges | 0 |
| Escaped regressions | 0 |
| Average human review time | Under 15 minutes |
| Average model/credit cost | Acceptable for feature value |
| Runs exceeding limits | 0 |

## Production-repository entry rules

For the first real repository:

1. Choose a non-critical project.
2. Keep concurrency at one.
3. Keep automatic merge off.
4. Remove production secrets from the worker.
5. Give GitHub credentials only repository-level access.
6. Start with documentation or tests.
7. Progress to isolated bug fixes.
8. Progress to small features.
9. Add architecture work only after several successful runs.
10. Keep deployment human-controlled.

## When to use each model

| Work | Recommended model |
|---|---|
| Clear mechanical edit | Luna or Terra |
| Normal implementation | Terra |
| Ambiguous planning | Sol |
| Difficult bug or migration | Sol High |
| Final architecture review | Sol or Fable when intentionally available |
| Repeated tests, Git status, formatting | Use tools/scripts, not an expensive model |

## Final stable workflow

```text
Human idea
→ grill / clarify
→ approved specification
→ vertical GitHub ticket
→ human applies agent-ready
→ bounded Sandcastle worker
→ isolated branch and tests
→ independent review
→ GitHub CI
→ pull request
→ phone notification
→ human review
→ human merge
```

---

# Security checklist

Before any unattended run, verify every item:

- [ ] Repository is the lab or an approved non-critical project.
- [ ] `main` cannot be merged by the agent.
- [ ] Automatic merge is disabled.
- [ ] Worker has no production credentials.
- [ ] `.env` files are ignored.
- [ ] GitHub authorization is scoped as narrowly as practical.
- [ ] Discord is restricted to Justin's numeric user ID.
- [ ] Raw chat text is never executed as a shell command.
- [ ] One issue is processed at a time.
- [ ] Maximum attempts and wall-clock time are enforced.
- [ ] Every issue must have `agent-ready`.
- [ ] `needs-human` stops execution.
- [ ] `pnpm check` must pass locally and in CI.
- [ ] Logs, branch, and failure reason survive a stopped run.
- [ ] Human approval is required for merge and deployment.
- [ ] The worker cannot read the full personal home directory.
- [ ] Tailscale is treated as networking, not as a filesystem sandbox.

---

# Recommended order of implementation

Do not skip directly to Hermes plus an infinite queue.

```text
Day 1
Milestones 1–4
Workstation + tiny repo + shared instructions + direct agent tests

Day 2
Milestones 5–7
Skills + issues + manually supervised PR + VS Code baby loop

Day 3
Milestones 8–9
Docker + Sandcastle + one bounded issue worker

Day 4
Milestones 10–13
AFK choice + Hermes restrictions + optional Tailscale + CI

After ten clean runs
Milestone 14
Move one low-risk workflow into a real repository
```

# Definition of complete

The lab is complete when you can submit this from an approved interface:

```text
/start-ticket 4
```

…and receive:

```text
Issue: #4
Status: pull request ready
Checks: passed
Attempts: 1
PR: https://github.com/<owner>/agent-workflow-lab/pull/<number>
Merge: waiting for human approval
```

Nothing should merge, deploy, purchase, delete, or access production data without you.

---

# 🗺️ AI Agent Workflow Architecture

> Architecture diagrams for the workflow guide above. Milestone numbers match the setup sections.

## Legend

- **Solid arrow:** primary workflow or control flow
- **Dashed arrow:** supporting context, optional integration, or feedback
- **Diamond:** decision, gate, or stopping condition
- **Subgraph:** a major region of the overall system

---

# 1. Zoomed-Out World Map

```mermaid
%%{init: {
  "flowchart": {
    "curve": "basis",
    "nodeSpacing": 30,
    "rankSpacing": 48,
    "padding": 10
  },
  "themeVariables": {
    "fontSize": "14px"
  }
}}%%
flowchart TB

  classDef node fill:#1f2937,color:#ffffff,stroke:#94a3b8,stroke-width:1px;
  classDef decision fill:#111827,color:#ffffff,stroke:#f59e0b,stroke-width:1.5px;
  classDef outcome fill:#0f172a,color:#ffffff,stroke:#22c55e,stroke-width:1.2px;

  Justin["Justin<br/>Ideas • priorities • approvals"]:::node

  subgraph R1["Region 1 — Foundation · M1–M4"]
    direction LR
    M1["M1 Workstation<br/>VS Code • WSL • Git • Node"]:::node
    M2["M2 Lab repository<br/>TypeScript • Vitest"]:::node
    M3["M3 Project brain<br/>AGENTS.md • CONTEXT.md"]:::node
    M4["M4 AI clients<br/>Copilot • Codex"]:::node
    M1 --> M2 --> M3 --> M4
  end

  subgraph R2["Region 2 — Workflow Design · M5–M7"]
    direction LR
    M5["M5 Agent Skills<br/>Grill • Spec • Tickets"]:::node
    M6["M6 GitHub work queue<br/>Issues • labels • blockers"]:::node
    M7["M7 Agent team<br/>Coordinator • Planner • Implementer • Reviewer"]:::node
    M5 --> M6 --> M7
  end

  subgraph R3["Region 3 — Autonomous Execution · M8–M9"]
    direction LR
    M8["M8 Isolated runtime<br/>Docker • worktree • Sandcastle"]:::node
    M9["M9 Bounded worker<br/>One issue • limits • stop rules"]:::node
    M8 --> M9
  end

  subgraph R4["Region 4 — AFK + Hosting · M10–M12"]
    direction LR

    subgraph RC["Remote control"]
      direction TB
      M10{"M10 Remote control<br/>choice"}:::decision
      MOBILE["Codex mobile<br/>Direct session steering"]:::node
      HERMES["M11 Hermes + Discord<br/>Validated commands"]:::node
      M10 --> MOBILE
      M10 --> HERMES
    end

    subgraph HOSTS["Worker host"]
      direction TB
      HOST{"Host choice"}:::decision
      PC["Current PC"]:::node
      LOCAL["Dedicated local machine"]:::node
      VPS["VPS"]:::node
      M12["M12 Tailscale<br/>Private host access"]:::node

      HOST --> PC
      HOST --> LOCAL
      HOST --> VPS
      LOCAL -. optional .-> M12
      VPS -. optional .-> M12
    end

    HERMES --> HOST
  end

  subgraph R5["Region 5 — Verification + Delivery · M13–M14"]
    direction LR
    PR["Pull request"]:::node
    M13["M13 GitHub CI<br/>Typecheck • tests"]:::node
    REVIEW{"Human review"}:::decision
    MERGE["Human merge"]:::outcome
    M14["M14 Graduate to real repository"]:::outcome

    PR --> M13 --> REVIEW
    REVIEW -->|approve| MERGE --> M14
  end

  Justin --> M1
  Justin --> M5

  M4 --> M5
  M7 --> M8
  M9 --> PR

  M3 -. shared rules .-> M7
  M3 -. shared rules .-> M9
  M6 -. approved issues .-> M9

  MOBILE -. optional supervision .-> M9
  HERMES -. start • status • stop .-> M9

  REVIEW -->|reject| M9
  M14 -. feedback loop .-> Justin
```

---

# 2. Full Architecture Map

```mermaid
flowchart TB
  subgraph HUMAN_LAYER["Human Direction + Gates"]
    IDEA["Idea or problem"]
    SPEC_GATE{"Approve specification?"}
    READY_GATE{"Apply agent-ready?"}
    REVIEW_GATE{"Approve pull request?"}
    DEPLOY_GATE{"Approve deployment?"}
  end

  subgraph KNOWLEDGE["M3 + M5 · Knowledge and Procedures"]
    AGENTS["AGENTS.md<br/>Shared operating rules"]
    CONTEXT["CONTEXT.md<br/>Domain vocabulary"]
    SPECS["docs/specs<br/>Approved behavior"]
    ADRS["docs/adr<br/>Architecture decisions"]
    SKILLS["Agent Skills<br/>Reusable procedures"]
  end

  subgraph INTERFACES["M4 + M7 · AI Interfaces and Roles"]
    COPILOT["VS Code Copilot"]
    CODEX["Codex IDE / CLI"]
    COORD["Coordinator"]
    PLAN["Planner"]
    IMPLEMENT["Implementer"]
    REVIEWER["Reviewer"]

    COPILOT --> COORD
    CODEX --> COORD
    COORD --> PLAN
    COORD --> IMPLEMENT
    COORD --> REVIEWER
  end

  subgraph TRACKING["M6 · Durable Work State"]
    GH_ISSUES["GitHub Issues"]
    LABELS["Labels<br/>agent-ready • needs-human • agent-done"]
    DEPS["Dependencies / blockers"]
    COMMENTS["Status comments + audit trail"]
    GH_ISSUES --> LABELS
    GH_ISSUES --> DEPS
    GH_ISSUES --> COMMENTS
  end

  subgraph EXECUTION["M8 + M9 · Safe Execution"]
    ELIGIBILITY{"Eligibility gate"}
    SANDCASTLE["Sandcastle orchestrator"]
    DOCKER["Docker sandbox"]
    WORKTREE["Isolated Git worktree / branch"]
    LIMITS["Limits<br/>1 issue • 3 attempts • 2 reviews • 45 min"]
    TOOLS["Tools<br/>Files • terminal • Git • gh"]

    ELIGIBILITY --> SANDCASTLE
    SANDCASTLE --> DOCKER
    SANDCASTLE --> WORKTREE
    SANDCASTLE --> LIMITS
    DOCKER --> TOOLS
    WORKTREE --> TOOLS
  end

  subgraph VERIFICATION["M9 + M13 · Verification"]
    LOCAL_CHECKS["Local checks<br/>pnpm check"]
    SELF_REVIEW["Agent review"]
    PR["GitHub pull request"]
    CI["GitHub Actions CI"]
    FINDINGS{"Findings or failures?"}

    LOCAL_CHECKS --> SELF_REVIEW --> PR --> CI --> FINDINGS
  end

  subgraph REMOTE["M10–M12 · AFK and Hosting"]
    CONTROL{"Remote interface"}
    CODEX_MOBILE["Codex mobile"]
    HERMES["Hermes + Discord"]
    HOST{"Execution host"}
    CURRENT_PC["Current PC"]
    DEDICATED["Dedicated machine"]
    VPS["VPS"]
    TAILSCALE["Tailscale"]

    CONTROL --> CODEX_MOBILE
    CONTROL --> HERMES
    HERMES --> HOST
    HOST --> CURRENT_PC
    HOST --> DEDICATED
    HOST --> VPS
    DEDICATED -. optional .-> TAILSCALE
    VPS -. optional .-> TAILSCALE
  end

  IDEA --> SKILLS
  SKILLS --> SPECS
  SPECS --> SPEC_GATE
  SPEC_GATE -->|No| SKILLS
  SPEC_GATE -->|Yes| GH_ISSUES
  GH_ISSUES --> READY_GATE
  READY_GATE -->|No| GH_ISSUES
  READY_GATE -->|Yes| ELIGIBILITY

  AGENTS -. rules .-> COPILOT
  AGENTS -. rules .-> CODEX
  AGENTS -. rules .-> COORD
  AGENTS -. rules .-> SANDCASTLE
  CONTEXT -. domain context .-> PLAN
  SPECS -. acceptance criteria .-> PLAN
  ADRS -. constraints .-> PLAN
  SKILLS -. procedures .-> COORD

  PLAN --> IMPLEMENT
  IMPLEMENT --> TOOLS
  TOOLS --> LOCAL_CHECKS
  FINDINGS -->|Yes, within limits| IMPLEMENT
  FINDINGS -->|Yes, limits reached| NEEDS_HUMAN["Apply needs-human<br/>Preserve logs and branch"]
  FINDINGS -->|No| REVIEW_GATE

  REVIEW_GATE -->|Reject| IMPLEMENT
  REVIEW_GATE -->|Approve| MERGE["Human merge"]
  MERGE --> DEPLOY_GATE
  DEPLOY_GATE -->|Not yet| DONE["Merged and complete"]
  DEPLOY_GATE -->|Approve| DEPLOY["Human-controlled deployment"]

  CODEX_MOBILE -. steer sessions .-> COORD
  HERMES -. start-ticket / status / stop .-> ELIGIBILITY
  TAILSCALE -. private access .-> HOST
```

---

# 3. One Issue: Complete Runtime Loop

```mermaid
flowchart TD
  START["GitHub issue exists"] --> LABEL{"Has agent-ready<br/>and no needs-human?"}
  LABEL -->|No| STOP1["Stop without model call"]
  LABEL -->|Yes| DUP{"Open PR already exists?"}
  DUP -->|Yes| STOP2["Return existing PR"]
  DUP -->|No| CLAIM["Claim issue and create run ID"]

  CLAIM --> BRANCH["Create isolated worktree + branch"]
  BRANCH --> READ["Read AGENTS.md, CONTEXT.md,<br/>spec, issue, and relevant code"]
  READ --> PLAN["Planner creates bounded plan"]
  PLAN --> AMBIG{"Requirements clear?"}
  AMBIG -->|No| HUMAN["Apply needs-human<br/>Post blocker comment<br/>Stop"]
  AMBIG -->|Yes| IMPLEMENT["Implementer edits code"]

  IMPLEMENT --> CHECK["Run focused tests + pnpm check"]
  CHECK --> PASS{"Checks pass?"}
  PASS -->|No| ATTEMPTS{"Attempts remaining?"}
  ATTEMPTS -->|Yes| IMPLEMENT
  ATTEMPTS -->|No| HUMAN

  PASS -->|Yes| REVIEW["Reviewer compares<br/>issue + spec + diff + tests"]
  REVIEW --> FINDINGS{"Actionable findings?"}
  FINDINGS -->|Yes| CYCLES{"Review cycles remaining?"}
  CYCLES -->|Yes| IMPLEMENT
  CYCLES -->|No| HUMAN

  FINDINGS -->|No| COMMIT["Commit with Conventional Commit"]
  COMMIT --> PUSH["Push branch"]
  PUSH --> PR["Open PR<br/>Closes issue"]
  PR --> CI["GitHub CI"]
  CI --> CI_PASS{"CI green?"}
  CI_PASS -->|No| ATTEMPTS
  CI_PASS -->|Yes| HUMAN_REVIEW{"Justin reviews PR"}

  HUMAN_REVIEW -->|Reject| IMPLEMENT
  HUMAN_REVIEW -->|Approve| MERGE["Justin merges"]
  MERGE --> CLOSE["Issue closes<br/>Branch deleted<br/>Run archived"]
```

---

# 4. AFK + Hosting Decision Map

```mermaid
flowchart TB
  START["Need to work away from keyboard"] --> CONTROL{"How should work be controlled?"}

  CONTROL -->|Direct agent session| MOBILE["Codex mobile"]
  CONTROL -->|Commands + notifications| HERMES["Hermes + Discord"]
  CONTROL -->|No remote control yet| LOCAL_ONLY["Run manually from VS Code / terminal"]

  MOBILE --> HOST_DECISION{"Where does the worker run?"}
  HERMES --> COMMANDS["Restricted commands<br/>status • start-ticket • stop-ticket • show-pr"]
  COMMANDS --> HOST_DECISION
  LOCAL_ONLY --> HOST_DECISION

  HOST_DECISION --> PC["Current PC<br/>$0 • fastest setup • weakest isolation"]
  HOST_DECISION --> SPARE["Dedicated local machine<br/>always-on • better isolation • hardware cost"]
  HOST_DECISION --> VPS["VPS<br/>monthly cost • strong separation • always-on"]

  PC --> PC_RULES["Supervised lab only<br/>Separate worktree<br/>No production secrets"]
  SPARE --> PRIVATE{"Need private SSH or dashboard?"}
  VPS --> PRIVATE

  PRIVATE -->|No| PROVIDER_ONLY["Use outbound GitHub + model connections"]
  PRIVATE -->|Yes| TAIL["Tailscale<br/>Private network path"]

  PC_RULES --> EXEC["Sandcastle + Docker worker"]
  PROVIDER_ONLY --> EXEC
  TAIL --> EXEC

  EXEC --> GATE["PR + CI + human merge"]
```

---

# 5. Knowledge, Skills, Agents, Models, and Tools

```mermaid
%%{init: {
  "flowchart": {
    "curve": "basis",
    "nodeSpacing": 34,
    "rankSpacing": 56,
    "padding": 10
  },
  "themeVariables": {
    "fontSize": "14px"
  }
}}%%
flowchart LR

  classDef leaf fill:#1f2937,color:#ffffff,stroke:#94a3b8,stroke-width:1px;
  classDef hub fill:#0f172a,color:#ffffff,stroke:#38bdf8,stroke-width:1.5px;
  classDef action fill:#111827,color:#ffffff,stroke:#22c55e,stroke-width:1px;

  subgraph INFO["Information Layer · What the system knows"]
    direction TB
    AGENTS["AGENTS.md<br/>Rules"]:::leaf
    CONTEXT["CONTEXT.md<br/>Domain"]:::leaf
    SPECS["Specifications<br/>Expected behavior"]:::leaf
    ADRS["ADRs<br/>Architecture decisions"]:::leaf
    ISSUES["GitHub Issues<br/>Current work"]:::leaf
    INFO_HUB["Project knowledge pack"]:::hub

    AGENTS --> INFO_HUB
    CONTEXT --> INFO_HUB
    SPECS --> INFO_HUB
    ADRS --> INFO_HUB
    ISSUES --> INFO_HUB
  end

  subgraph BEHAVIOR["Behavior Layer · How work should be done"]
    direction TB
    SKILLS["Skills<br/>Reusable procedures"]:::leaf
    PROMPTS["Prompt files<br/>Manual task starters"]:::leaf
    AGENT_FILES["Custom agents<br/>Roles + permissions"]:::leaf
    HOOKS["Hooks / scripts<br/>Deterministic enforcement"]:::leaf
    BEHAVIOR_HUB["Workflow behavior pack"]:::hub

    SKILLS --> BEHAVIOR_HUB
    PROMPTS --> BEHAVIOR_HUB
    AGENT_FILES --> BEHAVIOR_HUB
    HOOKS --> BEHAVIOR_HUB
  end

  subgraph INTELLIGENCE["Intelligence Layer · Who reasons"]
    direction TB
    SOL["Sol<br/>Orchestration • planning • hard bugs"]:::leaf
    TERRA["Terra<br/>Routine implementation"]:::leaf
    FABLE["Fable<br/>Architecture • adversarial review"]:::leaf
    MODEL_HUB["Model pool"]:::hub

    SOL --> MODEL_HUB
    TERRA --> MODEL_HUB
    FABLE --> MODEL_HUB
  end

  subgraph RUNTIME["Runtime Layer · What runs the loop"]
    direction TB
    COPILOT["Copilot Agent"]:::leaf
    CODEX["Codex"]:::leaf
    SANDCASTLE["Sandcastle"]:::leaf
    HERMES["Hermes"]:::leaf
    RUNTIME_HUB["Agent runtimes"]:::hub

    COPILOT --> RUNTIME_HUB
    CODEX --> RUNTIME_HUB
    SANDCASTLE --> RUNTIME_HUB
    HERMES --> RUNTIME_HUB
  end

  subgraph ACTIONS["Action Layer · What can change the world"]
    direction TB
    ACTION_HUB["World actions"]:::hub
    FILES["Filesystem"]:::action
    TERMINAL["Terminal"]:::action
    GIT["Git"]:::action
    GH["GitHub CLI / API"]:::action
    TESTS["Tests + typecheck"]:::action
    BROWSER["Browser / computer use"]:::action
    MCP["MCP tools"]:::action

    ACTION_HUB --> FILES
    ACTION_HUB --> TERMINAL
    ACTION_HUB --> GIT
    ACTION_HUB --> GH
    ACTION_HUB --> TESTS
    ACTION_HUB --> BROWSER
    ACTION_HUB --> MCP
  end

  INFO_HUB --> RUNTIME_HUB
  BEHAVIOR_HUB --> RUNTIME_HUB
  MODEL_HUB --> RUNTIME_HUB
  RUNTIME_HUB --> ACTION_HUB

  AGENTS -. shared instructions .-> COPILOT
  AGENTS -. shared instructions .-> CODEX

  SKILLS -. reusable procedures .-> COPILOT
  SKILLS -. reusable procedures .-> CODEX

  PROMPTS -. manual invocation .-> COPILOT
  AGENT_FILES -. Copilot-specific roles .-> COPILOT
  HOOKS -. enforce around tools .-> ACTION_HUB

  HERMES -. launches bounded jobs .-> SANDCASTLE
  SANDCASTLE -. isolated execution .-> GIT
  SANDCASTLE -. isolated execution .-> GH
  SANDCASTLE -. isolated execution .-> TESTS
```

---

# 6. End-to-End Sequence Diagram

```mermaid
sequenceDiagram
  autonumber
  actor Justin
  participant Skills as Skills / Spec Flow
  participant GitHub as GitHub Issues
  participant Control as Copilot, Codex, or Hermes
  participant Worker as Sandcastle Coordinator
  participant Planner
  participant Builder as Implementer
  participant Reviewer
  participant CI as GitHub Actions

  Justin->>Skills: Describe idea and answer questions
  Skills-->>Justin: Draft specification
  Justin->>Skills: Approve specification
  Skills->>GitHub: Create vertical issue(s)
  Justin->>GitHub: Apply agent-ready label

  Justin->>Control: Start approved ticket
  Control->>Worker: Submit validated issue number
  Worker->>GitHub: Verify labels, state, blockers, and existing PRs
  GitHub-->>Worker: Issue is eligible

  Worker->>Planner: Research and create bounded plan
  Planner-->>Worker: Plan or clarification request

  alt Requirements are ambiguous
    Worker->>GitHub: Add needs-human and post blocker
    GitHub-->>Justin: Notify human
  else Requirements are clear
    Worker->>Builder: Implement plan in isolated worktree
    Builder->>Builder: Edit, test, and typecheck
    Builder-->>Worker: Diff and check results

    Worker->>Reviewer: Review issue, spec, diff, and tests
    Reviewer-->>Worker: Findings

    loop Maximum two correction cycles
      Worker->>Builder: Fix actionable findings
      Builder->>Builder: Re-run checks
      Builder-->>Worker: Updated diff
      Worker->>Reviewer: Re-review
      Reviewer-->>Worker: Remaining findings
    end

    Worker->>GitHub: Push branch and open PR
    GitHub->>CI: Trigger CI
    CI-->>GitHub: Pass or fail

    alt CI fails
      GitHub-->>Worker: Failure details
      Worker->>Builder: Correct within attempt limit
    else CI passes
      GitHub-->>Justin: PR ready for review
      Justin->>GitHub: Approve and merge, or request changes
    end
  end
```

---

# Recommended Reading Order

1. **Zoomed-Out World Map** — understand the whole system.
2. **Full Architecture Map** — understand the components and cross-connections.
3. **One Issue Runtime Loop** — understand execution and stop conditions.
4. **AFK + Hosting Decision Map** — choose where and how it runs.
5. **Knowledge / Skills / Agents / Models / Tools** — understand the vocabulary.
6. **Sequence Diagram** — understand the exact communication order.
