# ai-skills

A Claude Code plugin marketplace (`cpdevtools`) of layered skills for building TypeScript,
Angular and backend projects.

## Layers

| Layer | What it holds | Where it lives | On when |
| --- | --- | --- | --- |
| 1. Facts | What a language, framework or library version does | a plugin here, or upstream (e.g. Google's `angular-developer`) | that technology is in the repo |
| 2. Universal conventions | How we want a platform used | `typescript`, `angular` | always, for that platform |
| 3. Library conventions | Rules for a library only some projects use | `angular-signal-store`, `angular-material`, `domain-packages`, `cpdevtools-gitflow` | enabled per project |
| 4. Project | One repo's own rules, places and commands | that repo's `.claude/skills/` | sessions in or adding that repo |

Each layer only adds to or overrides the one below it. Library plugins declare the plugins they
build on in `dependencies`, so enabling `angular-signal-store` also enables `angular` and
`typescript`.

## Plugins

| Plugin | Skill | Default |
| --- | --- | --- |
| `typescript` | `typescript:conventions` | on |
| `angular` | `angular:conventions` | per project |
| `angular-signal-store` | `angular-signal-store:conventions` | per project |
| `angular-material` | `angular-material:conventions` | per project |
| `domain-packages` | `domain-packages:conventions` | per project |
| `cpdevtools-gitflow` | `cpdevtools-gitflow:release` | per project |

## Use it

Add the marketplace once (a GitHub repo, or a local checkout):

```bash
claude plugin marketplace add cpdevtools/ai-skills
claude plugin marketplace add /path/to/ai-skills
```

Turn on what a project uses in its checked-in `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "cpdevtools": { "source": { "source": "github", "repo": "cpdevtools/ai-skills" } }
  },
  "enabledPlugins": {
    "angular-signal-store@cpdevtools": true,
    "angular-material@cpdevtools": true,
    "cpdevtools-gitflow@cpdevtools": true
  }
}
```

Project settings come from the folder a session starts in. In a multi-root workspace whose
sessions start in one folder, enable the union of the workspace's plugins in that folder's
`.claude/settings.local.json`.

## Writing skills

Every skill follows one of the two templates in [TEMPLATES.md](TEMPLATES.md). Check a plugin
with `claude plugin validate plugins/<name>`.
