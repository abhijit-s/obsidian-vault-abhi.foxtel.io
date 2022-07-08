#history #requirements #content-api

**Query endpoint(s):**
    -   New endpoint to get assets based on `category.path` using vcc discovery
    -   New endpoint to get assets using a specific attribute (ex: `sport=afl`) using vcc discovery
    -   Ability in the above endpoints to project payload fields (using the `fields` attribute)

OR 

-   General purpose query endpoint (unlike the current one which enriches the content further), which simply passes on queries to VCC in their raw format