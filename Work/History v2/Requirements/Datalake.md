As per the [revised design](https://foxsportsau.atlassian.net/wiki/spaces/OM/pages/1092911797/History+API#2.3-Revised-Design), the Datalake component performs the following actions in summary:

-   Send `End` events immediately.
-   Windowing period increased to 5mins (from 30s).
-   Send events through even when the asset doesn’t exist in master index (some info is missing in these events).

Events (player log events) originating from the UI (and which make their way through to Datalake) have certain `client` attributes which can help us to categorise and use these events in a certain manner

Sample event:

```json {
    "eventName": "encrypted_player_log_event",
    "originator": "web",
    "originatorId": "auth0|6170c2ea4f291b00683b079b",
    "subProfileId": "e646dfa412d8fe92a412587dca371e5d6069bf80",
    "versions":
    [
        "1.0"
    ],
    "tenant": "ares",
    "logData": "3156077514B8570E3AE0B7AAF54956E6551AA76A3A6B2D271013A45BC7B70E18E3DDC0D3A6B71E8FBFF83C887A6F645F8559FF313EDE3F728CB407B5472E4D4808E7FB74056588B810F702008C504DC1061500FC19088A2F97850DC3581AC6BB339AA760C47975BEB15FF7943BA11209405E9DA5384B5E98B5060015EDE896D1078FBED268C2247C31E95C53A3ED48FD7912A116A1EE494C5281E7689C88C30E537D01E2F5174E9C984B64871B7C75B32226A422A7D7800FF9108E49E069DB2B879B9E000EB7CB2513611068037A13E527FBF679D46AF5F332AE436B1C539BF8DBE6616B976183A8B40FD8C63CF2A982",
    "client":
    {
        "buildName": "ares-web",
        "buildVersion": "pt12.5.0-aw4.4.0",
        "deviceId": "24b37fa1-3548-4b37-88e0-2a289b1d2ba5",
        "drm": "Widevine",
        "envPlatform": "MacIntel",
        "envVersion": "5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36",
        "pageUrl": "https://binge.com.au/shows/show-silicon-valley!1207",
        "platform": "web",
        "playerEvent": "end",
        "playerState": "pause",
        "streamUrl": "https://d1iljf5fj3nl9e.cloudfront.net/out/v1/0db7f59a24c64b298b2c2239d60cf633/2717d11c878c47bb8aa56d210929c3f6/4cfe24eabd8b44d798ba60d9b62082d7/index.mpd",
        "userAgent": "Amazon CloudFront",
        "videoFormat": "application/xml+dash",
        "videoProtocol": "https",
        "viewingSession": "24b37fa1-3548-4b37-88e0-2a289b1d2ba5"
    },
    "progress":
    {
        "assetId": "513",
        "categoryId": "1208",
        "title": "Bad Money",
        "playbackType": "vod",
        "eventNumber": 23,
        "vod":
        {
            "duration": 1606.079,
            "position": 651
        }
    },
    "timestamp": "2022-07-11T04:26:14.784Z",
    "eventInterval": 30
}
```

| playerEvent | playerState | Received when                        |
| ----------- | ----------- | ------------------------------------ |
| period      | playing     | Video is playing                     |
| period      | pause       | Video is paused                      |
| end         | pause       | Player exited (midway through video) |

- `End` events (`playerEvent='end' & playerState='pause'`) can be detected simply using the `playerEvent` attribute, and thus allows Datalake to send them immediately.
- Similarly, the `Paused` events (`playerEvent='period' & playerState='pause'`) can be sent through immediately since those are an indication that the player has been explicitly paused.
- Each new video playing trigger (start playing or resume) causes a new session to be created (given by the `viewingSession` in the event)

🏗️  Under construction