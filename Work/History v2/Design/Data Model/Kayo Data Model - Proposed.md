
## Proposed

A single data record for a viewing history entry needs to contain 2 sets of fields:
- The viewing history related information
- Any extended set of attributes which assist in performing post-processing over history - aggregation/sorting/filtering.
- A _summary_ list of identifiers (`assetId`s) stored as a separate row for the profile key.

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
	}, 
	"paths" : {
	
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


