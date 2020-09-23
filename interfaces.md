# Interfaces
This document lists the core interfaces for the top level Solution API

## IItem
Represents the common meta information for a Portal Item

We can import this from  arcgis-rest-js

```js
interface IItem {
  id: string;
  title: string;
  type: string;
  categories?: string[];
  created: number;
  culture?: string;
  description?: string;
  documentation?: string;
  extent?: number[][];
  modified: number;
  numViews: number;
  owner: string;
  properties?: any;
  protected?: boolean;
  size: number;
  snippet?: string;
  spatialReference?: ISpatialReference;
  tags: string[];
  tags?: string[];
  typeKeywords?: string[];
  url?: string;
  [key: string]: any; // allow more properties
}
```

## ISolution

```js
interface ISolution {
  item: IItem;
  data: ISolutionData
}
```

## ISolutionData
**Note**: Using the same item type for both the Template and Output content is messy. Not sure if we want the `templates` in the "Deployed" Solution... currently Solutions replaces the content of the templates array w/ `{id: 'ed0', type: 'Web Map'}` which is super messy to model

```js
interface ISolutionData {
  templates: ITemplate[];
  // metadata: object; // This is currently in the Solution items, but does not seem to be used...
  params: object; // parameters needed for the solution - aka indicators
  output?: ITemplateOutput[] // pointers to created items/groups, with refs back to templates
}
```


## TemplateType
**Note**: Not sure if we really need this... we could move the `type` property up one level and it's an IGroupTemplate when `.type === "Group"`. Seems less typescripty, but this seems overkillish for how common Groups are.

```js
type TemplateType = "ITEM" || "GROUP";
```

## ITemplate
Base for IItemTemplate and IGroupTemplate

```js
interface ITemplate {
  id: string; 
  templateType: TemplateType; // "ITEM" || "GROUP" 
  properties: object; // other type-specific information, could be form, layer schema etc
  resources: ISolutionResource[] // array of resources associated with this template
}
```

## IGroupTemplate extends ITemplate
Details for groups that need to be created
```js
interface IGroupTemplate extends ITemplate {
  group: object; // group properties
  itemsToShare: string[]; // id of the items to be shared to this group when they are created
}
```

## IItemTemplate extends ITemplate
This is an entry in the solution templates array
```js
interface IItemTemplate extends ITemplate {
  type: string; // item type
  item: object; // basically an IItem w/o it's Id, and with {{variables}} injected
  data: object ;// the /data of the item
  properties: object; // other type-specific information, could be form, layer schema etc
  dependencies: string[]; // array of other templates this template depends on
  relatedItems: IRelatedItems[]; // relationship information so we can re-connect during deployment
  groupsToShareWith: string[]; // array of groupids the resulting item should be shared to
}
```

## ISolutionResource

When 'in-memory' the sourceUrl's point to the original resource location. When in a Solution Template item, the sourceUrl will point to resources of the Solution Template item.

```js
interface ISolutionResource {
  fileName: string; // file name of the resource itself
  type: ResourceType; // resoruce type information
  path: string; // folder path used on destination item
  sourceUrl: string; // url where this can be fetched from
}
```

### ResourceType
**TODO**: Check with Mike T re: when `fakezip` is used, and how we would handle that for the in-memory scenario

```js
type SolutionResourceType = "thumbnail" | "metadata" | "resource" | "data" | "fakezip"
```


## IDependency

```js
interface IDependency {
  sourceId: string;
  type: string;
  params: object; // hash of type specific properties
}
```

## ITemplateOutput
Information about the deployed entity (item or group) that is stored in the Solution item
```js
interface ITemplateOutput {
  sourceId: string; // id of the original item but more importantly the id of the template this was created from
  id: string; // id of the deployed item/group
  type: string; // 
  url?: string; // url of the deployed item/group, if appropriate
  properties?: object; // hash containing details of the created entity. Will be stripped before storing in the Solution
}
```