# Video Node Reference

Source: https://developer.roku.com/docs/references/scenegraph/media-playback-nodes/video.md

**Extends:** Group

The Video node provides controlled playback of live or VOD video. It includes internal nodes for trick play (TrickPlayBar), buffering indicators (ProgressBar), and BIF chapter selection (BIFDisplay).

Starting from Roku OS 8, the system overlay slides in when `*` is pressed while Video node is focused and the app does not handle `OnKeyEvent()`.

---

## Playback Fields

### Content & Playlist

| Field | Type | Default | Access | Description |
|-------|------|---------|--------|-------------|
| `content` | ContentNode | NULL | RW | ContentNode with video metadata. For playlists, children of this node are the playlist items |
| `contentIsPlaylist` | Boolean | false | RW | Enable playlist mode (sequence of videos) |
| `contentIndex` | integer | -1 | RO | Index of currently playing video in playlist |
| `nextContentIndex` | integer | -1 | RW | Set next playlist index. Resets to -1 after taking effect |

### Control & State

| Field | Type | Default | Access | Description |
|-------|------|---------|--------|-------------|
| `control` | string | "none" | RW | Set desired play state |
| `state` | string | "none" | RO | Current play state |
| `asyncStopSemantics` | boolean | false | W | Non-blocking stop (Roku OS 12.5+). State goes "stopping" → "stopped" |

**control options:** `none`, `play`, `stop`, `pause`, `resume`, `replay`, `prebuffer`, `skipcontent`

**state values:** `none`, `buffering`, `playing`, `paused`, `stopping` (OS 12.5+), `stopped`, `finished`, `error`

**prebuffer** — Start buffering before user selects play. Only one stream can prebuffer at a time. Use with `itemFocused` events to eliminate perceived start delay.

### Error Fields

| Field | Type | Description |
|-------|------|-------------|
| `errorCode` | integer | 0=none, -1=network, -2=timeout, -3=generic, -4=empty list, -5=media format, -6=DRM |
| `errorMsg` | string | Error description |
| `errorStr` | string | Diagnostic string: `category:{name}:error:{code}:ignored:{0|1}:{source}:...` |
| `errorInfo` | roAA | Detailed error: clipId, ignored, source, category, errcode, dbgmsg, drmerrcode |

### Timing

| Field | Type | Description |
|-------|------|-------------|
| `playStartInfo` | roAA | Timing measurements (seconds): `total_dur`, `manifest_dur`, `drm_load_dur`, `drm_lic_acq_dur`, `prebuf_dur` plus corresponding `*_start` fields |
| `timeToStartStreaming` | time | Seconds from play command to actual playback start. Valid only when state="playing" |
| `licenseStatus` | roAA | DRM license status: `response`, `status` (HTTP code), `keysystem`, `duration` |

---

## Trickplay Fields

| Field | Type | Default | Access | Description |
|-------|------|---------|--------|-------------|
| `duration` | time | 0 | RO | Video duration in seconds. Valid after playback begins |
| `position` | time | invalid | RO | Current position in stream (seconds). As of OS 9.3, recorded during pause events |
| `positionInfo` | roAA | invalid | RO | Detailed: `audio` (double), `video` (double), `clip_id`, `epoch` (0=relative, 1=UTC) |
| `clipId` | integer | 0 | RO | Clip ID of currently playing track |
| `notificationInterval` | time | 0.5 | RW | Interval (seconds) between position field notifications. 0 = no notifications |
| `seek` | time | invalid | WO | Seek to position (seconds from start, as double) |
| `seekMode` | string | "default" | RW | `"default"` = nearest sync frame, `"accurate"` = exact time if supported |
| `autoplayAfterSeek` | boolean | true | RW | Auto-play after rebuffering |
| `loop` | Boolean | false | RW | Loop video/playlist |

### Timed Metadata

| Field | Type | Description |
|-------|------|-------------|
| `timedMetaDataSelectionKeys` | array of strings | Keys to watch for (use `["*"]` for all EMSG data) |
| `timedMetaData` | roAA | Most recent matching metadata from stream |
| `timedMetaData2` | roAA | Extended: includes `data`, `position` (PTS), `source` ("emsg"/"id3"/"hls"/"unk") |

### Stream Info

| Field | Type | Description |
|-------|------|-------------|
| `streamInfo` | roAA | Current stream: `isUnderrun`, `isResume`, `measuredBitrate`, `streamBitrate`, `streamUrl` |
| `completedStreamInfo` | roAA | Last completed stream: same keys + `isFullResult` boolean |
| `streamingSegment` | roAA | Current segment (DASH/HLS): `segBitrateBps`, `segSequence`, `segStart`, `segUrl`, `segType`/`segTypeStr`, `width`, `height`, `hdrModeStr`, `latency`, `path` |
| `downloadedSegment` | roAA | Just-downloaded segment: `Status`, `SegSequence`, `SegUrl`, `DownloadDuration`, `SegSize`, `SegType`, `BitrateBPS`, `SegStart`, `SegDuration`, `Width`, `Height`, `HdrMode`, `Path` |
| `videoFormat` | string | Current format: "hevc", "mpeg4_15" (H.264), "vp9", etc. |
| `audioFormat` | string | Current format: "aac", "ac3", "eac3", "mp3", etc. |

