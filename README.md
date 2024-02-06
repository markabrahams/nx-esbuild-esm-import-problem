# Nx ESM Import Problem

This is an example repo showing how local ESM imports fail when using the ESM format with the Nx esbuild executor.

## What I have done to set up

1. Generate project and enter project directory
```
npx create-nx-workspace nx-esbuild-esm-import-problem --package-manager=npm --preset=ts --nxCloud=skip
cd nx-esbuild-esm-import-problem
```
2. Add node executor
```
npx nx add @nx/node
```
3. Generate node app
```
npx nx g @nx/node:application my-app --framework=none --tags=type:app --e2eTestRunner=none
```
4. Set up dependency
```
echo 'export const myNumber = 42;' >my-app/src/dependency.ts
sed -i '1s/^/import { myNumber } from ".\/dependency";\n/' my-app/src/main.ts
echo 'console.log("My number: " + myNumber);' >> my-app/src/main.ts
```
5. Switch esbuild format from default CommonJS to ESM
```
sed -i 's/"format": \["cjs"\],/"format": ["esm"],/' my-app/project.json
```
6. Run app
```
npx nx serve my-app
```

## Resulting error
```
mark@mymachine:~/node/nx-esbuild-esm-import-problem$ npx nx serve my-app

> nx run my-app:serve:development

[ watch ] build succeeded, watching for changes...
Debugger listening on ws://localhost:9229/c928aad8-8a6a-496b-97d5-a3dd13ea7399
For help, see: https://nodejs.org/en/docs/inspector

Error: Cannot find module '/home/mark/node/nx-esbuild-esm-import-problem/dist/my-app/dependency' imported from /home/mark/node/nx-esbuild-esm-import-problem/dist/my-app/main.js
    at new NodeError (node:internal/errors:405:5)
    at finalizeResolution (node:internal/modules/esm/resolve:226:11)
    at moduleResolve (node:internal/modules/esm/resolve:838:10)
    at defaultResolve (node:internal/modules/esm/resolve:1036:11)
    at DefaultModuleLoader.resolve (node:internal/modules/esm/loader:251:12)
    at DefaultModuleLoader.getModuleJob (node:internal/modules/esm/loader:140:32)
    at ModuleWrap.<anonymous> (node:internal/modules/esm/module_job:76:33)
    at link (node:internal/modules/esm/module_job:75:36)


>  NX  Process exited with code 1, waiting for changes to restart...
```

## Workarounds

Two workarounds for the import failures are:

1. Append a `".js"` to the import statement in `"main.js"` i.e.:
```
import { myNumber } from "./dependency.js";
```

2. Revert back to the CommonJS esbuild format in `"my-app/project.json"`, although this is a wider change with other effects as well:
```
...
"format": ["cjs"]
...
```
