Resume points & history summary will be stored fundamentally at the asset level. The primary set of attributes will be:

```json
{
    "assetId": "1981",
    "assetType": "tv-episode",
    "totalTime": "PT1H50.04S",
    "state": "In progress",
    "progress": 2518,
    "updateTime": "2021-01-27T23:37:42.155Z"
}
```

`rir:Information`  In Binge we currently also store the necessary hierarchy information which includes fields:

```json
{
    "categoryId": "1446",
    "seasonCategoryId": "1450",
}
```
However, this is primarily done because of demands of the read-path. There is a need to group/filter the assets at one of these hierarchical levels (  binge specific)