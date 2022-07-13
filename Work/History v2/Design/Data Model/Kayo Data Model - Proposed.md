See : [[Data Model - V2]]

#history #design #kayo 

## Proposed

A single data record for a viewing history entry needs to contain 2 sets of fields:
- The viewing history related information
- Any extended set of attributes which assist in performing post-processing over history - aggregation/sorting/filtering.
- A _summary_ list of identifiers (`assetId`s) stored as a separate row for the profile key.

### Queries
The type of lookups into viewing history that we need to support can be classified as below:
- Get entire history.
- Show assets: Lookup using the show, and optionally the season.
- 

### Kayo Data Model

Sample viewing history entry:

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

`rir:Information` This is merely a structural representation using JSON. It would imply associated data types to be used for the DynamoDB item.

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

#### Considerations
- There are a couple of `path.*` fields which would be indexed and used for performing lookups. 
- `path.hierarchy`: used when querying for assets associated with shows (and seasons). 
	- *NOTE* : In order to query using only the show category-id, we can use a `begins_with (a, substr)` expression that can then fetch assets belonging to the same show across season category-ids.
- `path.sport`: used when querying for assets using the sport / series categorisation. 
	- Similar hierarchical querying mechanism can be employed like that with the `path.hierarchy`, i.e. prefix based search to get all assets for the top level (i.e. `sport`)

Additionally, we need to be able to retrieve history using certain other set of dimensions; which could be hierarchical in nature as well.

*Example*: retrieve the viewing history for a particular sport, series and team.

For this purpose we intend to additionally create items in DynamoDB that can assist in such a search.

| Primary Key | Sort Key                  | Value              |
| ----------- | ------------------------- | ------------------ |
| profileId   | /{sport}/{series}/{teamA} | [list of assetIds] |
| profileId   | /{sport}/{series}/{teamB} | [list of assetIds] |
| profileId   | /show/season              | [list of assetIds] |

- Every viewing history update will also incur writes to the above summary rows where the `assetId` will be added to the list if not already present, along with its own item row as above.

`fas:Question` How do we track when an asset changes hierarchy (show/season) using the above design? 


_NOTE_: 
- The whole idea of including pre-computed `path.*` fields is to perform pivoted lookups. We can have LCIs (local secondary indexes) oh these fields and use them for efficient querying for use cases that don't directly deal with `assetId`s (ex: fetch all history for a given sport & series).
- We could potentially embed multiple paths such as these depending on the specific use cases in a product. 


- The `path.sport` could have been expanded to include team-ids (`"path.sports": "/cricket/4/60091,/cricket/4/60092"`), but due to DynamoDB limitations when querying on attributes, it won't prove to be useful. 
- Furthermore, we can't real

Sample summary row of assets:

```json
{
	"assetIds" : [ "1981", "1988", "20034", "10011", .... ]
}
```


Table Design:

![[Data Model - V2 2022-07-11 19.09.56.excalidraw]]


