## Pagination
As an improvement, we may need to implement pagination for viewing history lookups. 

#### Why do it?
Because the UI may want to request batches of viewing history, and perform lazy loads depending on the user's scrolling activity. 
Typically, for a case such as the 'Continue Binging' carousel, the viewable carousel length could be about ~10 items, beyond which the user could either want to scroll or not bother. UI can therefore use a paginated approach to render the carousel.

#### HOW to achieve it?
- The recommended approach would be based on using an externalised cursor. This would mean managing pagination as a separate aspect to querying.
- A separate table in DDB can store these cursors. Items in the table can be expired using [TTLs](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html). thus ensuring the *ephemerality* of these cursors.
- Pagination items would be created only _IF_ requested (such as by means of API params - `page` and/or `offset`)
- The pagination key Relevant attributes would be stored in the cursor and then base64 encoded. This base64 encoded value is returned to the 