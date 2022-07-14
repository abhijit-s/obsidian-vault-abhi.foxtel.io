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

1. `/viewing/profiles/{profile}/types/asset`
	- Description: Get complete viewing history for a `{profile}` for assets. Does not perform any kind of aggregation or grouping. 
	- Parameters:
		- `{profile}`: The profile ID
		- `{assets}`: comma-separated list of assets to filter-by optionally
2.  `/viewing/profiles/{profile}/types/show`
	- Description: Get complete viewing history for a `{profile}` for shows. Aggregation done at the show level, which means a single history item is considered to be a show. 
	- Parameters:
		- `{profile}`: The profile ID
		- `{assets}`: comma-separated list of assets to filter-by optionally