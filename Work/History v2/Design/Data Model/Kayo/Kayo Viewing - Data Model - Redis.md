# Proposed
Based on the results of load testing done on a dev MemoryDB cluster, it is concluded that we will go with a json array based data model to store user history, where
- Each individual item in the array corresponds to an asset and its watched progress
- Progress information for each asset is unique within the array

## Rationale
Redis (MemoryDB) is an in-memory DB and has key-value semantics. Naturally, it has some limitations along with the advantages of a key-value data store.

We need efficient ways of looking up the history and at the same time support flexibility in doing that (since product might have newer requirements coming up from time to time). There were a few choices to consider:

### 1. Use secondary indexING TO facilitate lookups along with individual history items stored.

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

| Pros                                                                                                                                                                      | Cons                                                                                                                                  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| The writer (history-recorder) can do blind writes, without worrying about performing any checked updates/upserts.                                                         | The blind writes will need to be done to multiple records - each of the sorted sets and the individual record holder.                 |
| Sorted sets are sorted using weights, which we can use in order to perform range lookups, or slice the entire set using indexes. This can help immensely with pagination. | Maintaining additional data structures and guaranteeing consistency at the service level can get complicated.                         |
|                                                                                                                                                                           | Insert and Remove are &amp;lt;code&amp;gt;O(log(N))&amp;lt;/code&amp;gt; operations                                                   |
|                                                                                                                                                                           | Can lead to key proliferation on account of maintaining additional sorted sets per profile; which in the future may further increase. |

### 2. Store individual history items as their own records and use [RediSearch](https://redis.io/docs/stack/search/) to perform any number of complex queries.

While this would have been an optimal solution satisfying every type of requirement, unfortunately we don't have the option of using it since AWS MemoryDB does not offer it. 

### 3. Store history as a singular json array.

This was finalised as the approach since it offers the best of both worlds. 
- Keeps keys to a minimum, i.e. single key for the entire history and possibly an additional one for maintaining viewing sessions (to be elaborated further)
- Allows for flexibility in performing searches for items within the array using `jsonpath` as supported by the Redisjson module 




## Sample
Sample




Good reading


https://aws.amazon.com/blogs/database/unlocking-json-workloads-with-elasticache-and-memorydb/