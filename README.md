# mobile-dev-skills

A Claude Code plugin marketplace bundling **Android** and **iOS** engineering skills. Drop it into Claude Code and your assistant gains opinionated, production-grade rules for writing and reviewing mobile code.

## What's inside

This repo is a **marketplace** that exposes two installable plugins:

### `android`
| Skill | Purpose |
| --- | --- |
| `android-coroutines` | Structured concurrency, lifecycle-safe coroutine patterns. |
| `code-review` | Clean architecture, SOLID, Jetpack Compose review checklist. |
| `material-3` | Material Design 3 / Material You for Compose, Flutter, web. |
| `unit-test` | MockK + `coroutines-test` + Turbine testing rules. |

### `ios`
| Skill | Purpose |
| --- | --- |
| `xcode-build` | Build the iOS project via `xcodebuild`. |
| `run-tests` | Run iOS unit/UI tests via `xcodebuild`. |

---

## Install

You install via the Claude Code CLI. Inside any Claude Code session:

```text
/plugin marketplace add canergulgec/mobile-dev-skills
```

That registers this GitHub repo as a marketplace. Then install the plugin(s) you want:

```text
/plugin install android@mobile-dev-skills
/plugin install ios@mobile-dev-skills
```

To browse / manage interactively:

```text
/plugin
```

To update later:

```text
/plugin marketplace update mobile-dev-skills
```

To remove:

```text
/plugin uninstall android@mobile-dev-skills
/plugin marketplace remove mobile-dev-skills
```

> Note: the `material-3` skill is a git submodule. If you clone this repo directly (instead of installing through the marketplace), run `git submodule update --init --recursive`.

---

## Injecting it into an existing Android project

You don't need to copy any files into your app. Skills live in Claude's config, not in your project.

1. `cd` into your Android project.
2. Launch Claude Code: `claude`.
3. Run the two commands above (`/plugin marketplace add …` then `/plugin install android@mobile-dev-skills`).
4. Ask Claude something like *"review this PR against the android code-review skill"* or *"write unit tests using the unit-test skill"* — Claude will pull the relevant skill in automatically.

Skills auto-activate based on their description (e.g. mentioning Compose pulls in `material-3`), but you can also invoke them explicitly.

---

## Scopes: user vs project

Claude Code has two main configuration scopes. Knowing the difference matters when you want a whole team to share these skills.

### User scope (a.k.a. global / "app scope")
- Lives in `~/.claude/settings.json` and `~/.claude/` data dirs.
- Applies to **every project** you open with Claude Code on this machine.
- This is where `/plugin install …` writes by default — installing once enables the plugin everywhere.
- Best for: solo use, or skills you personally always want available.

### Project scope
- Lives in `.claude/settings.json` inside the repo (commit it to git).
- Applies **only when Claude is launched from that project**.
- You can pin which marketplaces and plugins the project expects, e.g.:

  ```json
  {
    "extraKnownMarketplaces": {
      "mobile-dev-skills": {
        "source": { "source": "github", "repo": "canergulgec/mobile-dev-skills" }
      }
    },
    "enabledPlugins": {
      "android@mobile-dev-skills": true
    }
  }
  ```

  Teammates who open the repo in Claude Code get prompted to trust and install the marketplace automatically — no manual setup per developer.
- Best for: team repos where everyone should use the same review/testing rules.

### Local project scope
- Lives in `.claude/settings.local.json`, **gitignored**.
- Same shape as project scope but personal — good for opt-in tweaks you don't want to commit.

Rule of thumb: **install user-scope for yourself, declare project-scope for your team.**

---

## Repo layout

```
.
├── .claude-plugin/
│   └── marketplace.json        # marketplace manifest (lists the two plugins)
└── plugins/
    ├── android/
    │   └── skills/
    │       ├── android-coroutines/
    │       ├── code-review/
    │       ├── material-3/     # git submodule
    │       └── unit-test/
    └── ios/
        └── skills/
            ├── run-tests/
            └── xcode-build/
```

## Contributing

PRs welcome — add a new skill under the matching `plugins/<platform>/skills/<name>/SKILL.md` and it'll be picked up automatically.

## License

See individual skill folders for any upstream licenses (e.g. `material-3` follows its source repo).
