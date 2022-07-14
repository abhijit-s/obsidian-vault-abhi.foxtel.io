#history #design #kayo 

## Overview
Service to fetch the viewing history for a user profile.


## Design
- Directly talks to the backing data store, in this case DynamoDB.
- Provides a response with raw viewing history items.
- Supports filtering using `assetIds`.
- Supports lookups to fetch viewing history items using category paths (shows & seasons).
- Supports lookups to fetch viewing history items using other dimensional hierarchies (sport, series & team).


## API Endpoints

1. `/history/profiles/{profile}/facets/asset/{type}?assets={assetIds}`
	- Description: Get complete viewing history belonging to a `{profile}` for assets. Does not perform any kind of aggregation or grouping and retrieves history at the asset level.
	- Parameters:
		- `{profile}` <sup>*required</sup> : The profile ID
		- `{type}` : [all | resume| watched] 
			- `all`  <sup>default</sup>  : Get complete history, both `IN_PROGRESS` and `COMPLETED`
			- `resume` : Get history for resumable assets (i.e. `state=IN_PROGRESS`)
			- `watched` : Get history for completely watched assets (i.e. `state=COMPLETED)
		- `{assetIds}` : comma-separated list of `assetId`s to filter-by optionally.

2.  `/history/profiles/{profile}/facets/show/{showCategoryId}/{type}?season={seasonCategoryIds}`
	- Description: Get viewing history belonging to a `{profile}` for shows. Aggregation done at the show level, which means a single history item is considered to be a show. In order to evaluate history item for a show, all associated assets are considered and certain business rules are applied to arrive at the correct asset.
	- Parameters:
		- `{profile}`: The profile ID
		- `{type}` : [all | resume| watched] 
				- `all`  <sup>default</sup>  : Get complete history, both `IN_PROGRESS` and `COMPLETED`
				- `resume` : Get history for resumable assets (i.e. `state=IN_PROGRESS`)
				- `watched `: Get history for completely watched assets (i.e. `state=COMPLETED)
		- `{seasonCategoryIds}` : comma-separated list of season-`categoryId`s to filter-by optionally.

3. `/history/profiles/{profile}/facets/sport/{sport}/{type}?series={seriesIds}&team={teamIds}
	- Description: Get complete viewing history for a `{profile}` for shows. Aggregation done at the sport level. If  which means a single history item is considered to be a show. 
	- Parameters:
		- `{profile}`: The profile ID
		- `{type}` : [all | resume| watched] 
				- `all`  <sup>default</sup>  : Get complete history, both `IN_PROGRESS` and `COMPLETED`
				- `resume` : Get history for resumable assets (i.e. `state=IN_PROGRESS`)
				- `watched` : Get history for completely watched assets (i.e. `state=COMPLETED)
		- `{seriesIds}`: comma-separated list of series-ids to filter-by optionally. 
		- `{teamIds}`: comma-separated list of team-ids to filter-by optionally.  `{seriesIds}` is _mandatory_ when provided.
