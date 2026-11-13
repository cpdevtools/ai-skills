---
name: conventions
description: The user's architecture for pnpm workspaces split into domain packages, each with layers provide, feature, ui, data, model and testing, plus layer-free simple libraries, with the boundaries enforced by Sheriff (sheriff.config.ts). Use before creating, moving or importing between packages, adding a domain, layer, entry point, worker or simple library, naming a package or folder, or when Sheriff or the boundaries check reports a violation. Covers what each layer may hold and import, shared domains and the `uses` map, feature sub-modules for lazy loading, the testing entry point, tsconfig paths, and the names the user rejected.
---

# Domain packages

Layered on `typescript:conventions` (folders by feature) and, for Angular, `angular:conventions`.
Sheriff: <https://sheriff.softarc.io>.

## 0. Applies when

The workspace has a `sheriff.config.ts`, or `packages/<domain>/<layer>/` folders. Don't introduce
the structure into a workspace that doesn't use it unless asked.

## Shape

- **One pnpm package per domain** (`packages/<domain>/`). Its layers are folders, each an entry
  point in the package's `exports` (`@scope/<domain>/<layer>` → `./<layer>/index.ts`). Apps compile
  the packages from TypeScript source. The package root's `index.ts` exports only `VERSION`.
- **Simple libraries** (`libs/<name>/`) have no layers: framework-free code (parsers, analysers,
  which may carry domain meaning) or generic helpers. A main entry plus one entry point per part a
  consumer loads separately (client, worker). They import only other simple libraries.
- **Apps** (`apps/<name>/`) hold only their configuration and root routes.

## Layers (singular names, exactly these)

| Layer | Holds | May import |
| --- | --- | --- |
| `provide` | the featureset: `provide<Domain>Feature()`, routes, navigation, slot registrations, guards; no components | feature, ui, data, model, libs |
| `feature` | **all** smart components, routed and embedded | feature, ui, data, model, libs |
| `ui` | presentational components only (and presentation context, e.g. a team perspective) | ui, model, libs |
| `data` | services, stores, data access | data, model, libs |
| `model` | types, tokens, pure domain functions | model, libs |
| `testing` | test doubles; imported **only from `*.spec.ts`** | |

There is **no `util` layer**: a domain helper goes to the layer matching its role; a helper with no
domain goes to a simple library.

## Domains

- A domain uses itself, simple libraries, the **shared** domains, and the domains its `uses` entry
  lists, one way only. A shared domain uses another only through `uses`.
- Smart components loaded on their own (a routed page, a slot's component) are **feature
  sub-modules**: `feature/<name>/index.ts`, registered with Sheriff, so a lazy page doesn't pull the
  rest of the layer into the initial bundle.
- How the config expresses all this: [references/sheriff-config.md](references/sheriff-config.md).

## Names

- Say "smart component", never "container". Inside a layer, folders are domain nouns.
- Rejected for the smart layer: `containers` (strongly), `connected`, `presenters`, `smart`,
  `view`. "shell" means the app's global UI, not a layer.
- A presenter is allowed as an optional `<name>.presenter.ts` helper beside a ui component, not as
  a layer.

**Mistakes to avoid:**

- Importing your own package by its name from inside it: it duplicates InjectionTokens.
- A `utils/` or `common/` layer.
- Importing `testing` outside specs.
- A worker created with `new Worker(new URL(...))` that isn't in Sheriff's `entryPoints`:
  `sheriff verify` doesn't see it.
- A package's spec config that maps only its direct domain dependencies: it must map their domain
  dependencies too (the whole closure).

## Plan before code

Name the domain and layer of every new file, any new entry point or feature sub-module, and any
change to `uses`.

## Check it

The workspace's boundaries check (Sheriff through ESLint, e.g. `pnpm run check.boundaries`), then
`typecheck`, `lint`, `test`. Editors show Sheriff violations through the ESLint extension.
