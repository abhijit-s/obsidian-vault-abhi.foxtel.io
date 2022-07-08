#history #requirements #kayo 


### Corner case: Duplicate Content with different asset IDs
- Duplicate assets: Need another mechanism to update resume points for existing history
- This would be need to be facilitated using:
	- Vimond asset update events that are received
	- Linking the new Vimond asset ID to the old one ( ℹ️ #pending)
	- Either a pull or a push mechanism
	- ❓ TBD: how do we go about scanning DymamoDB and applying the updates to all profiles affected with the ID change
		- 💡 possibly another summary assets row? Makes updates complicated again.

> Essentially when an asset is published, we can get the asset ID of the older asset (same content) and therefore manipulate history for users accordingly 

