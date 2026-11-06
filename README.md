# Ace classic

> **Archived snapshot.** Frozen copy of ardallie/ace-in-my-space at 70d8f84 (version 0.4.2) -- the
> classic `agent-*` suite before `scope-*`/`ws-*` development. Read-only; no further changes. The
> plugin is renamed ace-classic so it installs alongside the current ace plugin; commands are
> `/ace-classic:<skill>`. Both suites write to the same .ace/ working directories, so avoid running
> both in one project at once. Install from the default branch as shown below. The tag
> v0.4.2-classic marks the upstream cut point (70d8f84, before the rename) and is not an install
> ref: its tree still declares plugin `ace` in marketplace `ace-in-my-space`, which clashes with
> the current ace plugin. Model tiers are mapped in skills/detect-harness/models.md -- adjust them
> first when running on other models, then run `node scripts/build-codex.mjs` so the Codex copy,
> codex/skills/detect-harness/models.md, follows. The Codex derivation, Updating, and Versioning
> sections below are inherited from upstream and apply only to local forks of this snapshot.

A suite of agent skills for planning, multi-agent review, pull-request work, report
publication, and harness detection. It ships as a single plugin named `ace-classic`,
distributed from the `ace-classic` marketplace (this repository).

Twenty skills and three subagent payloads. The skills invoke one another by name, so they
are packaged and versioned together rather than split per workflow.

## Installation

### Claude Code

```
/plugin marketplace add ardallie/ace-classic
/plugin install ace-classic@ace-classic
```

The same two steps from a shell:

```
claude plugin marketplace add ardallie/ace-classic
claude plugin install ace-classic@ace-classic
```

`claude plugin install` takes `--scope user` (default), `project`, or `local`. For local
development, point the marketplace at a checkout instead of the GitHub slug:

```
claude plugin marketplace add /path/to/ace-classic
```

Every skill registers under `/ace-classic:{name}` -- for example `/ace-classic:plan-start`. The `ace-classic`
plugin namespace supplies uniqueness, so the inner names carry no suite prefix of their
own. Subagents register under `ace-classic:{name}` -- `ace-classic:plan-drafter` -- and must be addressed
that way. Always use the plugin-qualified form; the suite's own chains do.

### Codex CLI

The Codex edition ships from this repository as a native Codex plugin. Add the Git
marketplace, then install `ace-classic`:

```
codex plugin marketplace add ardallie/ace-classic
codex plugin add ace-classic@ace-classic
```

Skills are then invoked as `$ace-classic:plan-start`, since Codex uses `$` mentions rather
than slash commands.

For local development, point the marketplace at a checkout and install from the same
marketplace name:

```
codex plugin marketplace add /path/to/ace-classic
codex plugin add ace-classic@ace-classic
```

## Commands

The command list below uses the Claude edition's slash form. Under Codex, replace the
leading `/` with `$`: `/ace-classic:plan-start` becomes `$ace-classic:plan-start`, `/ace-classic:pr-read` becomes
`$ace-classic:pr-read`, and so on. The skill name after the namespace is the same in both editions.

### Planning

- `/ace-classic:plan-start` -- drafts a plan from a brief, validates it, applies the findings, and
  publishes the result as a GitHub issue
- `/ace-classic:plan-validate` -- validates a plan against the codebase and writes a findings
  report
- `/ace-classic:plan-update` -- replaces a published plan issue's body with revised content
- `/ace-classic:plan-read` -- loads a plan and its handoffs from one or more GitHub issues
- `/ace-classic:plan-route` -- chooses the workflow (scope, pre-planner, plan-direct,
  implement-direct) for a work artefact or issue
- `/ace-classic:plan-implement` -- implements a published plan issue item by item, then posts the
  session handoff
- `/ace-classic:plan-handoff` -- posts a technical handoff comment to a GitHub issue

### Multi-agent analysis

- `/ace-classic:agent-scope` -- a deliberative panel over a large ambition, producing a staged
  scope envelope
- `/ace-classic:agent-arch-review` -- architectural review over a feature, subsystem, issue, or
  file set
- `/ace-classic:agent-pre-planner` -- pre-plan analysis producing an enhanced brief plus
  consolidated findings
- `/ace-classic:agent-code-review` -- pre-PR code review producing severity-graded findings
- `/ace-classic:agent-consultant` -- answers a cross-repository consultation request as the
  consultant side of the sibling-exchange protocol

### Pull requests

- `/ace-classic:pr-create` -- creates a pull request from the branch's commits
- `/ace-classic:pr-read` -- reads pull requests and all three of their comment surfaces into the
  session
- `/ace-classic:pr-response` -- responds to PR review comments with a summary and threaded replies

### Reports

- `/ace-classic:report-publish` -- publishes a report or plan markdown file as a GitHub issue
- `/ace-classic:report-triage` -- collates and de-duplicates findings from reports, issues, and PR
  threads, verifies each against HEAD, then applies them

### Utilities

- `/ace-classic:detect-harness` -- detects the agent harness and prints the capability-tier
  mapping the other skills resolve spawn models against
- `/ace-classic:run-interview` -- presents open questions interactively under the option-list
  rules
- `/ace-classic:run-retro` -- writes a retrospective report on a finished run

### Subagents

