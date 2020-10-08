# Terminology

## Solution 

A group of inter-related items that collectively solve a specific business problem. 

For example, a Hurricane Response Solution may contain:
- one or more feature services with multiple layers for tracking the storm, damage, road closures etc
- a set of webmaps that use the feature services
- a set of web apps that use the webmaps
- one or more Forms that feed data into the feature services
- one or more Dashboards that display status information from the feature services

## Solution Creation

The process of converting one or more items or groups, into an array of `ITemplate` objects which can later be deployed into fully operational items or groups.

## Solution Deployment

The process of converting an array of `ITemplate` objects into a fully operational Solution.


## Solution Item (aka Solution "Instance" Item)

An item of `type: "Solution"` with `typeKeywords: "Solution"`

It's `/data` contains an array of `IDeployedItem` objects, which reference the operational items that were deployed with this Solution.

## Solution Template Item

An item of `type: "Solution"` with `typeKeywords: "Solution", "Template"`

It's `/data` contains an array of `ITemplates` which describe the items that will be created when the Solution is Deployed.

## Type Specific Processor
A Processor is a class with static methods that is responsible for type specific logic to convert from an instance into a template and then back to an instance. Additional details in the [processor api documentation](./processor-api.md)




