# Item Processor API

## Determining if the processor should be used for an item

`public static shouldProcess(item: IItem): Promise<boolean>`

Given an IItem, the processor can inspect the type, typekeywords or anything else about the item to determine if it should process it. Currently this is done via a simple lookup based on item.type. This pattern inverts the problem while also allowing the processor itself to inspect the item and determine if it should process it.

## Converting to Template

`public static convertToTemplate(item:IItem, authentication: IAuthenticationManager):Promise<IItemTemplate>`

Main function that converts an item to a template. The function should fetch whatever resources/meta information required to create a template from which the processor can deploy a new copy. It must also inspect and return an arrays of item dependencies, and group dependencies. 

For groups, the processor can indicate if the group should simply be created, or if the group content should be included in the Solution.


## Resource Copying
This will be used when persisting an IItemTemplate into a Solution item. This allows for type specific logic/naming etc to be applied when copying resources from their source item, to the solution item

`public static copyResourcesToSolution(template: ITemplate, authentication: IAuthenticationManager):Promise<IItemTemplate>`

Returns an updated IItemTemplate, in which the resource urls have been udpated to point to the Solution vs the original item

## Deploy from Template

`public static createFromTemplate(template: ITemplate, authentication: IAuthenticationManager):Promise<ISolutionItem>`

TODO: Decide what the return from this is


## Deployment Post-Processing
Allows a type specific processor a chance to update other items that were created. Allows for updating of id's, sharing etc

`public static postProcess(item: IItem, otherItems:IItem[], authentication: IAuthenticationManager): Promise<???>`

TODO: Decide if this is basically fire-and-forget (i.e. `void`), or it returns a Promise for a simple `{success: <bool>}`, or it returns a complex object.


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