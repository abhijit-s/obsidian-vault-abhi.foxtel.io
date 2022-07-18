#history #design #kayo #datamodel 

- [[#Proposed|Proposed]]
- [[#Queries|Queries]]
- [[#Kayo Data Model|Kayo Data Model]]
	- [[#Kayo Data Model#Viewing history item|Viewing history item]]
	- [[#Kayo Data Model#Summary row|Summary row]]
- [[#DynamoDB Table Design|DynamoDB Table Design]]
	- [[#DynamoDB Table Design#Indexes|Indexes]]
- [[#Related Links|Related Links]]

## Proposed

A single data record for a viewing history entry needs to contain 2 sets of fields:
- The viewing history related information
- Any extended set of attributes which assist in performing post-processing over history - aggregation/sorting/filtering.
- A _summary_ list of identifiers (`assetId`s) stored as a separate row for the profile key.

## Queries
The type of lookups into viewing history that we need to support can be classified as below:
- Get entire history.
- Show assets: Lookup using the show -> [season] <sup>#</sup>
- Sports assets: Lookup using the sport -> [series] -> [team] <sup>#</sup>
- Get history for specific `assetId`s.
- ✏️  Get history for specific `categoryId`s. <sup>#</sup>
- ✏️  Get history for specific `assetIds` & `categoryIds`. <sup>#</sup>

<sup>#</sup> : Requires some logic for aggregation of history into designated level.<br>
✏️  : For Kayo this might not be needed as long as we can always have the `assetId` to lookup by.

## Kayo Data Model
### Viewing history item
Sample viewing history item for an `asset` under a `profileId`:

A json representation
```json
{
	"attr": {
		"assetId": "1981",    
		"assetType": "tv-episode",
		"totalTime": 6604,
		"state": "IN_PROGRESS",
		"progress": 2518,
		"progressPercentage": 38.12,
		"updateTime": "2021-01-27T23:37:42.155Z"
	},
	"ext" : {
		"showCategoryId": "1446",
		"seasonCategoryId": "1450",
		"sport": "cricket",
		"series-id": "4",
		"team-ids": "60091,60092"
	}
}
```

DynamoDB doesn't really have a JSON datatype, so we'd have to flatten out the structure like this (`rir:Information` This is merely a structural representation using JSON. It would imply associated data types to be used for the DynamoDB item):
```json
{
	"a.assetId": "1981",    
	"a.assetType": "tv-episode",
	"a.totalTime": 6604,
	"a.state": "IN_PROGRESS",
	"a.progress": 2518,
	"a.progressPercentage": 38.12,
	"a.updateTime": "2021-01-27T23:37:42.155Z",
	"e.showCategoryId": "1446",
	"e.seasonCategoryId": "1450",
	"e.sport": "cricket",
	"e.series-id": "4",
	"e.team-ids": "60091,60092",
	"path.hierarchy": "/1446/1450"
	"path.sport": "/cricket/4"
}
```

🆕  For `COMPLETED` viewing history items, we will modify the data structure slightly to include a list of timestamps which indicate the times when the asset was completely watched.

```json
{
	"a.assetId": "1981",    
	"a.assetType": "tv-episode",
	"a.totalTime": 6604,
	"a.state": "COMPLETED",
	"a.progress": 6530,
	"a.progressPercentage": 98.87,	
	"a.updateTime": "2021-01-27T23:37:42.155Z",
	"e.showCategoryId": "1446",
	"e.seasonCategoryId": "1450",
	"e.sport": "cricket",
	"e.series-id": "4",
	"e.team-ids": "60091,60092",
	"path.hierarchy": "/1446/1450"
	"path.sport": "/cricket/4",
	"m.timestamps" ["2021-01-20T11:20:00.155Z", "2021-01-27T23:37:42.155Z"]
}
```

#### Detailed Explanation
- Dimensional lookups: There are a couple of `path.*` fields included, which are done with the sole purpose of assisting certain dimensional lookups. These will be indexed (LSIs).
- `path.hierarchy`: used when querying for assets associated with shows (and seasons). 
	- We will employ a prefix-based search, in order to collate viewing history items at the upper levels of the hierarchy. That is, In order to query using only the show category-id, we can use a `begins_with (a, substr)` expression that can then fetch assets belonging to the same show across season category-ids.
- `path.sport`: used when querying for assets using the sport / series categorisation. To filter for teams further, we can simply utilise the `e.team-ids` field to filter in-memory.
	- Similar hierarchical querying mechanism can be employed like that with the `path.hierarchy`, i.e. prefix based search to get all assets for the upper levels (i.e. `sport`).

###### NOTE:
- The `path.sport` could have been expanded to include team-ids (`"path.sports": "/cricket/4/60091,/cricket/4/60092"`), but due to DynamoDB limitations when querying on attributes (cannot do a `contains()` and is restricted to `begins_with()`), it won't prove to be useful. 

### Summary row
Sample summary row of assets:

```json
{
	"assetIds" : [ "1981", "1988", "20034", "10011", .... ]
}
```

## DynamoDB Table Design
![[Kayo Data Model - Table Design.excalidraw]]

- `PartitionKey` will always be the `profileId`.
- `SortKey` for the summary row will either have `IN_PROGRESS` or `COMPLETED`, with each of them containing the set of `assetId`s in that respective state.
- `SortKey` for the viewing history item row will be a concatenation of the state and the `assetId`. Eg:
	- `IN_PROGRESS|10021`
	- `COMPLETED|20012`

### Indexes
Since DynamoDB limits the number of Local Secondary Indexes (LSI)  on a table, we will restrict them to the below fields:
- `path.hierarchy`
- `path.sport`


### Limiting Factors
#### Data Storage:
- DynamoDB's limit on the size of each record is 400KB.
	- If we intend to store the history as a list (as is done currently,) then, this implies:
		- we'd have to assess and restrict the number of history items that can be contained in a single record/row, and
		- also ensure that the shape of the individual items within the record list is known and kept constrained.
	- Therefore we'd have to go with a data model that stores history items individually (per asset) for a given profile.


| New Column               | New Column                                | Disadvantages                                                           |
| ------------------------ | ----------------------------------------- | ----------------------------------------------------------------------- |
| History as a List        | GETs and PUTs become simpler and quicker. | History will have to be processed in memory (similar to current design) |
| Individual History Items |                                           |                                                                         |
|                          |                                           |                                                                         |
|                          |                                           |                                                                         |
|                          |                                           |                                                                         |


#### SECONDARY INDXES
- LSIs are limited to 5 per table.
	- Given that we want to support lookups on the history 
- GSIs are limited to 20 per table.
	- This doesn't really affect our solution.

- GSIs are limited to 20 per table.
- Usage of LSIs on a table imposes a 10 GB size limit per partition key value.

- Data Retrieval: 
	- Scans & Queries are restricted to fetch 1 MB in a single request. Requires pagination if data is not present in the first request's response by using (`NextPageToken` = `LastEvaluatedKey`) in the subsequent request.
- Partition Throughput:
	- Each partition has its own throughput limit, it is set to 3,000 [RCUs (Read Capacity Units)](https://dynobase.dev/dynamodb-pricing-calculator/) and 1,000 [WCUs (Write Capacity Units)](https://dynobase.dev/dynamodb-pricing-calculator/) _per second_. This would allow you to read 12MB of strongly-consistent data or 24MB of eventually-consistent data _per second_, as well as to write 1MB of data per second.



## Related Links
[[Data Model - V2]]