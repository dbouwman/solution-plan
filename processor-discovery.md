# Processor Discovery

The system needs a means to figure out what Processor should be used for a specific `IItem` or `ITemplate`. We want to encapsulate this decision logic inside the Processor's themselves, and we use "discovery functions" to achieve this.

## Planned Implementation
Example Repl https://repl.it/@dbouwman/ItemProcessorFactory#index.ts

### Processor Discovery functions

TODO: Review other patterns for this - perhaps using a factory class which we inject the available processors into, allowing more specific applications to work with a subset of the processors...

```js
export getConversionProcessor (
  item: IItem
): BaseProcessor {
  // List of all the Processors
  // Not wild about this being a fixed list vs injected
  const _processorTypes = [SiteProcessor, PageProcessor];
  // iterate the processors, check if any can handle the item, and return an instance
  const processorToUse = _processorTypes.reduce((a, p) => {
    if (p.canConvert(item)) {
      a = p;
    }
    return a;
  }, GenericItemProcessor);
  
  return processorToUse;
}

export getDeploymentProcessor (
  template: ITemplate
): BaseProcessor {
  // List of all the Processors
  // Not wild about this being a fixed list vs injected
  const _processorTypes = [SiteProcessor, PageProcessor];
  // iterate the processors, check if any can handle the item, and return an instance
  const processorToUse = _processorTypes.reduce((a, p) => {
    if (p.canDeploy(item)) {
      a = p;
    }
    return a;
  }, GenericItemProcessor);
  
  return processorToUse;
}
```
