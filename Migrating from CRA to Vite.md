## Migrate from Create React App (CRA) to Vite
#migrate #cra #vite 
https://www.asserts.ai/blog/migrataion-react-cra-vitejs/

```bash
npm install --save-dev vite @vitejs/plugin-react vite-tsconfig-paths vite-plugin-svgr vite-plugin-checker

npm uninstall react-scripts
```

##### `vite.config.ts`
Create `/vite.config.ts`
```ts
import { defineConfig, } from 'vite';
import react from '@vitejs/plugin-react';
import viteTsconfigPaths from 'vite-tsconfig-paths';
import svgrPlugin from 'vite-plugin-svgr';
import checker from 'vite-plugin-checker';

// https://vitejs.dev/config/
export default defineConfig({
  build: {
    outDir: 'build',
  },
  plugins: [
    react(),
    checker({
      overlay: { initialIsOpen: false },
      typescript: true,
    }),
    viteTsconfigPaths(),
    svgrPlugin(),
  ],
});
```
Maybe add  [[#How to visualize bundle content|rollup-plugin-visualizer]]?


##### `index.html`
Move `src/index.html` to `/`
Remove all `%PUBLIC_URL%` in `index.html`
Add  `<script type="module" src="/src/index.tsx"></script>` to `index.html`

##### `tsconfig.json`
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */,
    "module": "esnext" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */,
    "allowJs": true /* Allow javascript files to be compiled. */,
    "checkJs": true /* Report errors in .js files. */,
    "jsx": "react-jsx" /* Specify JSX code generation: 'preserve', 'react-native', 'react', 'react-jsx' or 'react-jsxdev'. */,
    "noEmit": true /* Do not emit outputs. */,
    "isolatedModules": true /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */,
    "moduleResolution": "node" /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */,
    "baseUrl": "./" /* Base directory to resolve non-absolute module names. */,
    "paths": {
      "components/*": ["src/components/*"]
    } /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */,
    "types": [
      "vite/client",
      "vite-plugin-svgr/client"
    ] /* Type declaration files to be included in compilation. */,
    "allowSyntheticDefaultImports": true /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */,
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,
    "skipLibCheck": true /* Skip type checking of declaration files. */,
    "forceConsistentCasingInFileNames": true /* Disallow inconsistently-cased references to the same file. */
  }
}

```
See [[#Recommended `tsconfig.json` tsconfig vite]]

Create `src/vite-env.d.ts`
```ts
/// <reference types="vite/client" />
```

Change all `process.env.NODE_ENV` to `import.meta.env`
```
// before
if (process.env.NODE_ENV === 'production') { ... }

// after
if (import.meta.env.PROD) { ... }
```

##### `package.json`
```json
// remove all react-script mentions

"scripts": {
  "start": "vite",
  "build": "tsc && vite build",
  "serve": "vite preview"
},
```


