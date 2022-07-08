#design #history #kayo

Resume points & history summary will be stored for each profile, 
- Fundamentally at the **ASSET** level. 
- As disparate sets of assets in `IN_PROGRESS` and `COMPLETED` states
- Different products will need to have a modified data model, essentially based on how the data needs to be fetched and presented.

The primary set of attributes for an asset resume point will be:

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
However, this is primarily done because of demands of the read-path. There is a need to group/filter the assets at one of these hierarchical levels (<mark style="background: #FFB86CA6;"> ⚠️ binge specific </mark>)

 For Kayo this might mean having to do the same in a different manner. For now we know Kayo (and perhaps Flash) needs to do it using:
- The hierarchy (shows)
- Other attributes such as `sport`, `series`, `teams[]` or `season` , etc (`rir:Question` TBD )

## Data model design considerations

The governing aspects for the design would be:
- The way the data needs to be viewed/projected. Behind the scenes, this would mean `viewing-api` would facilitate it with a combination of API calls + filters + aggregation + sorting. 
- DynamoDB storage design & API consideration: In order to make fetch/read data efficiently, it is recommend that `GetItem` requests are made instead of `Query` requests (naturally the queries are slower, with the exception that querying using the partition key for all results can be quicker than a table scan).
	- Therefore the attempt is to ensure we strike a balance between making keyed `GetItem` calls and post processing (filtering/sorting/aggregation)
- We can possibly externalise certain pre-lookups (such as arriving at a limited set of asset IDs) to then get specific history items. 

### How can we narrow down the set of assets that we have to get resume points for from DynamoDB?
There can be multiple approaches to this depending on the use case.

Some known use cases (`rir:Information` with potentially more for Kayo and/or Flash):
- Get viewing history for a particular show (in which case the show category ID will be known)
- Get viewing history for a particular show & season (in which case the show & season category IDs will be known)
- `fas:FileImport`  Get the entire viewing history (Resume/Watched) - Special case wherein no identifiers or contextual information is known. This would perhaps be addressed using a two-step approach involving:
	- Fetch all history (`IN_PROGRESS` or `COMPLETED`) for a profile
	- Apply any aggregation/filtering/sorting rules to it

📔  Furthermore, according to a corner case that needs to be handled (as mentioned in [[Indexer#Corner case Duplicate Content with different asset IDs]] ) we may have to extend the data model to incorporate a summary row that contains the list of ALL `assetId`s per `profileId` for which there is a history entry.