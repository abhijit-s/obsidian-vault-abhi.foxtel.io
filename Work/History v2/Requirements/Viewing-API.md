#history #requirements #viewing-api #kayo
Service which provides the APIs to query and fetch a user's viewing history.

Current implementations:
- Ares:
	- [API-doc](https://ares-viewing-api.content.platform-lb.streamotion-platform-prod.com.au/api-doc/index.html)
	- [Github](https://github.com/fsa-streamotion/streamotion-platform-ares-viewing-api)
- Flash:
	- [API-doc](https://flash-viewing-api.content-flash.platform-lb.streamotion-platform-prod.com.au/api-doc/index.html)
	- [Github](https://github.com/fsa-streamotion/streamotion-platform-flash-viewing-api)


### Requirements (per current design)
_NOTE_: viewing history below could mean either resumable or watched content (or both).
- Ability to fetch a user's (profile) complete viewing history (resumable & watched).
- Ability to fetch a user's (profile) viewing history for specific assets and/or categories.

#### Kayo specific
- Ability to fetch a user's (profile) viewing history for a team.
- Ability to fetch a user's (profile) viewing history for a sport.
- Ability to fetch a user's (profile) viewing history for a league.

#### ARES Specific
- Ability to fetch a user's (profile) viewing history for a show.
- Ability to fetch a user's (profile) viewing history for a season (?)

#### Flash specific
TBD


The viewing history items inherently should be sorted per 
