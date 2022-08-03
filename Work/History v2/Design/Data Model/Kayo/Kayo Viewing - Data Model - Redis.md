## Proposed
Based on the results of load testing done on a dev MemoryDB cluster, it is concluded that we will go with a json array based data model to store user history, where
- Each individual item in the array corresponds to an asset and its watched progress
- Progress information for each asset is unique within the array

### Rationale
Redis (MemoryDB) is an in-memory DB and has key-value semantics. Naturally, it has some limitations along with the advantages of a key-value data store.

We need efficient ways of looking up the history and at the same time support flexibility in doing that (since product might have newer requirements coming up from time to time). There were a few choices to consider:

#### 1. Use secondary indexING TO facilitate lookups along with individual history items stored.

There are a couple of ways in which the individual history items could be stored:
- Individual json records
- A single Hash map with all records 
In either of the cases, in order to do lookups, we'd have to utilise auxiliary data structures (in this case sorted sets). These would need to be per the querying requirements, meaning, be able to support the different types of dimensional queries.
- By `asset-id`
- By `sport`
- By `sport` and `series`
- By `sport`, `series` and `team`(s) 
- By `show` 
- By `show` and `season`

| Pros                                                                                                              | Cons                                                                                                                  |
| ----------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| The writer (history-recorder) can do blind writes, without worrying about performing any checked updates/upserts. | The blind writes will need to be done to multiple records - each of the sorted sets and the individual record holder. |
| Sorted sets provide pre-sorted                                                                                    |                                                                                                                       |

Each of these sets would need to be created and maintained for every profile. This causes key proliferation and could prove to be a limitation in the long run. 

#### 2. Store individual history items as their own records and use RediSearch to perform any number of complex queries.



3. Store history as a singular json array.





## Sample
Sample
