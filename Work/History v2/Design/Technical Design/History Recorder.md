#history #design #kayo #indexer

## Recording Viewing History to DynamoDB

![[Viewing History - Write.svg]]

For every viewing history event, depending on the evaluated viewing `state` we will carry out the following actions:
#### `IN_PROGRESS` state:
1. Update/Insert the viewing history item row for `IN_PROGRESS|{assetId}` with the necessary attributes.
2. Update/Insert the summary row for `IN_PROGRESS` -  in order to include the `assetId` of the viewing history event in the set of assets.

#### `COMPLETED` state:
1. Update/Insert the viewing history item row for `COMPLETED|{assetId}` with the necessary attributes. Add the event timestamp to the list of timestamps included in the row `m.timestamps`.
2. Update the summary row for `IN_PROGRESS|{assetId}` (if present) - remove the `assetId` in question from the list.
3. Delete any pre-existing viewing history item row for `IN_PROGRESS|{assetId}` if exists.

## Processing Flow (Sequence Diagram)

![[History Recorder Sequence.svg]]



## Related

Current Design: [[V1 (Current)]]

Future Design: [[V2 (Future)]]