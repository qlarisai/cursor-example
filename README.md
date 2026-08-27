# qlaris — Base Template

A **base template project for qlaris**. Clone it and you have a working qlaris setup: the MCP
server registered, the operating rules in place, and the research directories already gitignored.
Nothing to scaffold, nothing to wire up.

Research runs on the [qlaris](https://app.qlaris.ai) MCP server against digital-twin persona
panels — surveys, UX tests, and interviews — not by an AI pretending to be a user.

There is no application source, no build, and no tests. What this repo produces is research:
markdown briefs, HTML prototypes, and served report URLs.

Use it as the starting point for a new piece of research, or copy the two config files into a
project you already have (see [Setting this up in another project](#setting-this-up-in-another-project)).

---

## Setup (5 minutes)

### 1. Install Cursor

Install [Cursor](https://cursor.com) and sign in.

### 2. Clone and open the repo

```bash
git clone https://github.com/qlarisai/cursor-example.git qlaris-workspace
```

Then open that folder in Cursor: **File → Open Folder…**, or drag it onto the Cursor icon.

The `cursor` terminal command is **not** installed by default. If you want to open folders from the
shell, run **Shell Command: Install 'cursor' command** from the Command Palette (**Cmd+Shift+P**)
first — after that, `cursor qlaris-workspace` works.

Name the target directory after the research you're doing — `qlaris-workspace` is a placeholder.
If you have write access and prefer SSH, use `git@github.com:qlarisai/cursor-example.git`.

`.cursor/mcp.json` is checked in, so Cursor sees the `qlaris` server as soon as the folder opens.

### 3. Enable the server and authenticate

Open **Customize → MCPs** and toggle `qlaris` **on**. The first connection opens a browser window
to authenticate against your qlaris account — if that didn't happen, use the login prompt next to
the server in that panel.

`qlaris` must show as **connected** before you do anything else. If a tool call misbehaves, check
the logs: **Cmd+Shift+U** → select **MCP Logs** from the dropdown.

### 4. Stop it prompting mid-study

In **Cursor Settings → Agents → Approvals & Execution**, set the run mode to **Allowlist** and add
this single entry to the MCP allowlist:

```
qlaris:*
```

That covers every qlaris tool. The format is the part that trips people up: entries are
**`server:tool`**, where `server` is the key from `.cursor/mcp.json`. An entry without a colon is
silently ignored, so a bare `qlaris` — or a `mcp__qlaris` style string — does nothing.

| Entry | Matches |
|---|---|
| `qlaris:*` | every qlaris tool |
| `qlaris:run_survey` | that one tool |
| `qlaris:list_*` | globs work inside names |
| `*:*` | every tool from every server |

The run mode matters as much as the entry. **Allowlist** runs allowlisted actions without approval.
**Auto-review** does too, but it needs a Cursor-managed classifier model — on a team plan that
blocks those models it is greyed out and unavailable. **Run Everything** skips the allowlist
entirely and prompts for nothing.

`.cursor/permissions.json` is checked in carrying the same entry:

```json
{ "mcpAllowlist": ["qlaris:*"] }
```

Where Cursor reads it, the setup above happens on clone and the in-app list goes read-only citing
the file — that is expected, not a problem. Where it doesn't (a team-managed run mode overrides
both the file and the in-app list), add the entry by hand as above. Adding it manually always works.

Worth knowing what that covers: allowing the whole server means a run isn't interrupted by
permission prompts mid-study, and that includes the tools that spend research units and the ones
that delete. The `dryRun` step still shows cost before anything large runs. Allowlist individual
tools instead if you want the prompt back on spend.

### 5. Check your units

Make sure your qlaris account has research units available. Anything large is dry-run for cost
first, so you see the price before it is spent.

### Setting this up in another project

Nothing here is repo-specific. To get the same command surface anywhere, drop this into that
project's `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "qlaris": {
      "url": "https://app.qlaris.ai/api/mcp"
    }
  }
}
```

Then copy `AGENTS.md` if you want the same operating rules (thresholds, no-fabrication, pipeline
structure) to apply there too.

---

## Using it: `with qlaris:`

**`with qlaris:` is the entire interface.** Any message containing the phrase `with qlaris` (any
casing, colon optional) routes through the qlaris skill server before the agent answers — so you get
the real command surface, not the model's memory of it.

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
Run each phase in a **new chat**, passing the handover forward — the pipeline is designed so that no
single conversation carries the whole run. See `AGENTS.md` for how it's structured and which gates
it enforces.

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

These are enforced by `AGENTS.md` and by the qlaris skills, because the failure mode of synthetic
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
.cursor/mcp.json              registers the qlaris MCP server (checked in)
.cursor/permissions.json      allows every qlaris tool without a prompt (checked in)
AGENTS.md                     operating rules, loaded automatically by Cursor
discovery/<slug>/             pipeline run state, briefs, prototypes (gitignored)
context/                      optional local context files (gitignored)
```

All discovery skills live on the qlaris server, not in this repo — the agent discovers them with
`list_skills` and loads them with `get_skill`. There is nothing to install or update locally.
