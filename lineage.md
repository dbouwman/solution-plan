# Lineage

**In Process*

Goals:
- track a deployed item back to
  - the Solution Template it was deployed from
  - the Solution instance it is related to
  - the original item it was templated from

- track a Solution instance back to
  - the Solution Template it was created from

## Questions
- must these be "searchable" or merely possible?

## Lineage Json in Deployed Items
- stored in `item.properties` and thus not searchable, but we can traverse the ids
```
{
  properties: {
    solutionId: '3ef', // Solution Instance
    solutionTemplateId: 'bc7', // Solution Template deployed from
    sourceItemId: '87c', // original item id
  }
}
```

If they must be searchable, store them in `typeKeywords` as follows:

- `solutionid|3ef` Solution Instance
- `solutionTemplateId|bc7` Solution Template it was deployed from
- `sourceItemId|87c` Original Item Id

*Note* Checking if this is "ok" with the API team
