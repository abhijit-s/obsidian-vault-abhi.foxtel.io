See : [[Data Model - V2]]

#history #design #kayo #datamodel 

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

DynamoDB doesn't really have a JSON datatype, so we'd have to flatten out the structure like this:
```json
{
	"a.assetId": "1981",    
	"a.assetType": "tv-episode",
	"a.totalTime": 6604,
	"a.state": "IN_PROGRESS",
	"a.progress": 2518,
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

`rir:Information` This is merely a structural representation using JSON. It would imply associated data types to be used for the DynamoDB item.


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


## Writing Viewing History to DynamoDB

![[Drawing 2022-07-13 16.24.34.excalidraw]]
