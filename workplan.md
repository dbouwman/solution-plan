# Solution.js v2 Workplan

## 1 Create Packages - 2 days
In order to keep things clean, we will create three new packages in the mono-repo, and a test harness application. When this project is complete, the other packages will be removed.

### Github Setup
- new `vnext` branch where all this work will be PR'd into

### Helpers Package
- Eventually we will rename this to `common` but that currently exists and has too much baggage to carry forward
- the solution specific interfaces
- shared helper functions that are specific to Solutions
  - more generic helpers should be PR'd to rest-js

### [Core Package](./core-api.md)
- the main orchestration functions
  - depends on `helpers` package

### [Processor Package](./processor-api.md)
- contain the item type specific processors
- contains the [processor discovery](./processor-discovery.md) functions
- depends on the `helpers` package


## 2 Orchstrator Functions - 5 days
With the packages in place, focus in on the main iteration functions and the surrounding "plumbing"

### Implement the core iteration functions
- `convertFromItems` and `deployTemplates`
- use a StubProcessor created specifically for testing

### Add secondary functions
- add the other fns that delegate to those core functions

### Release Tooling
- automation tooling to streamline (single command) the package + release when merging into `vnext`. 
- Release should have version numbers like `1.6.32-alpha`, and assign it a tag via

```
npm publish --tag next
```

which avoids issues of `npm i @esri/solution-core@latest` picking up the alpha stream

At this point we should do something to ensure that the UMD outputs are working - perhaps a simple umd test harness app

## 3 Harness App - 5 days
Before we can dive into work on the processors, we need an environment to start flexing the core, and pick items to run through the system, while seeing intermediate outputs - aka a Test Harness application

- in the /harness folder, create a react app 
- ensures ESM output is viable and optimized
- use create-react-app and react-router and calcite-components
- must be configurable to work against dev/qa/prod
- must have centralized auth (vs on individual routes)
- any dev can add routes as needed - this is purely a development/debugging tool
- first route `/convert` and that should have:
  - input for starting itemId, starting groupId
  - convert button
  - running log showing the callback information
  - running log showing debug logging
  - when complete, show the solution model in formatted json
    - extra credit - show in ace editor


## 4 Refactor Baseline Processors - 5 days
- web app, web map
  - focus is to keep this really simple at this stage, but get some things fully working
  - ensure all functions have separate unit tests, used for coverage
  - ensure suites of functions have integration tests (fetchMock) not included in coverage

- use this to document how to write & test a processor and register it into the system

## 5 Refactor Site Processor - 2 days
- Refactor the Site Processor, used to validate the guide

## 6 Engage other Developers - intermittent over a month
- engage Solutions team, and Hub team to migrate other types
  - Feature Layer, Views, Quick Capture, Form, Page, etc
- engage others to complete this work 

## 7 Persistence Functions - 5 days
- create module exporting the main Persistence function `saveAsSolution`
- add resource copy hooks into the processors

## 8 Live Tests - 5 days
At this stage we should have enough code to start building out e2e style tests. 
- Hub team will add users to our e2e qa orgs specifically for this purpose.
- System will be built in node, and can run locally or in CI
- System will store telemetry information - specifically around resource api

# Can we do this as secondary pipeline in Solution.js?
In a word - No. This project will introduce a number of significant breaking changes so efforts to do this work "in-place" or as a secondary pipeline within the package will incure substantial effort for little near-term gain, and possible long-term pain.

# Milestones

### Oct 31st 
- #1 initial packages 
- #2 core api
- #3 started harness app
- evaluate progress and re-visit milestones

### Nov 20th
- #3 Complete harness app
- #4 Baseline Processor
- #5 Site Processor
- evaluate progress and re-visit milestones

### Dec 18th
- #6 Other processors from other devs
- #7 Persistence to Solution Item
- #8 E2e system
- evaluate progress and re-visit milestones