# sheriff.config.ts for domain packages

The config is generated from three small tables, so adding a domain or layer is one line.

```ts
import type { SheriffConfig } from '@softarc/sheriff-core';

/** Domain packages and the layers each has (packages/<domain>/<layer>/index.ts). */
const domains: Record<string, readonly string[]> = {
  players: ['provide', 'feature', 'ui', 'data', 'model'],
  replays: ['provide', 'feature', 'ui', 'data', 'model', 'testing'],
  shell: ['feature', 'ui', 'data', 'model'],
};

/** Smart components loaded on their own: packages/<domain>/feature/<name>/index.ts. */
const featureModules: Record<string, readonly string[]> = {
  players: ['players-page', 'player-page'],
};

/** Shared domains: any domain may use them. */
const shared = ['shell'];

/** The domains each domain may use besides itself, the shared ones and simple libraries. */
const uses: Record<string, readonly string[]> = {
  players: ['replays'],
};

const isSpec = (path: string) => path.endsWith('.spec.ts');
const testingInSpecs = ({ to, fromFilePath }: { to: string; fromFilePath: string }) =>
  to === 'type:testing' && isSpec(fromFilePath);

export const config: SheriffConfig = {
  // Every bundle entry, workers included (sheriff verify misses new Worker(new URL(...))).
  entryPoints: {
    web: 'apps/web/src/main.ts',
    'replay-worker': 'packages/import/data/replay.worker.ts',
  },
  modules: {
    ...Object.fromEntries(
      Object.entries(domains).flatMap(([domain, layers]) => [
        [`packages/${domain}`, [`domain:${domain}`, 'type:main']],
        ...layers.map((layer) => [`packages/${domain}/${layer}`, [`domain:${domain}`, `type:${layer}`]]),
      ]),
    ),
    ...Object.fromEntries(
      Object.entries(featureModules).flatMap(([domain, names]) =>
        names.map((name) => [`packages/${domain}/feature/${name}`, [`domain:${domain}`, 'type:feature']]),
      ),
    ),
    'libs/analysers/src': ['type:lib'],
  },
  depRules: {
    root: ({ toTags, fromFilePath }) => !toTags.includes('type:testing') || isSpec(fromFilePath),
    ...Object.fromEntries(
      Object.keys(domains).map((domain) => [
        `domain:${domain}`,
        ({ to }: { to: string }) => {
          const target = to.replace('domain:', '');
          return (
            to === 'type:lib' ||
            target === domain ||
            (!shared.includes(domain) && shared.includes(target)) ||
            (uses[domain] ?? []).includes(target)
          );
        },
      ]),
    ),
    'type:main': [],
    'type:provide': ['type:feature', 'type:ui', 'type:data', 'type:model', 'type:lib', testingInSpecs],
    'type:feature': ['type:feature', 'type:ui', 'type:data', 'type:model', 'type:lib', testingInSpecs],
    'type:ui': ['type:ui', 'type:model', 'type:lib', testingInSpecs],
    'type:data': ['type:data', 'type:model', 'type:lib', testingInSpecs],
    'type:model': ['type:model', 'type:lib', testingInSpecs],
    'type:testing': ['type:testing', 'type:data', 'type:model', 'type:lib'],
    'type:lib': ['type:lib'],
  },
};
```

## TypeScript paths

TypeScript treats workspace packages as external, so the root `tsconfig.json` maps each package
for Sheriff and the editor:

```json
"paths": {
  "@scope/players": ["./packages/players/index.ts"],
  "@scope/players/*": ["./packages/players/*"]
}
```

Map `@scope/<domain>/*` to `packages/<domain>/*`; Sheriff rejects `*/index.ts` targets. Tag a
package-root `index.ts` `type:main`. A package with its own `rootDir` sets `"paths": {}`.

## Known limits (checked 2026-11-02)

- Sheriff 0.20.0 crashes on TypeScript 7.0.2 and works on 6.0.3. Check before moving a Sheriff
  workspace to TypeScript 7.
- Sheriff's `sameTag` and `anyTag` help for packages not yet converted (`'domain:x': [sameTag, 'type:lib']`).
