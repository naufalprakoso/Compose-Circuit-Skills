# Compose Circuit Skills

Codex Agent Skill for building, reviewing, refactoring, debugging, and testing Kotlin Android or Kotlin Multiplatform features that use Slack Circuit with Jetpack Compose or Compose Multiplatform.

## What It Covers

- Slack Circuit screens, presenters, UI state, UI events, factories, navigation, overlays, retained state, and SubCircuit.
- Compose UI architecture, performance, stability, previews, and accessibility.
- Repository, networking, coroutine, lifecycle, resilience, and security patterns.
- Android-only and Kotlin Multiplatform boundaries.
- Presenter, UI, navigation, networking, coroutine, accessibility, and review-focused testing.

## Install For AI Agents

This repository is the skill folder. Install it so the final path contains `SKILL.md` directly:

```text
compose-circuit-skills/
├── SKILL.md
├── agents/
└── references/
```

Avoid this shape because most skill loaders will not find the skill automatically:

```text
skills/
└── compose-circuit-skills/
    └── compose-circuit-skills/
        └── SKILL.md
```

### Codex / Agents Desktop

Clone this repository into your local skills directory:

```bash
mkdir -p "$HOME/.agents/skills"
git clone git@github.com:naufalprakoso/Compose-Circuit-Skills.git \
  "$HOME/.agents/skills/compose-circuit-skills"
```

Restart the agent app or start a new session after installing. Invoke it with:

```text
$compose-circuit-skills
```

### Codex CLI / CODEX_HOME

If your Codex setup uses `$CODEX_HOME/skills`, install there instead:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
git clone git@github.com:naufalprakoso/Compose-Circuit-Skills.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/compose-circuit-skills"
```

Then start a new Codex session and invoke:

```text
$compose-circuit-skills
```

### Other AI Coding Agents

This is a Codex Skill, so automatic discovery depends on whether the agent supports Codex-style skills. For agents that do not auto-load skills, use the repository as a portable instruction bundle:

1. Point the agent to `SKILL.md`.
2. Tell it to read only the relevant files under `references/`.
3. Use this starter prompt:

```text
Use the Compose Circuit Skills instructions in SKILL.md.
First inspect this repository's Gradle files, Circuit version, Compose version,
module conventions, and existing Screen/Presenter/Ui patterns. Load only the
reference files relevant to the task before implementing or reviewing.
```

### Update

Pull the latest changes from the installed skill directory:

```bash
cd "$HOME/.agents/skills/compose-circuit-skills"
git pull
```

Or, for `$CODEX_HOME` installs:

```bash
cd "${CODEX_HOME:-$HOME/.codex}/skills/compose-circuit-skills"
git pull
```

### Verify Installation

Check that the skill file exists:

```bash
test -f "$HOME/.agents/skills/compose-circuit-skills/SKILL.md" && echo "installed"
```

If you have the Codex skill validator locally, run:

```bash
python3 /path/to/skill-creator/scripts/quick_validate.py \
  "$HOME/.agents/skills/compose-circuit-skills"
```

## Usage

Invoke explicitly:

```text
$compose-circuit-skills Implement a new ProductDetails Circuit screen.

$compose-circuit-skills Review this presenter for state, coroutine, and navigation problems.

$compose-circuit-skills Refactor this Compose ViewModel feature into Slack Circuit.
```

The skill can also be invoked implicitly for Kotlin Android or KMP work involving Slack Circuit, Compose UI, Circuit presenters, Circuit UI tests, navigation, retained state, SubCircuit, accessibility, networking, coroutines, and migration to Circuit.

## Structure

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── accessibility-quality.md
    ├── architecture-state.md
    ├── clean-code-review.md
    ├── compose-ui-performance.md
    ├── coroutines-lifecycle.md
    ├── cross-platform.md
    ├── data-networking.md
    ├── examples.md
    ├── network-resilience-security.md
    ├── source-map.md
    └── testing.md
```

## Validate

Run the Codex skill validator:

```bash
python3 /path/to/skill-creator/scripts/quick_validate.py /path/to/compose-circuit-skills
```

The skill should validate without errors before publishing or updating it.
