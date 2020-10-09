# Interfaces
This document lists the core interfaces for the top level Solution API

## IItem
Represents the common meta information for a Portal Item

We can import this from  [arcgis-rest-js](https://esri.github.io/arcgis-rest-js/api/types/IItem/)

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
  data: ISolutionData;
}
```

## ISolutionData
**Note**: Using the same item type for both the Template and Output content is confusing. Not sure if we want the `templates` in the "Deployed" Solution... currently Solutions replaces the content of the templates array w/ `{id: 'ed0', type: 'Web Map'}` which is difficult to model using types

**CURRENTLY**

```js
interface ISolutionData {
  templates: ITemplate[]; // mix of items and groups
  metadata: object; // This is currently in the Solution items, but does not seem to be used...
}
```

**PROPOSED**

This separated the Item and Group templates, simplifying the Processor interfaces.
This will require a schema transformation for existing solution items.

```js
interface ISolutionData {
  itemTemplates?: IItemTemplate[]; // items to be deployed
  groupTemplates?: IGroupTemplate[]; // Groups to be deployed
  params: object; // parameters needed for the solution - aka indicators
  output?: ITemplateOutput[] // pointers to created items/groups, with refs back to templates
}
```

## ITemplate

Base for `IItemTemplate` and `IGroupTemplate`

```js
interface ITemplate {
  sourceId: string; // id of the original source
  resources: ISolutionResource[] // array of resources associated with this template
}
```

## IGroupTemplate extends ITemplate
Details for groups that need to be created. Stored in `data.groupTemplates` array
```js
interface IGroupTemplate extends ITemplate {
  group: object; // group properties with {{variables}} injected if needed
  itemsToShare: string[]; // id of the items to be shared to this group when they are created
}
```

## IItemTemplate extends ITemplate
This is an entry in the solution's `data.itemTemplates` array

```js
interface IItemTemplate extends ITemplate {
  type: string; // item type
  item: object; // basically an IItem w/o it's Id, and with {{variables}} injected
  data: object ;// the /data of the item
  properties: object; // other type-specific information, could be form, layer schema etc
  dependencies: string[]; // array of item id's that must be deployed before this item
  relatedItems: IRelatedItems[]; // relationship information so we can re-connect during deployment
  relativeCost: number; 
}
```

**Note** `relativeCost` denotes the deployment cost relative to creating a Web Map. i.e. creating a feature service may be 10, but a Hub Page may be a 2. Templates that simply involve creating an item and uploading a thumbnail should have the default value of 1.

## ISolutionResource
This is different from `IItemResourceResponse` in REST-JS, specifically because we need additional information.

When 'in-memory' the sourceUrl's point to the original resource location. When in a Solution Template item, the sourceUrl will point to resources of the Solution Template item.

```js
interface ISolutionResource {
  fileName: string; // file name of the resource itself
  type: SolutionResourceType; // resource type information
  path: string; // folder path used on destination item
  sourceUrl: string; // url where this can be fetched from
}
```

### ResourceType
**TODO**: Determine how we would handle `fakezip` files for the in-memory scenario

```js
type SolutionResourceType = "thumbnail" | "metadata" | "resource" | "data" | "fakezip" | "info"
```

**Note** `fakezip` is used for files with extensions that AGO will not accept, i.e., any extension that is not [".json, .xml, .txt, .png, .pbf, .zip, .jpeg, .jpg, .gif, .bmp, .gz, .svg, .svgz, .geodatabase"](https://developers.arcgis.com/rest/users-groups-and-items/add-resources.htm). By adding ".zip" to the filename, AGO accepts the file, but we need a way to know that this file is not truly a zip file.

**Note** `info` is used for files in an item's `esriinfo` folder that aren't covered by another `SolutionResourceType`, e.g., a Form's `forminfo.json` and `form.webform.json` files.

## ITemplateOutput

Information about the deployed entity (item or group) that is stored in the Solution item

```js
interface ITemplateOutput {
  sourceId: string; // id of the original item but more importantly the id of the template this was created from
  id: string; // id of the deployed item/group
  type: string; // item type
  url?: string; // url of the deployed item/group, if appropriate
  properties?: object; // hash containing details of the created entity. Will be stripped before storing in the Solution
}
```

## IProcessOptions

Generalized options hash for Conversion and Serializing into a Solution Item

```js
interface IProcessOptions {
  authentication: IAuthenticationManager,
  callback?: ProcessCallbackFunc
}
```

## IDeploymentOptions
Extended options used during the deployment of templates

```js
interface IDeploymentOptions extends IProcessOptions {
  variables: object; // hash of properties for template interpolation
  params: object; // additional parameters, i.e. existing folderid
}
```


## IProcessCallback & IProcessStatus

This is the callback function that is passed into the various core api functions, which is used to return status information. This will only communicate information about what the system is currently working on. The final status (success/failure) of a process will be returned via the resolution of the originating promise.

Since we don't know how long converting an item into a set of templates will take (we don't know the depth of the dependency graph) we can not return a `percentDone` number. However, the system will return an `ITranslatableMessage` with information regarding what is being worked on so we can some form of progress to the user.

During the deployment, we know how many templates we are going to process, so we can compute a rough percent complete, and return that along with the message.

Messages will originate either in the core-api functions, or from the results of the type specific processor functions. The `ProcessCallbackFunc` will not be passed into the processor functions.

```js
interface IProcessStatus {
  message: ITranslatableMessage; // Message
  percentDone?: number; // only for deployment, approx. percent complete
}

interface ProcessCallbackFunc { 
  (status: IProcessStatus): void;
}
```

## IPostProcessResult

Indicating the results of the post-processing phase

```js
interface IPostProcessResult {
  status: boolean; // was the process successful?
  reasons: ITranslatableMessage[]; // array of messages specifying why things failed
}
```

## IDeployable

Simple object indicating if a Template can be deployed by a given user.

```js
interface IDeployable {
  username: string; // username
  canDeploy: boolean: // can the user deploy the template?
  reasons: ITranslatableMessage[] // array of messages detailing why the user can't deploy the template
}

// Example
const result = {
  username: 'dbouwman',
  canDeploy: false,
  reasons: [
    {
      id: "SE001",
      message: "Can not deploy workforce project; User is not licensed"
    },
    {
      id: "SE002",
      message: "Can not create shared editing group. User lacks portal:admin:createUpdateCapableGroup privilege"
    }
  ]

} as IDeployable
```

## ITranslatableMessage

```js
interface ITranslatableMessage {
  id: string; // unique key that can be used with a i18n system in a consuming application
  message: string; // simple english message describing the issue
}
```