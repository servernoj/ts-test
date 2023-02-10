# Introduction

This repo mimics the real problem that I am trying to solve. I created this repo for demonstration purposes. 

## Root level

On the root level is a TS project that declares some types and constants. It can be built with `yarn tsc` from the project directory.

## Another TS project, a.k.a. UI

In subdirectory `ui` is another TS project that doesn't import any extra dependencies except for Typescript itself. The `src/index.ts` is meant to use types and constants defined in the root project, but since `ui` project is also meant to have an isolated scope it doesn't import those entities from `src` of the root, but instead from `dist` where their compiled versions are dumped into. For the type import to work we need to re-compile the root project with `yarn tsc -d`. 

To `ui` project uses TS alias to import types and uses relative path to import constants
```` ts
import type {Hello} from '$/types'
import {boo} from '../../dist/types.js'
````

## A Vue3 project with Vite, a.k.a. UI2

Lastly, subdirectory `ui2` contains files of the 3rd project which was initialized with `npm init vue@latest` and has been simplified to include bare minimum from the original boilerplate. The `App.vue` file also does two imports in its `script` section
```` ts
import type {Hello} from '$/types'
import {boo} from '$/types'
````

It also imports type and constant from the root project by analogy to `ui` project. For this kind of import to work the following extra steps were made:
1. installed package `@rollup/plugin-commonjs`
1. modified `vite.config.ts` to import that package as `import commonjs from '@rollup/plugin-commonjs'` and add `commonjs()` to the list of plugins.
1. added alias `'$': fileURLToPath(new URL('../dist', import.meta.url))` into `resolve.alias` to support import of constant with `$`.

# Outcomes

Root level can be compiled from project's root directory with `yarn tsc -d` and run  as `node dist` 

The `UI` project can be compiled with `yarn tsc` from `ui` subdirectory and started as `node dist` (also from `ui` subdirectory)

Lastly, `UI2` project (the one with Vue) can be built from `ui2` with `yarn build` and served to the browser with globally installed `serve` as `serve dist` (also from `ui2` subdirectory

# The Problem

The `UI2` project cannot be started in development mode with `yarn dev`. It throws the following error (once page is open in the browser)
````
yarn dev
yarn run v1.22.19
$ vite

  VITE v4.1.1  ready in 263 ms

  ➜  Local:   http://127.0.0.1:5173/
  ➜  Network: use --host to expose
  ➜  press h to show help
Failed to resolve import "commonjsHelpers.js" from "../dist/types.js". Does the file exist?
6:20:17 PM [vite] Internal server error: Failed to resolve import "commonjsHelpers.js" from "../dist/types.js". Does the file exist?
  Plugin: vite:import-analysis
  File: /Users/simon/Projects/ts-test/dist/types.js:1:35
  1  |  import * as commonjsHelpers from "commonjsHelpers.js";
     |                                    ^
  2  |  import { __exports as types } from "\u0000/Users/simon/Projects/ts-test/dist/types.js?commonjs-exports"
  3  |  
      at formatError (file:///Users/simon/Projects/ts-test/ui2/node_modules/vite/dist/node/chunks/dep-3007b26d.js:41389:46)
      at TransformContext.error (file:///Users/simon/Projects/ts-test/ui2/node_modules/vite/dist/node/chunks/dep-3007b26d.js:41385:19)
      at normalizeUrl (file:///Users/simon/Projects/ts-test/ui2/node_modules/vite/dist/node/chunks/dep-3007b26d.js:39693:33)
      at async TransformContext.transform (file:///Users/simon/Projects/ts-test/ui2/node_modules/vite/dist/node/chunks/dep-3007b26d.js:39826:47)
      at async Object.transform (file:///Users/simon/Projects/ts-test/ui2/node_modules/vite/dist/node/chunks/dep-3007b26d.js:41660:30)
      at async loadAndTransform (file:///Users/simon/Projects/ts-test/ui2/node_modules/vite/dist/node/chunks/dep-3007b26d.js:39466:29)
````


It seems to try but fails to import `import "commonjsHelpers.js" from "../dist/types.js"`, which I'm not sure why...