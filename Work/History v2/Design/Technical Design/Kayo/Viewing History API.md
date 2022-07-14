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
	- Description: Get complete viewing history for a `{profile}` for assets. Does not perform any kind of aggregation or grouping and retrieves history at the asset level.
	- Parameters:
		- `{profile}` <sup>*required</sup>: The profile ID
		- `{type}` : [all | resume| watched] 
			- `all` : Get complete history, both `IN_PROGRESS` and `COMPLETED`
			- `resume`: Get history for resumable assets (i.e. `state=IN_PROGRESS`)
			- `watched`: Get history for completely watched assets (i.e. `state=COMPLETED)
		- `{assets}`: comma-separated list of assets to filter-by optionally
2.  `/history/profiles/{profile}/facets/show/{showCategoryId}/{type}?season={seasonCategoryIds}`
	- Description: Get complete viewing history for a `{profile}` for shows. Aggregation done at the show level, which means a single history item is considered to be a show. 
	- Parameters:
		- `{profile}`: The profile ID
		- `{type}` : [all | resume| watched]
	3. `/history/profiles/{profile}/facets/sport/{sport}/{type}?series={series-ids}&team={team-ids}
		- `{type}` : [all | resume| watched]