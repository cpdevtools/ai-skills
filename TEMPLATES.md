# Skill templates

Two shapes: a **facts** skill says what a technology does in a given version; a **conventions**
skill says how we want it used. A plugin may hold one of each (e.g. NgRx facts beside our NgRx
conventions). Project skills use the conventions shape plus "Where things are" and "Run it".

General rules for both:

- **The description triggers the skill.** Lead with what it covers, name every feature, then say
  when to use it: the detection (package, target framework, config file), the words a user would
  say, the old patterns it replaces, the error codes it explains. Under 1,536 characters.
- **`SKILL.md` stays short** (an index, the gate, the checklist; well under 500 lines). Detail goes
  in `references/<topic>.md`, which loads only when read. Link each reference from `SKILL.md`
  saying when to read it.
- **Section 0 is a gate.** How to check that the skill applies, and what to do when it doesn't
  (don't apply it; don't add the dependency unless asked).
- **Every rule ends with its mistakes.** The value is in what would otherwise go wrong.
- **Tables for choices**, not paragraphs.
- **Keep the why.** A rule with its reason lets the reader judge cases the rule doesn't cover.
- **No dates without a source.** Version-dependent facts name their source and the date checked.

## Plugin layout

```text
plugins/<plugin>/
├── .claude-plugin/plugin.json      name, description, dependencies
└── skills/
    └── <skill>/
        ├── SKILL.md
        └── references/<topic>.md
```

Add the plugin to `.claude-plugin/marketplace.json` with `"defaultEnabled": false` unless it
applies to every project of its platform. Skills appear as `<plugin>:<skill>`, so a conventions
skill is simply named `conventions`.

## A. Facts skill

````markdown
---
name: <tech>-<version>
description: <What it covers, every feature named>. Use when <detection: package / target
  framework / version>, when the user mentions <features>, when replacing <old patterns>, or when
  the build reports <error codes>.
---

# <Tech> <version>

One paragraph: what it ships with, what it is the default for.

| Feature | Syntax / API | Stable? | Details |
| --- | --- | --- | --- |
| … | … | … | [references/<feature>.md](references/<feature>.md) |

## 0. Before using any of it

1. How to check the version (command or file).
2. If the project is older: don't use these features; don't upgrade unless asked.
3. Runtime or configuration it needs.
4. Words or syntax that now break existing code.

## Checklist

- [ ] …

## Diagnostics quick reference

| Diagnostic | Meaning | Fix |
| --- | --- | --- |

## Sources

- <title>: <url> (checked <date>)
````

Each `references/<feature>.md`: the rules, one example, edge cases, then **Mistakes to avoid**.

## B. Conventions skill

````markdown
---
name: conventions
description: The user's conventions for <tech>, layered on <facts skill / upstream skill>. Use
  before planning or writing <what>; where they differ from <defaults>, this wins. Covers <the
  topics, named>.
---

# <Tech> conventions

Layered on: <skills and defaults below this one>. Project rules live in the project's own skill.

## 0. Applies when

<Detection, e.g. `@ngrx/signals` in a package.json.> If absent: don't apply these rules and don't
add the dependency unless asked.

## Overrides

| Topic | Default | Override | Why |
| --- | --- | --- | --- |

## Rules

### <Area>

- The rule, with a short example.

**Mistakes to avoid:** …

## Plan before code

What to state before writing (e.g. which components are smart, what is provided where).

## Conditional skills

| If the repo has | Also use |
| --- | --- |

## Check it

The commands that prove it works (typecheck, lint, test, run).
````