`ace-classic:plan-drafter`, `ace-classic:plan-validate`, and `ace-classic:plan-revisor` are spawned by
the planning skills and are not invoked directly.

## Rules

The plugin ships no rules files and adopts none of the host's as its own. Host repositories
supply their own rules -- language, style, and design conventions -- and no suite behaviour
depends on them. The one cross-cutting behavioural rule the suite carries, the option-list
rules every interview surface follows, lives in `skills/run-interview/SKILL.md`.

## Codex derivation

The Claude tree (`skills/` and `agents/`) is the source of truth. Regenerate
and check the committed Codex edition after changing it:

```
node scripts/build-codex.mjs
node scripts/check-codex.mjs
```

The conversion reduces skill frontmatter to `name` and `description`, moves argument
hints into the body, qualifies internal skill mentions, converts plugin-root paths to
relative references, adapts questions and subagent spawns, and copies the three supporting
agent payloads into `codex/`.

## Harness differences

- Claude invokes skills with slash commands; Codex invokes installed skills as
  `$ace-classic:{name}`. Codex chains are model-mediated instructions that name both the
  qualified skill and its sibling `SKILL.md`; Claude chains use the slash-command
  surface.
- Claude discovers the three custom agents from the plugin's `agents/` directory. The
  Codex plugin manifest has no custom-agent component, so the Codex orchestrators read
  `codex/agents/*.md` as supporting payloads and pass them through
  `collaboration.spawn_agent.message` on the built-in `default` role. The named custom
  role is not preserved.
- Claude's per-spawn worktree isolation field has no Codex collaboration equivalent.
  The Codex consultant confines file-creating probes to a verified operating-system
  temporary directory and leaves the shared repository tree untouched.
- Claude uses `AskUserQuestion`. Codex uses `request_user_input` when Plan mode exposes
  it and asks the same ordered options in plain text in Default mode. The structured
  Codex surface accepts at most three questions and two or three options per question,
  so larger interviews are batched; both paths retain keep-current behaviour and one
  first-position `(Recommended)` option.
- Claude substitutes `${CLAUDE_PLUGIN_ROOT}` in skill content. Codex resolves supporting
  files relative to the active `SKILL.md`; `${PLUGIN_ROOT}` is not used because it is a
  hook-command variable only.
- Claude frontmatter carries `argument-hint` and `disable-model-invocation`. Codex folds
  the hint into the skill body and relies on the default
  `policy.allow_implicit_invocation: true`, so no `agents/openai.yaml` is emitted.
- Claude agent payloads name Claude tools and permission modes. The Codex payloads use
  `exec_command`, `rg`, `apply_patch`, and `collaboration.spawn_agent`; their prompt-level
  write boundaries remain binding.
- The Codex harness detector publishes only the Codex `(model, reasoning_effort)` tier
  mapping. The Claude edition retains its Claude-model mapping.

## Host requirements

The suite assumes the host session provides:

- an authenticated `gh` CLI -- the plan, PR, and report skills create and edit GitHub
  issues, pull requests, and comments through it
- `git` -- branch, diff, and commit resolution
- a POSIX shell -- the skills' embedded command snippets are written for one (Git Bash on
  Windows)

These are hard requirements, not optional integrations: `/ace-classic:plan-start`,
`/ace-classic:plan-update`, `/ace-classic:plan-handoff`, `/ace-classic:report-publish`, and every `/ace-classic:pr-*`
skill stop without them.

Skills that cite a host surface this plugin does not ship -- a repository's own skills
tree, registries, script plane, or work-packaging artefacts -- mark the citation `[host]`
and state what happens where the surface is absent. No finding follows from the absence.

## Artefacts

The suite writes its reports and plans into an `.ace/` tree at the root of whatever
repository it runs in:

- `.ace/plans/` -- plan drafts, revisions, and validation reports
- `.ace/reports/` -- code-review reports, triage inputs, consultation envelopes
- `.ace/scope/` -- scope envelopes
- `.ace/arch-review/` -- architectural review reports
- `.ace/retro/` -- retrospectives

Filenames are `{yyyyMMdd}-{HHmm}-{kind}-{suffix}.md`. The `.ace/` root is a convention,
not a configuration surface; add it to the host repository's `.gitignore` if the artefacts
should not be committed.

## Updating

Claude Code:

```
claude plugin marketplace update ace-classic
claude plugin update ace-classic
```

Codex CLI has no plugin-update command, so the update path is a marketplace upgrade
followed by a remove and add:

```
codex plugin marketplace upgrade ace-classic
codex plugin remove ace-classic@ace-classic
codex plugin add ace-classic@ace-classic
```

For a local checkout, bump the SemVer value in `.codex-plugin/plugin.json` (a build
metadata cachebuster is sufficient), then reinstall without a marketplace upgrade:

```
codex plugin remove ace-classic@ace-classic
codex plugin add ace-classic@ace-classic
```

## Versioning

Semantic versioning from `0.1.0`. The Claude and Codex manifests carry the same version
from the same repository release, and `.claude-plugin/marketplace.json` carries it in both
its `metadata.version` and its plugin entry -- `node scripts/check-codex.mjs` fails if the
four disagree.

See `CHANGELOG.md` for release notes, including the `0.2.0` invocation migration.

## Licence

MIT. See `LICENSE`.
