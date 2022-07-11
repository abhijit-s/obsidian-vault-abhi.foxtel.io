#design #history #kayo


## Summary
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

## How can we narrow down the set of assets that we have to get resume points for from DynamoDB?
There can be a couple of different approaches to this depending on the use case (or across all use cases)

Some known use cases (`rir:Information` with potentially more for Kayo and/or Flash):
1. Get viewing history for a particular show (in which case the show category ID will be known)
2. Get viewing history for a particular show & season (in which case the show & season category IDs will be known)
3. `fas:FileImport`  Get the entire viewing history (Resume/Watched) - Special case wherein no identifiers or contextual information is known. This would perhaps be addressed using a two-step approach involving:
	- Fetch all history (`IN_PROGRESS` or `COMPLETED`) for a profile
	- Apply any aggregation/filtering/sorting rules to it

### Approach 1
- Ensure that we have enough information within the history row so that we can 'post-process' in order to achieve the desired viewing history output. This would mean embedding additional attributes which would then facilitate aggregation/filtering/sorting of the retrieved history rows. 

| Pros                                                                                                                                                                     | Cons                                                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Provides a 'general' way of achieving the objective and thus can be thought of as a platform approach.                                                                   | We'd have to build support for performing such a lookup to get the list of candidate assets to retrieve history.                                                  |
| Provides flexibility. Any required attributes are readily available so slicing/dicing/aggregation can be done in any number of manners as supported by those attributes. | Requires storage of additional attributes within the history row, thus increasing the size of the row.                                                            |
| Simpler to implement. Even on the write-path, we just have to 'enrich' the history enough to include the additional attributes.                                          | In the future, if demand for such additional attributes increases, it would increase the size of the data row.                                                    |
|                                                                                                                                                                          | In general, this would mean we can fetch the entire viewing history of the profile as a first step and then post-process (similar to existing design) every time. |
|                                                                                                                                                                          | If pagination is to be supported, would make it cumbersome to implement such a feature.                                                                           |

### Approach 2
- Include minimal information (mostly pertaining to the resume point & status) within the history row, but establish a mechanism elsewhere (*prior*) to _reach specific history rows (assets)_ for a given profile. 
- This could be done in any manner that's suitable, such as, having an API that can provide such a service for every use case, or by having some summary/meta information within DynamoDB itself that can help with such a navigation.

| Pros | Cons |
| ---- | ---- |
| Provides a 'general' way of achieving the objective and thus can be thought of as a platform approach. |      |


 
## Efficient Querying in DynamoDB

From the looks of it, it seems we may have to build a hierarchy evaluation mechanism - 
- either using the table design, or 
- externalising it (another API perhaps)


#### 🚧   WIP:

- 🤔  Can we look at storing assets using hierarchical keys such that we can make range queries efficiently. See: [DynamoDB: Best Practices for using sort keys to organise data](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-sort-keys.html)
- 🤔  Can we utilise Global or Local Secondary Indexes to provide for dimensional queries (hierarchical perhaps). See: [DynamoDB: General guidelines for secondary indexes in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-indexes-general.html)
- 🤔  Can we use DynamoDB streams and lambdas to perform aggregations? This might be helpful if we want to retain a summary/collection of `assetIds` (and/or `categoryId`s) per profile for query/join purposes. See: [DynamoDB: Using Global Secondary Indexes for materialised aggregation queries](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-gsi-aggregation.html)

See also:
 [DynamoDB: Take advantage of sparse indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-indexes-general-sparse-indexes.html)