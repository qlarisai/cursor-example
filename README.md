# Product Discovery with AI — qlaris Workspace

A ready-to-run workspace for doing **AI-assisted product discovery** on a real backlog item.
Research is run by the [qlaris](https://app.qlaris.ai) MCP server against digital-twin persona
panels — surveys, UX tests, and interviews — not by Claude pretending to be a user.

There is no application source, no build, and no tests. The output of this repo is research:
markdown briefs, HTML prototypes, and served report URLs.

---

## Setup (5 minutes)

### 1. Install Claude Code

Install [Claude Code](https://claude.com/claude-code) and sign in.

### 2. Clone and open the repo

```bash
git clone <this-repo> pm-ai-example && cd pm-ai-example && claude
```

When Claude Code asks whether to trust the folder and enable the `qlaris` MCP server, say **yes**.

### 3. Check the MCP server

`.mcp.json` is checked in and registers a single HTTP MCP server:

```json
{
  "mcpServers": {
    "qlaris": {
      "type": "http",
      "url": "https://app.qlaris.ai/api/mcp"
    }
  }
}
```

`.claude/settings.json` is checked in and enables it for everyone who clones the repo, with every
qlaris tool pre-allowed so a run isn't interrupted by permission prompts mid-study:

```json
{
  "enabledMcpjsonServers": ["qlaris"],
  "permissions": {
    "allow": ["mcp__qlaris"]
  }
}
```

`mcp__qlaris` is a whole-server rule — it covers every current and future qlaris tool, including
the ones that spend research units (`run_survey`, `run_user_test`, `start_interview`) and the ones
that delete (`delete_persona`, `delete_prototype`, `remove_research_document`). Cost is still shown
by the `dryRun` step before anything large runs, but the permission prompt is gone. Narrow it to
specific `mcp__qlaris__<tool>` rules if you want the prompt back on spend.

`.claude/settings.local.json` sits next to it, is gitignored, and is where your own permission
grants land. It overrides `settings.json` key by key — leave it as `{}` unless you need a personal
override.

Verify it connected:

```
/mcp
```

`qlaris` must show as **connected**. The first connection opens a browser window to authenticate
against your qlaris account — if that didn't happen, run `/mcp` and pick `qlaris` to authenticate
manually.

**If it doesn't connect, stop.** Nothing in this repo works without it, and there is deliberately
no fallback — Claude will not roleplay a persona panel in place of real qlaris results.

### 4. Check your units

Make sure your qlaris account has research units available. Anything large is dry-run for cost
first, so you see the price before it is spent.

### Setting this up in another project

Nothing here is repo-specific. To get the same command surface anywhere:

```bash
claude mcp add --transport http qlaris https://app.qlaris.ai/api/mcp
```

Then copy `CLAUDE.md` if you want the same operating rules (thresholds, no-fabrication, pipeline
structure) to apply there too.

---

## Using it: `with qlaris:`

**`with qlaris:` is the entire interface.** Any message containing the phrase `with qlaris` (any
casing, colon optional) routes through the qlaris skill server before Claude answers — so you get
the real command surface, not Claude's memory of it.

### Three ways to ask

| Say | What happens |
|---|---|
| `with qlaris: <research goal>` | qlaris does it — picks the audience, runs the study, delivers findings. It asks first when the goal is large or unclear. |
| `with qlaris: how do I …` | Explains and points at the right tool or skill. **Runs nothing.** |
| `with qlaris:` | Prints the full command cheat sheet. |

Add as much or as little detail as you like — audience, sub-questions, method, sample size, output
format. What you leave out, qlaris fills in sensibly or asks about.

### The same question, two ways

Quick:

```
with qlaris: Find out if Americans prefer chocolate or vanilla ice cream at the cinema.
```

Steered:

```
with qlaris: Should a new US cinema chain mainly sell ice cream, and which flavor?
Focus on the western USA, under 45. Detailed report, qualitative through quantitative.
Ask clarifying questions first.
```

Both work. The second just spends fewer rounds converging.

### Personas — build and reuse audiences

| Say | What happens |
|---|---|
| `with qlaris: create a persona <description>` | Builds a reusable audience definition from a short narrative. |
| `with qlaris: list my personas` | Shows existing personas to reuse. |

A persona is a **set of criteria matching many panel members**, not one individual. When you create
one you'll see `matchedCount` — how many real panel members it reaches. Keep descriptions as short
narratives; over-defining shrinks the panel to nobody.

### Research methods

| Say | What happens |
|---|---|
| `with qlaris: run a survey <specifications>` | Quantitative survey on a digital-twin panel. |
| `with qlaris: run a UX test <specifications>` | Twins navigate a URL and report their experience. |
| `with qlaris: run an interview <specifications>` | Moderated discussion or open focus group. |

### The discovery pipeline

| Say | Phase |
|---|---|
| `with qlaris: run the discovery flow <idea>` | The whole pipeline, end to end |
| `with qlaris: frame this opportunity` | 1 · Opportunity |
| `with qlaris: explore solutions` | 2 · Solution |
| `with qlaris: write the PRD` | 3 · Concept |
| `with qlaris: prototype this` | 4 · Prototype |
| `with qlaris: validate this` | 5 · Validation |
| `with qlaris: reconcile` | 6 · Reconciliation |

The full flow takes hours, not minutes, and produces a validated concept plus a working prototype.
See `CLAUDE.md` for how it's structured and which gates it enforces.

### Querying what's already known

```
qlaris knowledge: <question>
```

A separate trigger. Queries the existing research repository and answers with sources — it runs no
new study. If the repository doesn't cover your question, it says so plainly instead of guessing.

### Tips

- **Say the decision, not the topic.** "Should we charge for X?" beats "ask about pricing".
- **Be as specific or as hands-off as you like.** Name the audience and the exact questions, or
  leave both to qlaris.
- **Interrupt freely.** Ask what a digital twin is, why a threshold sits where it does, or to
  rewrite a question mid-flow.

---

## The rules that don't bend

These are enforced by `CLAUDE.md` and by the qlaris skills, because the failure mode of synthetic
research is *agreeable, plausible, wrong*.

**Thresholds are fixed and applied mechanically.** Likert needs mean ≥4.0/5 *and* ≥70% positive. A
single-choice winner needs ≥60%. Willingness to pay needs ≥80% willing with 0% "would not pay".
Below by any margin is a fail — no rounding, no "practically equivalent".

**Lukewarm agreement is a fail, even at a high percentage.** "Somewhat appealing" and "partially
addresses my problem" are politeness, not validation.

**The panel is locked before the research runs.** Which persona decides and which merely informs is
recorded first, so results can't be re-labelled after the fact.

**Predictions are never reported as observations.** If a study hasn't finished, it's reported as
in-flight with a link. It never says what the panel "would have" said.

**A strong contradiction is the finding.** If ≥40% of responses cut against the framing, that goes
at the top — and the upstream framing gets re-run, not annotated.

**Nothing is invented on your behalf.** Unknown inputs are recorded as `[UNKNOWN]`. A thin brief
that's true produces useful research; a rich brief that's made up produces confident nonsense.

---

## Repo layout

```
.mcp.json                     registers the qlaris MCP server (checked in)
.claude/settings.json         enables it + allows all qlaris tools (checked in)
.claude/settings.local.json   your personal overrides (gitignored)
CLAUDE.md                     operating rules for Claude in this repo
AGENTS.md                     symlink to CLAUDE.md, for non-Claude agents
discovery/<slug>/             pipeline run state, briefs, prototypes (gitignored)
context/                      optional local context files (gitignored)
```

All discovery skills live on the qlaris server, not in this repo — Claude discovers them with
`list_skills` and loads them with `get_skill`. There is nothing to install or update locally.
