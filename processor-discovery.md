# Processor Discovery

The system needs a means to figure out what Processor should be used for a specific `IItem` or `ITemplate`. We want to encapsulate this decision logic inside the Processor's themselves, and we use "discovery functions" to achive this.

## Planned Implementation
Example Repl https://repl.it/@dbouwman/ItemProcessorFactory#index.ts

### Processor Discovery functions

Using these functions independenty is somewhat inconvenitent in that we have to pass in the `availableProcessors`. However, we expect most consumers to leverage the [class wrappers](./class-wrappers.md) which will internalize this.

```js
export getConversionProcessor (
  item: IItem,
  availableProcessors: IProcessor[]
): BaseProcessor {

  // iterate the processors, check if any can handle the item, and return an instance
  const processorToUse = availableProcessors.reduce((a, p) => {
    if (p.canConvert(item)) {
      a = p;
    }
    return a;
  }, GenericItemProcessor);
  
  return processorToUse;
}

export getDeploymentProcessor (
  template: ITemplate,
  availableProcessors: IProcessor[]
): BaseProcessor {
  // iterate the processors, check if any can handle the item, and return an instance
  const processorToUse = availableProcessors.reduce((a, p) => {
    if (p.canDeploy(item)) {
      a = p;
    }
    return a;
  }, GenericItemProcessor);
  
  return processorToUse;
}
```