# Layered stores

A large store is composed, not monolithic. The store file holds only the chain; each concern's
views and methods live in their own file beside it.

## The `StoreOf` helper

One small file in the project (e.g. `editing/store-of.ts`):

```ts
import type {
  SignalStoreFeature,
  SignalStoreFeatureResult,
  StateSignals,
  WritableStateSource,
} from '@ngrx/signals';

/** What a store feature makes: its state, props and methods. */
type MadeBy<F extends () => SignalStoreFeature> =
  ReturnType<F> extends SignalStoreFeature<never, infer O extends SignalStoreFeatureResult>
    ? O
    : never;

/** A store as a feature makes it, as what is computed over it sees it. */
export type StoreOf<F extends () => SignalStoreFeature> = StateSignals<MadeBy<F>['state']> &
  MadeBy<F>['props'] &
  MadeBy<F>['methods'];

/** The same, as its methods see it: its state writable too (`patchState`). */
export type WritableStoreOf<F extends () => SignalStoreFeature> = StoreOf<F> &
  WritableStateSource<MadeBy<F>['state']>;
```

## The store file: only the chain

```ts
const editorBase = () =>
  signalStoreFeature(
    withState<EditorState>(initialState),
    withSharedFeature(),
    withProps(() => ({ _draft: inject(DraftStore), _ui: inject(EditorUiStore) })),
  );
export type EditorBase = StoreOf<typeof editorBase>;

const withScene = () => signalStoreFeature(editorBase(), withProps(sceneViews));
export type WithScene = StoreOf<typeof withScene>;

const withCameras = () => signalStoreFeature(withScene(), withProps(cameraViews));
export type WithCameras = StoreOf<typeof withCameras>;

/** The store as its methods see it: everything computed under them, its state writable. */
export type EditorMethodsStore = WritableStoreOf<typeof withCameras>;

export const EditorStore = signalStore(
  withCameras(),
  withMethods((store) => {
    const w = editorWrites(store);
    return { ...sceneMethods(store, w), ...cameraMethods(store, w) };
  }),
  withHooks({ onInit(store) { /* effects */ } }),
);
```

## A concern's file (`cameras.ts`)

```ts
/** Views: a plain function of the layer below, returning computed signals and functions. */
export function cameraViews(store: WithScene) {
  const look = computed(() => /* … */);
  return { look };
}

/** Methods: a plain function of the writable store and the shared writes. */
export function cameraMethods(store: EditorMethodsStore, w: EditorWrites) {
  return {
    moveCamera(to: CameraPose) { w.changeCamera(to); },
  };
}
```

## Shared writes (`editor-writes.ts`)

Helpers several method groups use, plus any mutable state they share (as a holder object):

```ts
export function editorWrites(store: EditorMethodsStore) {
  return { changeCamera(to: CameraPose) { patchState(store, { /* … */ }); } };
}
export type EditorWrites = ReturnType<typeof editorWrites>;
```

## Rules

- A new concern is a new layer (views) or method group in its own file, never more code in the
  store file.
- Each layer's views take the type of the layer *below* (`WithScene` for `cameraViews`), so the
  order of the chain is the order of dependence.
- A value a later layer needs but the public shouldn't see becomes a private `_name` member;
  never reuse a name that already exists lower down.
- Layers use `withProps`, not `withComputed`.
- Views and methods are plain functions: test them directly, beside their file.
