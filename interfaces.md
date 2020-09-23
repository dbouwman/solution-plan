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

## ISolutionMetadata 

## ISolution

```js
interface ISolution {
  item: IItem;
  data: ISolutionData
}
```

## ISolutionData
```js
interface ISolutionData {
  templates: ITemplate[];
  // metadata: object; // This is currently in the Solution items, but does not seem to be used...
  params: object; // parameters needed for the solution - aka indicators
}
```

## TemplateType
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
{
  fileName: string; // url where this resource can be fetched from
  type: ResourceType; // filename to store on destination item
  path: string; // folder path used on destination item
  sourceUrl: string; // url where this can be fetched from
}
```

### ResourceType
```js
type SolutionResourceType = "thumbnail" | "metadata" | "resource" | "data" | "fakezip"
```


## ISolutionData

```js
{
  templates: IItemTemplate[],
  metadata: object; 
}
```


## ITemplateOutput
Information about the deployed entity (item or group) that is stored in the Solution item
```js
{
  id: string; // id of the deployed item
  type: string; // type of the deployed item
  url?: string; // url of the deployed item, if appropriate
  properties?: object; // hash containing details of the created entity. Will be stripped before storing in the Solution
}
```