# Uniwind × React Native Strict TypeScript API — repro

Uniwind's `className` prop types disappear when a project opts into React Native's
[Strict TypeScript API](https://reactnative.dev/docs/strict-typescript-api) via the
`react-native-strict-api` custom condition.

## Reproduce

```bash
npm install
npm run typecheck
```

Fails with (both call sites in `src/app.tsx`):

```
error TS2322: ... Property 'className' does not exist on type ...
```

Remove `"react-native-strict-api"` from `customConditions` in `tsconfig.json` and
`npm run typecheck` passes — same code, legacy types.

## Why

The strict tree (`react-native/types_generated/`) exports `ViewProps`, `TextProps`, etc.
as generated **type aliases** (`Readonly<Omit<...>>`). `uniwind/types.d.ts` adds its
`className`/`*ClassName` props via `declare module 'react-native'` **interface merging**
into those names — which cannot merge into a type alias, so the augmentation becomes an
orphan interface and the props silently vanish (`skipLibCheck` suppresses the collision
error inside the `.d.ts`). Same mechanism as uni-stack/uniwind#627, which fixed it for
`VirtualizedListWithoutRenderItemProps` — but it affects the whole core-component surface.

- `uniwind` 1.11.0 · `react-native` 0.86.2 (the strict tree becomes the default in 0.87) · `typescript` 5.9
