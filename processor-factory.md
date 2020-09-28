# Processor Factory


## Planned Implementation
Example Repl https://repl.it/@dbouwman/ItemProcessorFactory#index.ts

### Base Class with Static Methods
Type specific processors extend this base, providing their own implementations of the static methods.

```js
// Base class we extend and provide static methods that override
export class BaseProcessor {
  static shouldProcess(item:any):boolean {
    return false;
  }
  static convertToTemplate(
    item:IItem, 
    authentication: IAuthenticationManager
    ): Promise<IItemTemplate> {
    return Promise.reject('Type Specific Processor Must override convertToTemplate()');
  }
  static copyResourcesToSolution(
    template: IItemTemplate,
    solutionItem: IItem,
    authentication: IAuthenticationManager
    ):Promise<IItemTemplate> {
      return Promise.reject('Type Specific Processor Must override copyResourcesToSolution()');
    }
  static createFromTemplate(
    tmpl:IItemTemplate,
    authentication: IAuthenticationManager
    ): Promise<IItem> {
    return Promise.reject('Type Specific Processor Must override createFromTemplate()');
  }

  static postProcess(
    item: IItem, 
    otherItems:IItem[], 
    authentication: IAuthenticationManager
  ): Promise<tbd>
}
```

### Processor Factory function

```js
export getProcessorForItem (
  item: IItem
): BaseProcessor {
  // List of all the Processors
  const _processorTypes = [SiteProcessor, PageProcessor];
    // iterate the processors, check if any can handle the item, and return an instance
    const processorToUse = _processorTypes.reduce((a, p) => {
      if (p.shouldProcess(item)) {
        a = p;
      }
      return a;
    }, GenericItemProcessor);
    
    return processorToUse;
}
```