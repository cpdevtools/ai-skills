---
name: conventions
description: The user's conventions for Angular apps and libraries, layered on Angular's defaults (Google's angular-developer skill, the angular-cli MCP best practices, the repo's CLAUDE.md). Use before planning or writing any Angular component, directive, service, store or page; where they differ, this wins. Covers separate .ts/.html/stylesheet files, smart and presentational components, component-tree dependency injection, data sources passed as inputs, extending components with child directives, CSS custom properties for size-dependent styling, and plain attribute names instead of data- attributes.
---

# Angular conventions

Layered on: Angular's defaults, from Google's `angular-developer` skill, the `angular-cli` MCP
server (`get_best_practices`) and the repo's CLAUDE.md, and on `typescript:conventions`. Follow
them except where a rule below replaces them. Library rules: `angular-signal-store:conventions`,
`angular-material:conventions`. A project's own rules live in its project skill.

## 0. Applies when

`@angular/core` is in a `package.json`.

## Overrides

| Topic | Angular default | Override | Why |
| --- | --- | --- | --- |
| Template and styles | inline `template`/`styles` for small components | separate `.ts`, `.html` and stylesheet (`templateUrl`, `styleUrl`). Inline only for tiny components (a handful of template lines, few or no styles); when in doubt, separate | the user dislikes TS, template and styles in one file |
| Component roles | single-responsibility components | smart and presentational components wherever it fits (below) | composable views; loading kept in one place |
| Shared state | a service holding a private `signal()`, exposed `.asReadonly()` | depends on the project: with `@ngrx/signals`, `angular-signal-store:conventions`; without it, Angular's default, and don't add NgRx unless asked | not every project uses NgRx |
| Context and isolation | DI as a way to reach services | component-tree DI first: a token with a root default that a subtree overrides with its own providers | context comes from where a component sits |
| Lazy data inside a view | the view injects a service | a data-source object the smart component passes in as an input (like CDK `DataSource`) | presentational pieces never inject services |
| Size-dependent styling | per-case CSS, media queries | CSS custom properties set from bindings or resize observers, used by the component's stylesheet and children | one mechanism, scoped to the component |
| Custom attributes | `data-*` | a plain attribute name (`theme`, `row-index`; `[attr.theme]`, CSS `[theme="red"]`) | "you don't do that in Angular" |

## Rules

### Smart and presentational components

The pattern is Lars Gyrup Brink Nielsen's Model-View-Presenter for Angular. Call them "smart" and
"presentational" components; the user rejects "container" as a name.

- **Presentational:** data in through `input()`, user actions out through `output()`; no services,
  stores, fetching or app state; only isolated UI state (hover, a camera).
- **Smart:** a *component*, not a directive. It takes an id or route param, injects services and
  stores, loads, keeps the state, passes the model down and turns child events into state changes.
  Its template is the children and bindings, with no presentation of its own.
- **Extending** a presentational piece: child directives placed inside it that hook into it (extra
  layers on a viewer), not more options on the parent.
- A presentational piece may have a `<name>.presenter.ts` helper beside it for its logic.

**Mistakes to avoid:** a directive that loads data onto a presentational component; a
presentational component that injects a store "just for one value"; toolbars or controls nobody
asked for.

### Dependency injection

- Isolation and context come from providers: a page that needs a draft provides its draft store
  under the same token the app's published store uses, and everything below reads the draft.
- Prefer a provider override to a global setter plus a release on destroy.

**Mistakes to avoid:** handing state into a component and taking it back on destroy; a global
"current X" setter where a subtree provider would do.

### Attributes

- Pick a name no component input or directive selector uses: an input of the same name would
  capture a static attribute.
- Names that merely contain "data-" (`data-views`, `data-grid`) are fine.

**Mistakes to avoid:** `[attr.data-theme]`, `[data-row-index]`, `data-` in specs or generators.

## Plan before code

Before writing an Angular feature, say:

1. which components are smart and which are presentational;
2. the injector tree: what is provided where;
3. where state lives (with NgRx: which stores are global and which component-level, which store
   features are reused or new);
4. the files each component gets.

## Conditional skills

| If the repo has | Also use |
| --- | --- |
| `@ngrx/signals` | `angular-signal-store:conventions` |
| `@angular/material` | `angular-material:conventions` |
| `sheriff.config.ts`, or domain packages | `domain-packages:conventions` |

If the dependency is there but the skill isn't loaded, say so and suggest enabling the plugin.

## Check it

The repo's `typecheck`, `lint` and `test`, then look at the change in the running app.