### Buffering

| Field | Type | Description |
|-------|------|-------------|
| `bufferingStatus` | roAA | `percentage` (int), `isUnderrun` (bool), `prebufferDone` (bool), `actualStart` (time) |

**Seek-to-pause pattern:**
1. Set `content.playStart` to desired time
2. Set `control` = `"prebuffer"`
3. Wait for `bufferingStatus.prebufferDone` = true
4. Check `bufferingStatus.actualStart` if needed
5. Set `control` = `"play"`

### Pause Buffer (Live Video Only)

| Field | Type | Description |
|-------|------|-------------|
| `pauseBufferStart` | time | Beginning of buffered range |
| `pauseBufferEnd` | time | End of buffered range |
| `pauseBufferPosition` | time | Current presentation position |
| `pauseBufferOverflow` | Boolean | Buffer couldn't save all video since pause |
| `pauseBufferEpochOffset` | double | Translate relative time to UTC |

### Decoder Stats

| Field | Type | Description |
|-------|------|-------------|
| `enableDecoderStats` | boolean | Enable `decoderStats` updates |
| `decoderStats` | roAA | `renderCount`, `repeatCount`, `frameDropCount`, `streamErrorCount` |

### Thumbnails

| Field | Type | Description |
|-------|------|-------------|
| `thumbnailTiles` | roAA | HLS/DASH thumbnail tile info (VOD since OS 9.1, live since OS 11.0 with `enableThumbnailTilesDuringLive`) |
| `enableThumbnailTilesDuringLive` | Boolean | Enable thumbnail tiles for live streams (OS 11.0+) |
| `enableLiveAvailabilityWindow` | Boolean | Enable trickplay bar scrubbing during live |

---

## Audio Fields

| Field | Type | Default | Access | Description |
|-------|------|---------|--------|-------------|
| `mute` | Boolean | false | RW | Mute video audio |
| `audioTrack` | string | | RW | Selected audio track identifier |
| `currentAudioTrack` | string | | RO | Actually playing audio track |
| `availableAudioTracks` | array of AA | [] | RO | Available tracks: `Language`, `Name`, `Track`, `HasAccessibilityDescription` (OS 13.0+) |
| `seamlessAudioTrackSelection` | Boolean | false | RW | Continue video during audio switch (HLS only, OS 13.0+). Must set before any control command |
| `supplementaryAudioVolume` | int | 50 | RW | Description track volume (0-100) |

**Automatic audio track selection** — Roku OS selects best track based on device settings (language, country, descriptive). Priority: user explicit selection > preferred language+country+descriptive > preferred language+country > preferred language default > preferred language > first track.

---

## Subtitle / Closed Caption Fields

| Field | Type | Default | Access | Description |
|-------|------|---------|--------|-------------|
| `globalCaptionMode` | string | "Off" | RW | "Off", "On", "Instant replay", "When mute" (Roku TV only) |
| `suppressCaptions` | boolean | false | RW | Suppress captions (e.g., when drawing UI over video) |
| `subtitleTrack` | string | | RW | Selected subtitle track identifier |
| `currentSubtitleTrack` | string | | RO | Actually playing subtitle track |
| `availableSubtitleTracks` | array of AA | [] | RO | Available: `Description`, `Language` (ISO 639-2), `TrackName`, `HasAccessibilityDescription` (OS 13.0+), `HasAccessibilityCaption`, `HasAccessibilitySign` |
| `subtitleSelectionPreferences` | roAA | {} | WO | Priority order for subtitle selection (OS 12.5+) |
| `audioSelectionPreferences` | roAA | {} | WO | Priority order for audio selection (OS 12.5+) |
| `captionStyle` | roAA | system | RW | Style captions: Text/{Font,Effect,Size,Color,Opacity}, Background/{Color,Opacity}, Window/{Color,Opacity} |

### Subtitle/Audio Selection Preferences (Roku OS 12.5+)
```brightscript
video.subtitleSelectionPreferences = {
    values: [
        {language: ["es-419", "es", "es-*", "fr", "en"]},
        {caption: "true"},
        {descriptive: ["false"]},
        {easyReader: "true"}
    ],
    overrideSystem: true
}
```

---

## UI Fields

