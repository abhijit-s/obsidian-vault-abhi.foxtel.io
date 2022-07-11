#history #requirements #content-api #kayo

## 1. New Query endpoint(s):
* New endpoint to get assets based on `category.path` using vcc discovery
* New endpoint to get assets using a specific attribute (ex: `sport=afl`) using vcc discovery
* Ability in the above endpoints to project payload fields (using the `fields` attribute)

**OR** 
-   General purpose query endpoint (unlike the current one which enriches the content further), which simply passes on queries to VCC in their raw format 

## 2. Existing Endpoint(s) changes:
* Enhancing the GET Asset API to pass on the `category.path` as returned by VCC Discovery API
* Enhancing the GET Asset API to provide marker information ([VCC sample](https://ares.content-discovery.cf.streamotion-prod.vmnd.tv/api/v1/assets/9437?extraFields=markers))