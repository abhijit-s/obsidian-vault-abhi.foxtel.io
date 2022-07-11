## Considerations
- Fast lookups
- Fast writes
- Scalable

### Choices
#### Database : DynamoDB
##### Why? 
- Very fast writes when using the primary keys (partition key + sort key)
- Fast lookups when using primary keys
- Scalable and also can provide high throughput since it's distributed
- Can be autoscaled (on-demand + provisioned capacity)

##### Limitations
- Data Storage - Items (or rows) cannot exceed 400KB.
- Data Retrieval: 
	- Queries are restricted to fetch 1 MB in a single request