| Field | Type | Default | Access | Description |
|-------|------|---------|--------|-------------|
| `width` / `height` | float | 0.0 | RW | Video window size (0 = full screen) |
| `enableUI` | Boolean | true | RW | Show/hide built-in UI (progress bar, trick play). When false, app must implement UI. Captions dialog still shows on `*` press |
| `enableTrickPlay` | Boolean | true | RW | Allow trick play buttons (only when enableUI=true) |
| `trickPlayBar` | TrickPlayBar | internal | RW | Customize: `textColor`, `thumbBlendColor`, `filledBarBlendColor`, `trackBlendColor`, `filledBarImageUri`, `trackImageUri`, `currentTimeMarkerBlendColor`, `liveFilledBarBlendColor` |
| `bufferingBar` | ProgressBar | internal | RW | Customize: `width`, `height`, `emptyBarBlendColor`, `emptyBarImageUri`, `filledBarBlendColor`, `filledBarImageUri`, `trackBlendColor`, `trackImageUri`, `percentage` |
| `retrievingBar` | ProgressBar | internal | RW | Initial buffering indicator (same fields as bufferingBar) |
| `bufferingTextColor` | color | system | RW | Buffering text color |
| `retrievingTextColor` | color | system | RW | Retrieving text color |
| `bifDisplay` | BIFDisplay | internal | RW | BIF chapter selection: `frameBgBlendColor`, `frameBgImageUri`, `getNearestFrame` (WO), `nearestFrame` (RO) |
| `pivotNode` | node | - | RW | Custom node displayed when paused |
| `trickPlayBackgroundOverlay` | uri | "" | W | Background overlay when playback UI visible |
| `playbackActionButtons` | array of AA | [] | RW | Pause screen buttons: `text`, `icon`, `focusIcon`, `buttonIsDisabled` |
| `playbackActionButtonSelected` | integer | 0 | RW | Selected button index |
| `playbackActionButtonFocused` | integer | 0 | RW | Focused button index |

---

## CDN Fields (Failover & Load Balancing)

```brightscript
video.cdnSwitch = [
    {URLFilter: "cdn1.example.com", Priority: 1, Weight: 8},
    {URLFilter: "cdn2.example.com", Priority: 1, Weight: 8},
    {URLFilter: "cdn3.example.com", Priority: 2, Weight: 4, ContentFilter: "period-1"}
]
```
Monitor `cdnSwitch` field for CDN change events.

---

## Miscellaneous Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `MaxVideoDecodeResolution` | vector2d | [0,0] | Max decode resolution. Useful when combining 2D APIs with video. 0 = unlimited |
| `cgms` | integer | 0 | Copy guard: 0=none, 1=no more, 2=once, 3=none |
| `enableScreenSaverWhilePlaying` | Boolean | false | Allow screensaver if video <50% of screen |
| `disableScreenSaver` | Boolean | false | Suppress screensaver (for audio-only streams) |
| `contentBlocked` | Boolean | false | Whether current content is blocked (OS 8+) |

---

## Content Meta-Data (set on ContentNode)

Key attributes to set on the ContentNode assigned to `video.content`:

| Attribute | Type | Description |
|-----------|------|-------------|
| `url` | string | Stream URL |
| `streamFormat` | string | "hls", "dash", "smooth", "mp4", "mkv", "ism" |
| `title` | string | Video title |
| `playStart` | time | Start position (seconds) |
| `HttpCertificatesFile` | string | SSL cert file for HTTPS |
| `HttpCookies` | array | Cookies for HTTPS |
| `HttpHeaders` | array | HTTP headers |
| `HttpSendClientCertificates` | boolean | Send client certs |

---

## Usage Example

```xml
<Video id="myVideo" width="1280" height="720" translation="[0,0]" />
```

```brightscript
sub init()
    m.video = m.top.findNode("myVideo")
    m.video.observeField("state", "onVideoState")
    m.video.observeField("position", "onVideoPosition")
end sub

sub playVideo(url as string, title as string)
    content = CreateObject("roSGNode", "ContentNode")
    content.url = url
    content.title = title
    content.streamFormat = "hls"
    m.video.content = content
    m.video.control = "play"
end sub

sub onVideoState(event as object)
    state = event.getData()
    if state = "error"
        print "Error: "; m.video.errorMsg
        print "Code: "; m.video.errorCode
        print "Debug: "; m.video.errorInfo.dbgmsg
    else if state = "finished"
        ' handle completion
    end if
end sub

sub onVideoPosition(event as object)
    position = event.getData()
    ' update UI with current position
end sub
```

### Playlist Example
```brightscript
playlistContent = CreateObject("roSGNode", "ContentNode")
for each item in videoList
    child = playlistContent.createChild("ContentNode")
    child.url = item.url
    child.title = item.title
    child.streamFormat = "hls"
end for
m.video.content = playlistContent
m.video.contentIsPlaylist = true
m.video.control = "play"
```
