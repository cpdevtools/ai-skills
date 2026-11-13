---
name: conventions
description: The user's conventions for NgRx SignalStore (@ngrx/signals) in Angular, layered on angular:conventions. Use before planning or writing any signalStore, signalStoreFeature, withState/withProps/withComputed/withMethods code, or when deciding where shared state lives. Covers global (providedIn root) vs component stores, signalStoreFeature for reused logic, large stores composed as a typed chain of layers (StoreOf) with views and method groups in files by concern, private _ members, and the withProps/withComputed and feature-input pitfalls.
---

# NgRx SignalStore conventions

Layered on `angular:conventions` and NgRx's own documentation (<https://ngrx.io/guide/signals>).

## 0. Applies when

`@ngrx/signals` is in a `package.json`. If it isn't, shared state follows Angular's default (a
service with a private `signal()` exposed `.asReadonly()`); don't add NgRx unless asked.

## Overrides

| Topic | Angular default | Override | Why |
| --- | --- | --- | --- |
| Shared state | a service holding signals | a `signalStore` | one shape for state, views and methods |
| Where it lives | a root service | **global** (`providedIn: 'root'`) for app-wide state; **component stores** (`providers: [...]` on a smart component) for state owned by one page or subtree, created and destroyed with it | state lives as long as its owner |
| Reused logic | copied between services | a `signalStoreFeature` once a second store needs it | no duplication |
| Large stores | one long store file | a typed chain of layers, each concern in its own file ([references/layered-stores.md](references/layered-stores.md)) | small files by concern |

## Rules

### Global or component store

- Ask who owns the state. The whole app: global. One page, one editor, one viewer: a component
  store provided on that smart component, so each instance gets its own and it goes away with it.
- A subtree that needs a different instance of a global store (a draft) provides its own under the
  same token (see component-tree DI in `angular:conventions`).

### Store features

- Stores are a mix of features plus a little of their own (a computed value, a method or two).
  Pull logic into a `signalStoreFeature` when a second store needs it; don't force everything
  into features, and don't duplicate.

### Layered stores

When a store grows past one concern, compose it: a base feature, then one layer per concern, each
typed with `StoreOf`, views and method groups in files by concern. Read
[references/layered-stores.md](references/layered-stores.md) before adding a concern to such a
store.

**Mistakes to avoid:**

- `withComputed` around a layer of plain functions: it turns them into signals. Use `withProps`
  for a layer that returns computed signals and functions.
- A private `_name` member that already exists lower down: it silently replaces the existing one.
- Expecting `_` members in a separately declared feature's input: NgRx strips them. A feature that
  needs a service injects it itself (feature factories run in an injection context).
- `mutate`-style edits: use `patchState` with new objects.

## Plan before code

Say which stores are global and which are component-level, where each component store is
provided, and which features are reused or new.

## Check it

The repo's `typecheck` and `test`; store layers and method groups are plain functions, so test
them directly in `<concern>.spec.ts` beside them.
