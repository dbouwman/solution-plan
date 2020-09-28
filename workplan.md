# Solution.js v2 Workplan

## 1 Create packages and Harness
In order to keep things clean, we will create three new packages in the mono-repo. When this project is complete, the other packages will be removed.

### [Core Package](./core-api)
- the main orchestration functions
- the solution specific interfaces
- shared helper functions that are specific to Solutions
  - more generic helpers should be PR'd to rest-js

### [Processor Package](./processor-api)
- contain the item type specific processors
- depends on the `core` package

### Harness App
- in the /harness folder, create a react app that will act as a debugging harness during development
- ensures ESM output is viable and optimized
- use create-react-app and react-router and calcite-components
- must be configurable to work against dev/qa/prod
- must have centralized auth (vs on individual routes)
- any dev can add routes as needed - this is purely a development/debugging tool

## 2 Refactor Conversion Functions and Processors
- create module exporting the [core api](./core-api) functions
- implement functions, add tests
- focus on the core loop of loading an item, delegating to processor, adding it's dependencies to the list, continuing
  - this needs to be deterministic, performant, and implemented in a manner that other devs can reason about
- at the same time, move existing processor code into new modules, focusing on the conversion functions

## 3 Persistence Functions
- create module exporting the main Persistence function `saveAsSolution`
- add resource copy hooks into the processors

## 4 Refactor Deployment Functions and Processors
- implement the deployment functions in the core package
- again, focus is on streamlining the core orchestration so that it's deterministic, performant and can be reasoned about
- again, delegate all item specific work to the processors
- move exiting processor code into the new modules, focusing on the deployment functions

# Can we do this as secondary pipeline in Solution.js?
This naturally breaks into two parts -  "Conversion" and "Deployment". In theory, we could integrate the two prior to the complete refactor, but it would not "unlock" much, but would bloat the size of the imports in the consuming application.