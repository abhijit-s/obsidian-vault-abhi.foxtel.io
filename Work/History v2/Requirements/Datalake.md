As per the [revised design](https://foxsportsau.atlassian.net/wiki/spaces/OM/pages/1092911797/History+API#2.3-Revised-Design), the Datalake component performs the following actions in summary:

-   Send `End` events immediately
-   Windowing period increased to 5mins (from 30s)
-   Send events through even when the asset doesn’t exist in master index (some info is missing in these events)


🏗️  Under construction