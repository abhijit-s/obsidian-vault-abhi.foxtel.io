## Pagination
As an improvement, we may need to implement pagination for viewing history lookups. 

#### Why do it?
Because the UI may want to request batches of viewing history, and perform lazy loads depending on the user's scrolling activity. 
Typically, for a case such as the 'Continue Binging' carousel, the viewable carousel length could be about ~10-15 items, beyond which the user could either want to scroll or not bother. UI can therefore use a paginated approach to render the carousel.

#### HOW to achieve it?
- The recommended approach would be based on using an externalised cursor. This would mean managing pagination as a separate aspect to querying.
- A separate table in DDB can store these cursors. Items in the table can be expired using [TTLs](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html). thus ensuring the *ephemerality* of these cursors.
- The pagination item key (also the cursor key) would contain relevant attributes (contextual) and will be `base64` encoded. Any additional attributes can be stored as the item value in the table. 
	- **NOTE**: as a part of the design, we may have to store all the 'visited' viewing history items along with the cursor since this would help in discarding the said items in further requests. 
- The pagination item key is returned as a part of the API response, using which the client can make further requests for additional items (ex: `cursor=<base64 encoded cursor>&next=20`)

#### TO NOTE
- The definition of a single item of viewing history would depend on the product's definition. However, simplistically, this would imply the highest level of aggregation applied to the profile's history items when returning the API response (viewing api). Thus, for example, in Binge this would mean that the single item is for a show or a movie.
- Pagination implementation will therefore have to account for such semantics.]


### Deleting History Items
In the future, we want to provide the ability to a user to remove individual items from a user's history. 