---
layout: page
title: '"jest" task'
navigation_source: docs_nav
---

This task invokes the [Jest](https://jestjs.io/en/) test framework for unit testing.


## When to use it

We recommend Jest for several reasons:

- **All-in-one**: Unlike frameworks such as `mocha` that require many components to be hooked together, Jest provides a single integrated solution for your: test runner, assertion library, mock/spy API, instrumentation, code coverage, and reporting.  Jest also has first class support for React.

- **Interactive**: Jest supports a "watch mode" for automatically re-running tests whenever a file is saved, plus a [snapshot testing](https://jestjs.io/docs/en/snapshot-testing) that can automatically update assertions when a contract changes.

- **Isolated runtime**: Jest runs web tests in a Node.js environment rather than launching a web browser, and leverages the [Node.js VM](https://nodejs.org/api/vm.html) feature to prevent tests from leaking state.

That said, if for some reason you need to run tests in some other runtime such as an Android client or real web browser, Jest may not be the best bet.


## Config files

There isn't a Heft-specific file for this task.  Heft looks for [Jest's config file](https://jestjs.io/docs/en/configuration) in the standard path  `config/jest.config.json`.  Although Jest itself supports other config file names and even embedding settings in your **package.json** file, Heft only supports the name `jest.config.json`.  Using one standard filename makes it easy to search these files, perform bulk edits, and copy configuration recipes between projects.

Generally your Jest configuration should simply extend Heft's [standard preset](https://github.com/microsoft/rushstack/blob/master/apps/heft/includes/jest-shared.config.json):

**&lt;project folder&gt;/jest.config.json**
```js
{
  "preset": "./node_modules/@rushstack/heft/includes/jest-shared.config.json"
}
```

_**Note:** If you find yourself frequently adding lots of custom settings to this file, please create a GitHub issue and tell us about it.  Our aim is to provide a configuration that minimizes the need for project-specific customizations._


## package.json dependencies

Heft has direct dependencies on the Jest packages that it needs, so you don't need to add Jest to your project's **package.json** file.

Your project should get its typings from `@types/heft-jest` instead of `@types/jest`:

```bash
$ rush add --package @types/heft-jest --exact --dev
```

...and then reference `heft-jest` in your **tsconfig.json** file, like this example:

```js
{
  "extends": "./node_modules/@rushstack/heft-node-rig/profiles/default/tsconfig-base.json",
  "compilerOptions": {
    "types": [
      "heft-jest", // <---- ADD THIS
      "node"
    ]
  }
}
```

## Differences from ts-jest

Internally, Jest supports TypeScript compilation via plugins called [transforms](https://jestjs.io/docs/en/tutorial-react#custom-transformers), which are modeled as a synchronous function that receives a single `.ts` file as input, and returns a `.js` file and `.map` file as its output. The official `babel-jest` transform actually does compile one file at a time, but that approach cannot support language features such as `const enum` that require analyzing imported types.  The `ts-jest` transform solves this problem by performing a full compiler analysis and reusing it each time the transform is invoked, but this won't support other build steps such as preprocessors.  Both `babel-jest` and `ts-jest` also impose a significant performance cost, by invoking the compiler a second time when running tests.

Heft takes a different approach of performing a conventional build and then invoking Jest on the output.  If your build targets a browser runtime, you'll need to use the [emitFolderNameForTests]({% link pages/heft_tasks/webpack.md %}) setting to emit CommonJS outputs in a secondary folder. (Emitting extra files is still significantly faster than invoking the compiler twice.)  Heft's `jest-build-transform.js` does not compile anything itself, but rather returns the output of the full pipeline.

Some helpful examples of Jest techniques can be found in the [heft-node-jest-tutorial](https://github.com/microsoft/rushstack/tree/master/tutorials/heft-node-jest-tutorial) project folder.

> **Important differences when using Jest with Heft:**
>
> - Invoke Jest using the `heft` command line.  Invoking the `jest` command line will successfully run tests, but it will not compile anything.
>
> - Do not add `ts-jest` or `babel-jest` as a dependency for your project.
>
> - Instead of `import { mocked } from "ts-jest/utils";`, use the global `mocked()` function that is provided by `@types/heft-jest`.  Besides this difference, the [API documentation](https://kulshekhar.github.io/ts-jest/user/test-helpers) from `ts-jest` is still applicable to Heft's implementation.
>
> - The `ts-jest` transform will magically "hoist" calls to `jest.mock();`.  Heft does not consider this a best practice. Instead, use the [@rushstack/hoist-jest-mock](https://www.npmjs.com/package/@rushstack/eslint-plugin#rushstackhoist-jest-mock) lint rule to remind developers to manually hoist their calls.  It is enabled by default with [@rushstack/eslint-config](https://www.npmjs.com/package/@rushstack/eslint-config).
>


## Debugging Jest tests

To debug your Jest tests, it's recommended create a VS Code [launch.json file](https://code.visualstudio.com/docs/editor/debugging#_launch-configurations) like this:

**my-app/.vscode/launch.json**
```
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Jest tests",
      "program": "${workspaceFolder}/node_modules/@rushstack/heft/lib/start.js",
      "cwd": "${workspaceFolder}",
      "args": ["--debug", "test", "--clean"],
      "console": "integratedTerminal",
      "sourceMaps": true
    },
  ]
}
```

This launches the full Heft toolchain in your debugger.  The `--debug` switch prevents Jest from being spawned as a separate process.  The `--clean` flag is optional, but fixes an issue where in rare situations Jest's "haste-map" may become corrupted by an aborted run.

To restrict the debugger to run one specific test, you can add the `--test-name-pattern` parameter. (See [here]({% link pages/heft/cli.md %}) for command-line documentation.)  Another option is to use Jest's [test.only()](https://jestjs.io/docs/en/api#testonlyname-fn-timeout) API.


## See also

- [heft-node-jest-tutorial](https://github.com/microsoft/rushstack/tree/master/tutorials/heft-node-jest-tutorial) sample project
- Jest's [API reference](https://jestjs.io/docs/en/api)
- [jest.config.json](https://jestjs.io/docs/en/configuration) documentation
