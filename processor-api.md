# Processor API

A Processor is a class with static methods that is responsible for type specific logic to convert from an instance into a template and then back to an instance.

## Determining if the processor should be used

Separate "conversion" from "deployment" for typing

```js
public static canConvert(
    item: IItem,
  ): Promise<boolean> {...}

public static canDeploy(
    template: ITemplate
  ): Promise<boolean> {...}
```

Given an `IItem` or an `ITemplate`, the processor can inspect the type, typekeywords or anything else about the object to determine if it should process it. Currently this is done via a simple lookup based on `item.type`. This new pattern inverts the pattern allowing the processor itself to make the determination, keeping all the type-specific logic in the same place. More information is in the [Processor Discovery](./processor-discovery.md) document.


## License & Privilege Checking

Used during the deployment process, before starting the actual deployment of template, we run a check if the current user has the licensing / privileges required to deploy the template. 

```js
public static canUserDeployTemplate(
  user: IUser,
  template: ITemplate,
  authentication: IAuthenticationManager
): Promise<IDeployable>
```

## Converting to Template
Main function that converts an item to an array of templates. The function should fetch whatever resources/meta information required to create a template from which the processor can deploy a new copy. It must also return an array of id's of items the item depends on. 

If the item being templated requires a Group, an `IGroupTemplate` should also be returned, and it's `dependencies` should contain the Id's of any items in the group which should also be templated. The core api will handle storing the `IItemTemplate` and `IGroupTemplate` objects in the associated arrays in the `ISolutionData`.

```js
public static convertToTemplate(
    item: IItem,
    authentication: IAuthenticationManager
  ):Promise<ITemplate[]> {...}
```

## Resource Copying

This will be used when persisting an `ITemplate` into a Solution item. This allows for type specific logic/naming to be applied when copying resources from their source group/item, to the solution item.

```js
public static copyResourcesToSolution(
    template: ITemplate,
    targetItem: IItem,
    authentication: IAuthenticationManager
  ):Promise<ITemplate> {...}
```

Returns a new `ITemplate` instance, in which the resource urls have been updated to point to the Solution vs the original item.

## Deploy from Template

Deploy an item or group from a template.

```js
public static deployFromTemplate(
    template: ITemplate, 
    authentication: IAuthenticationManager
  ):Promise<ITemplateOutput> {...}
```

## Deployment Post-Processing
Allows a type specific processor a chance to update other items that were created. Allows for  second-pass interpolation to resolve circular dependencies, sharing etc.

```js
public static postProcess(
  item: ITemplateOutput, 
  otherItems:ITemplateOutput[], 
  authentication: IAuthenticationManager
  ): Promise<IPostProcessResult> {...}`
```

**NOTE** Groups will be handled internally by a `GroupProcessor`, and the `postProcess` phase will be where the sharing calls will be made.


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
    ):Promise<IDeployable> {
    return Promise.resolve(false);
  }
  static convertToTemplate(
    item:IItem, 
    authentication: IAuthenticationManager
    ): Promise<ITemplate[]> {
    return Promise.reject('Type Specific Processor Must override convertToTemplate()');
  }
  static copyResourcesToSolution(
    template: ITemplate,
    targetItem: IItem,
    authentication: IAuthenticationManager
    ):Promise<IItemTemplate> {
      return Promise.reject('Type Specific Processor Must override copyResourcesToSolution()');
    }
  static deployFromTemplate(
    template:ITemplate,
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