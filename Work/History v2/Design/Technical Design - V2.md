## Considerations
- Fast lookups
- Fast writes
- Scalable

## Choices
#### Database : DynamoDB
##### Why? 
- Very fast writes when using the primary keys (partition key + sort key).
- Fast lookups when using primary keys.
- Scalable and also can provide high throughput since it's distributed.
- Can be autoscaled (on-demand + provisioned capacity).
- Batch operations can help in efficient reads and writes.

##### Limitations
- Data Storage:
	- DynamoDB's limit on the size of each record is 400KB.
	- GSIs are limited to 20 per table.
	- Usage of LSIs on a table imposes a 10 GB size limit per partition key value.
- Data Retrieval: 
	- Scans & Queries are restricted to fetch 1 MB in a single request. Requires pagination if data is not present in the first request's response by using (`NextPageToken` = `LastEvaluatedKey`) in the subsequent request.
- Partition Throughput:
	- Each partition has its own throughput limit, it is set to 3,000 [RCUs (Read Capacity Units)](https://dynobase.dev/dynamodb-pricing-calculator/) and 1,000 [WCUs (Write Capacity Units)](https://dynobase.dev/dynamodb-pricing-calculator/) _per second_. This would allow you to read 12MB of strongly-consistent data or 24MB of eventually-consistent data _per second_, as well as to write 1MB of data per second.
	
- Others
	- Throughput Default Quotas per table - 40,000 read capacity units and 40,000 write capacity units
	- Partition Key Length - from 1 byte to 2048 bytes
	- Sort Key Length - from 1 byte to 1024 bytes
	- Table Name Length - from 3 characters to 255
	- Item's Attribute Names - from 1 character to 64KB long
	- Item's Attribute Depth - up to 32 levels deep
	- ConditionExpression, ProjectionExpression, UpdateExpression & FilterExpression length - up to 4KB
	-  `DescribeLimits` API operation should be called no more than once a minute.

#### Transport
**Message broker**: Kafka
- Topics will be partitioned using the viewing profile ID (Datalake producer's responsibility)
- Dedicated dead-letter retry topic to accomodate any workflows resulting in failure. 
- Number of topic partitions TBD based on current and projected capacity (can go with current configuration - `64`)

#### Services & APIs
Will be split into 2 services:
- **History Indexer** (write-path): 
	- Listens to kafka topic(s) and indexes the resume points, after applying business logic. 
	- Needs to perform quick writes and therefore can use DDB conditional writes for this purpose
	- needs to implement retry mechanisms such that events are not lost.
	- retries can be 
- **Viewing API** (read-path): 
	- Provides APIs necessary to lookup and/or retrieve viewing-history for profiles.
	- Does so by possibly performing additional lookups/co-ordination with other APIs, such as Content-API.
 
##### Framework: 
Spring WebFlux on Kubernetes (Java v17 `rir:Question`)
