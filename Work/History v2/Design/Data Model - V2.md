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

| Pros                                                                                                                                                                                                                                                                                      | Cons                                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Provides a 'general' way of achieving the objective and thus can be thought of as a platform approach.                                                                                                                                                                                    | May warrant additional effort to build such a lookup mechanism to narrow down the list of resume points (assets). |
| Can provide flexibility, by means of generalisation, meaning, tomorrow additional lookup methods can be added to facilitate newer requirements, whilst keeping the general flow the same.  Lookup interested list of assetIds -> Fetch resume for those asseets -> Post-process if needed | Additional latency impact depending on the way this is achieved.                                                  |
| There might already be an API we can leverage and utilise for this purpose -> VCC content discovery, which has the ability to fetch a set of assets using different flexible queries (lucene based).                                                                                      | Implementing pagination may or may not be trivial (depends on implementation).                                    |
|                                                                                                                                                                                                                                                                                           | The lookup mechanism (API) will need to be customised per product.                                                |
| | If pagination is to be supported, would make it cumbersome to implement such a feature. |
| | Requires the data model to be customised per product (since looking up viewing history may be different for different products). |


### Approach 2
- Include minimal information (mostly pertaining to the resume point & status) within the history row, but establish a mechanism elsewhere (*prior*) to _reach specific history rows (assets)_ for a given profile. 
- This could be done in any manner that's suitable, such as, having an API that can provide such a service for every use case, or by having some summary/meta information within DynamoDB itself that can help with such a navigation.

🌟   We can potentially leverage the current VCC Discovery API to facilitate such a lookup.
Sample queries using VCC Discovery API:
-   [Fetch all assets for sport=afl](https://kayo.content-discovery.cf.streamotion-prod.vmnd.tv/api/v1/assets/search?query=sport:afl&page%5Bsize%5D=50&page%5Bnumber%5D=1&fields=id,description,category.* "https://kayo.content-discovery.cf.streamotion-prod.vmnd.tv/api/v1/assets/search?query=sport:afl&page%5Bsize%5D=50&page%5Bnumber%5D=1&fields=id,description,category.*")
-   [Fetch all assets for show:1207](https://ares.content-discovery.cf.streamotion-prod.vmnd.tv/api/v1/assets/search?query=(category.path:1207)&page%5Bsize%5D=50&page%5Bnumber%5D=1&extraFields=markers&fields=id,description,category.* "https://ares.content-discovery.cf.streamotion-prod.vmnd.tv/api/v1/assets/search?query=(category.path:1207)&page%5Bsize%5D=50&page%5Bnumber%5D=1&extraFields=markers&fields=id,description,category.*")

| Pros                                                                                                                                                                     | Cons                                                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Provides a 'general' way of achieving the objective and thus can be thought of as a platform approach. | May warrant additional effort to build such a lookup mechanism to narrow down the list of resume points (assets). |
| Can provide flexibility, by means of generalisation, meaning, tomorrow additional lookup methods can be added to facilitate newer requirements, whilst keeping the general flow the same.  `Lookup interested list of assetIds` -> `Fetch resume for those assets` -> `Post-process if needed` | Additional latency impact depending on the way this is achieved. | 
| There might already be an API we can leverage and utilise for this purpose -> VCC content discovery, which has the ability to fetch a set of assets using different flexible queries (lucene based). | Implementing pagination may or may not be trivial (depends on implementation). | 
|  | The lookup mechanism (API) will need to be customised per product. | 


### Approach 3
- As a compromise, we could have combination of Option 1 and Option 2, wherein we partly do pre-lookups to narrow down the resume points (assets) based on the lookup flow and additionally perform post-processing to aggregate/filter/sort as required.

| Pros                                                                                                                                                                     | Cons                                                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Provides the best of both options in a way and a healthy balance could be achieved between the two. | It's not completely generalised, therefore still requires product specific data model customisations. |
| Keeping the post-processing is kept minimal or small enough would imply faster lookups. |  |


## Proposed

A single data record for a viewing history entry needs to contain 2 sets of fields:
- The viewing history related information
- Any extended set of attributes which assist in performing post-processing over history - aggregation/sorting/filtering.
- A _summary_ list of 

### Kayo Data Model

Sample viewing history entry:







#### 🚧   WIP:

- 🤔  Can we look at storing assets using hierarchical keys such that we can make range queries efficiently. See: [DynamoDB: Best Practices for using sort keys to organise data](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-sort-keys.html)
- 🤔  Can we utilise Global or Local Secondary Indexes to provide for dimensional queries (hierarchical perhaps). See: [DynamoDB: General guidelines for secondary indexes in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-indexes-general.html)
- 🤔  Can we use DynamoDB streams and lambdas to perform aggregations? This might be helpful if we want to retain a summary/collection of `assetIds` (and/or `categoryId`s) per profile for query/join purposes. See: [DynamoDB: Using Global Secondary Indexes for materialised aggregation queries](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-gsi-aggregation.html)

See also:
 [DynamoDB: Take advantage of sparse indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-indexes-general-sparse-indexes.html)