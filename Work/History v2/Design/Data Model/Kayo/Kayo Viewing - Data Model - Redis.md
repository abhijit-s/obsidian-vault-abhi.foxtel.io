## Proposed
Based on the results of load testing done on a dev MemoryDB cluster, it is concluded that we will go with a json array based data model to store user history, where
- Each individual item in the array corresponds to an asset and its watched progress
- Progress information for each asset is unique within the array

### Rationale
Redis (MemoryDB) is an in-memory DB and has key-value semantics. Naturally, it has some limitations along with the advantages of a key-value data store.

We need to be able to 


## Sample
Sample
