# tsx-test

`packages/esm` is an ESM package that depends on `packages/cjs`, a CJS package.

I get an error if I run the script in `packages/esm` without building `packages/cjs` first:
```
me@here tsx-test % npx tsx packages/esm/src/main.ts

node:internal/modules/run_main:105
    triggerUncaughtException(
    ^
Error [ERR_MODULE_NOT_FOUND]: Cannot find module '/home/me/tsx-test/node_modules/@tsx-test/cjs/dist/public/index.js' imported from /home/me/tsx-test/packages/esm/src/main.ts
    at finalizeResolution (node:internal/modules/esm/resolve:274:11)
    at moduleResolve (node:internal/modules/esm/resolve:859:10)
    at defaultResolve (node:internal/modules/esm/resolve:983:11)
    at nextResolve (node:internal/modules/esm/hooks:748:28)
    at resolveBase (file:///home/me/.npm/_npx/fd45a72a545557e9/node_modules/tsx/dist/esm/index.mjs?1761940550517:2:3744)
    at resolveDirectory (file:///home/me/.npm/_npx/fd45a72a545557e9/node_modules/tsx/dist/esm/index.mjs?1761940550517:2:4243)
    at resolveTsPaths (file:///home/me/.npm/_npx/fd45a72a545557e9/node_modules/tsx/dist/esm/index.mjs?1761940550517:2:4984)
    at resolve (file:///home/me/.npm/_npx/fd45a72a545557e9/node_modules/tsx/dist/esm/index.mjs?1761940550517:2:5361)
    at nextResolve (node:internal/modules/esm/hooks:748:28)
    at Hooks.resolve (node:internal/modules/esm/hooks:240:30) {
  code: 'ERR_MODULE_NOT_FOUND',
  url: 'file:///home/me/tsx-test/node_modules/@tsx-test/cjs/dist/public/index.js'
}

Node.js v24.1.0
```

Build `packages/esm` and try again:
```
me@here tsx-test % cd packages/cjs 
me@here cjs % yarn tsc
me@here cjs % cd ../..
me@here tsx-test % npx tsx packages/esm/src/main.ts
42
```

I added a conditional export to `packages/cjs` to force reading the .ts file, let's try it:

```
me@here tsx-test % rm -rf packages/cjs/dist        
me@here tsx-test % npx tsx --conditions=tsx packages/esm/src/main.ts
/home/me/tsx-test/packages/esm/src/main.ts:1
import { add } from "@tsx-test/cjs"
         ^

SyntaxError: The requested module '@tsx-test/cjs' does not provide an export named 'add'
    at #_instantiate (node:internal/modules/esm/module_job:217:21)
    at async ModuleJob.run (node:internal/modules/esm/module_job:319:5)
    at async onImport.tracePromise.__proto__ (node:internal/modules/esm/loader:663:26)
    at async asyncRunEntryPointWithESMLoader (node:internal/modules/run_main:99:5)

Node.js v24.1.0
```

This sounds like https://github.com/privatenumber/tsx/issues/38 which was marked fixed last year. What am I doing wrong?
