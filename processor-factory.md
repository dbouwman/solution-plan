# Processor Factory


## Planned Implementation
Example Repl https://repl.it/@dbouwman/ItemProcessorFactory#index.ts

### Base Class with Static Methods
Type specific processors extend this base, providing their own implementations of the static methods.

```js
// Base class we extend and provide static methods that override
export class BaseProcessor {
  static canConvert(item:IItem):Promise<boolean> {
    return Promise.resolve(false);
  }
  static canDeploy(template:ITemplate):Promise<boolean> {
    return Promise.resolve(false);
  }
  static canUserDeployTemplate(
    user: IUser,
    template:ITemplate,
    authentication: IAuthenticationManager
    ):Promise<boolean> {
    return Promise.resolve(false);
  }
  static convertToTemplate(
    item:IItem, 
    authentication: IAuthenticationManager
    ): Promise<IItemTemplate> {
    return Promise.reject('Type Specific Processor Must override convertToTemplate()');
  }
  static copyResourcesToSolution(
    template: ITemplate,
    targetItemId: string,
    authentication: IAuthenticationManager
    ):Promise<IItemTemplate> {
      return Promise.reject('Type Specific Processor Must override copyResourcesToSolution()');
    }
  static deployFromTemplate(
    tmpl:ITemplate,
    authentication: IAuthenticationManager
    ): Promise<ITemplateOutput> {
    return Promise.reject('Type Specific Processor Must override createFromTemplate()');
  }

  static postProcess(
    item: ITemplateOutput, 
    otherItems:ITemplateOutput[], 
    authentication: IAuthenticationManager
  ): Promise<IPostProcessResult> {
    return Promise.reject('Type Specific Processor Must override postProcess()');
  }
}
```

### Processor Factory function

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