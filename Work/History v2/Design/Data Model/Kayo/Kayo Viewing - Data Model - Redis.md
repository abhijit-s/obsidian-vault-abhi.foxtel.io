## Proposed
Based on the results of load testing done on a dev MemoryDB cluster, it is concluded that we will go with a json array based data model to store user history, where
- Each individual item in the array corresponds to an asset and its watched progress
- Progress information for each asset is unique within the array

The user history will be split into 2 sub-sets of json arrays, each for:
- `FINISHED` (completely watched assets)
- `IN_PROGRESS` (resumable assets)



