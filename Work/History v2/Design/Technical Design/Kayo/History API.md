#history #design #kayo 

## Overview
Service to fetch the viewing history for a user profile.


## Design
- Directly talks to the backing data store, in this case DynamoDB
- Provides a response with raw viewing history items.
- Supports filtering using `assetIds` 
- Supports lookups to fetch viewing history items using category paths (shows & seasons).
- Supports lookups to fetch viewing history items using other dimensional hierarchies (sport, series & team).


## API Endpoints

1. `/viewing/profiles/{profile}` : Get complete viewing history for a `{profile}`
2. 