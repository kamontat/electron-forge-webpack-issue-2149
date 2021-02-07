# Simple project for electron-forge webpack issue

This is simple project decribe how [pull/2149][pull-2149] resolve

## Problem

Because electron-force plugin-webpack include `resolve.modules` in default webpack configuration, so it always resolve root node_modules not the dependencies version that libraries depend on. This simple project I use **ajv** as a problem libraries. The **ajv** introduce break changes between major version 6 and version 7 which incompatible to each other.

The dependencies tree will be like below

- node_modules
  - ajv@6.12.6
  - electron-store@7.0.1
    - conf
      - ajv@7.0.4

Since plugin always specify root node_modules so the **ajv@6.12.6** will be used cause application to crash

## Reproduce steps

This repository already setup to reproduce above problem, so you can just clone and run `yarn start` or follow manually steps below

### Manually steps

1. Initial electron-forge project with typescript and webpack

```bash
$ yarn create electron-app electron-forge-webpack-issue-2149 --template=typescript-webpack
```

2. Install **ajv** version `6.12.6`

```bash
$ yarn add ajv@6.12.6
```

3. Install **electron-store** version `7.0.1`

```bash
$ yarn add electron-store@7.0.1
```

4. Add store setup in `createWindow` method in [process/main][process-main] file with schema enabled (this allow **electron-store** to use **ajv** libraries)

```typescript
const store = new Store({
 defaults: {
   hello: "world",
 },
 schema: {
   hello: {
     type: "string",
   },
 },
});

console.log(store.size);
```

5. run development application

```bash
$ yarn start
```

6. You should see error like screenshot below

![error-window][error-window]

## Temporarily fixed

1. open file at **node_modules/@electron-forge/plugin-webpack/dist/WebpackConfig.js**
2. go to return section of the method
3. remove all `resolve` object from default webpack config

```diff
 return (0, _webpackMerge.merge)({
   devtool: 'source-map',
   target: 'electron-main',
   mode: this.mode,
   output: {
     path: _path.default.resolve(this.webpackDir, 'main'),
     filename: 'index.js',
     libraryTarget: 'commonjs2'
   },
   plugins: [new _webpack.default.DefinePlugin(this.getDefines())],
   node: {
     __dirname: false,
     __filename: false
   },
-  resolve: {
-    modules: [_path.default.resolve(_path.default.dirname(this.webpackDir), './'), _path.default.resolve(_path.default.dirname(this.webpackDir), 'node_modules'), _path.default.resolve(__dirname, '..', 'node_modules')]
-  }
}, mainConfig || {});
```

4. start application again `yarn start`
5. application started successfully

<!-- links section -->

[error-window]: ./docs/error-window.png

[process-main]: ./src/index.ts
[pull-2149]: https://github.com/electron-userland/electron-forge/pull/2149