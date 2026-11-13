---
name: conventions
description: The user's conventions for TypeScript workspaces (pnpm monorepos, libraries, apps, Node scripts). Use before planning or writing any TypeScript code, tool config, build script, generator or package script; where it differs from a tool's or framework's default, this wins. Covers strict typing, the .ts form of config files, no other languages in a TypeScript workspace, folders by feature with the domain's own nouns, tests beside their files, root verb.noun scripts, generators that write formatter-clean output, and required behaviour built in instead of optional.
---

# TypeScript conventions

Layered on: TypeScript's and Node's defaults. Framework rules sit above this one
(`angular:conventions`); a project's own rules live in its project skill.

## 0. Applies when

The repo has a `tsconfig.json` or TypeScript in a `package.json`. A project in another language by
design (a Python tool) is not a TypeScript workspace, though "Required behaviour" below applies to
everything.

## Overrides

| Topic | Default | Override | Why |
| --- | --- | --- | --- |
| Types | loose where convenient | strict type checking; inference for obvious types; `unknown`, never `any`, for uncertain ones | errors at compile time |
| Tool config and scripts | `.js`, `.cjs`, `.mjs` | the `.ts` form whenever the tool loads one (`release-artifacts.ts`, `vite.config.ts`); YAML is fine where no code is needed | the user always prefers TS |
| Other languages | a Python or shell helper "just for this" | none in a TypeScript workspace: builders, generators and checks are TypeScript behind root scripts | one language per workspace |
| Folder layout | by kind (`services/`, `models/`, `types/`, `utils/`) | by feature, named with the domain's own nouns | the code says what the app is about; Angular's style guide says the same |
| Tests | a `test/` or `__tests__/` folder | beside the file (`foo.spec.ts`); shared test data beside what it's about | found with the code they test |
| Commands people run | `pnpm --filter <pkg> <script>` in the README | a root `package.json` script named `verb.noun` (`generate.protocols`) that delegates with `--filter`; the README documents the root alias | the filter form is awkward to type |
| Generated files | a formatter run afterwards | the generator formats its own output (e.g. `prettier.format` with `resolveConfig`) | regenerating gives the committed bytes; `pnpm fix` afterwards changes nothing |
| Options | a flag for every behaviour | required behaviour is always on; values the data holds are read from it; an override exists only for a named need | every option is one more thing to set wrong |

## Rules

### Folders and names

- Name folders and files after what they're about, using the nouns the domain and its UI use
  (`library/`, `scenes/`, `lights/`), never the kind of code or an invented category (`content/`,
  `stacking/`, `staging/`).
- When unsure of a name, propose the domain's nouns and let the user choose.

**Mistakes to avoid:** `utils/`, `helpers/`, `types/`, `models/`, `services/` folders; abstract
umbrella names coined to hang files under.

### Config files and scripts

- In an ESM package (`"type": "module"`), use `import.meta.dirname`, not `__dirname`. Node's type
  stripping then runs `.ts` scripts directly.
- One-off analysis outside the repo may use anything; whatever is committed or run as a project
  script is TypeScript.

**Mistakes to avoid:** writing `.cjs` because the example did; adding a `.py` builder to a TS repo.

### Root scripts and generators

- Script names are dot-separated `verb.noun` (`generate.protocols`, `serve.maps`, `check.boundaries`),
  not the colon style used inside packages.
- Put formatting inside the generator, not in a package script after it, so it holds when the
  generator runs on its own.

**Mistakes to avoid:** a README that tells people to run `pnpm --filter …`; generators whose
output differs from the committed tree until `pnpm fix` runs.

### Required behaviour

- If a run needs it, it's on, with no switch to forget.
- If it can be read from the data (a file, the game's data, a manifest), read it there; don't ask
  for it as an argument.
- An override exists only for a specific, named need (a diagnostic, an experiment) and stays out
  of the normal path. Tools run start to finish with no arguments where possible.

**Mistakes to avoid:** `--with-x` flags for things every run needs; passing dimensions or ids the
input already contains.

## Plan before code

Say which folders (by feature noun) the change adds, which root scripts it adds, and, for each
option it introduces, the named need it serves.

## Conditional skills

These come from plugins enabled per project. If the repo has the dependency but the skill isn't
loaded, say so and suggest enabling the plugin rather than guessing its rules.

| If the repo has | Also use |
| --- | --- |
| `@angular/core` | `angular:conventions`, with Google's `angular-developer` skill for Angular's defaults |
| `@ngrx/signals` | `angular-signal-store:conventions` |
| `@angular/material` | `angular-material:conventions` |
| `sheriff.config.ts`, or `packages/<domain>/<layer>` folders | `domain-packages:conventions` |
| `.publish/versions.yml` | `cpdevtools-gitflow:release` |

## Check it

Run the repo's root `typecheck`, `lint` and `test` scripts.
