# SGDEX — SceneGraph Developer Extensions

Source: https://github.com/rokudev/SceneGraphDeveloperExtensions

SGDEX provides pre-built, reusable Roku SceneGraph components for rapid development. It manages screen navigation (view stack), content loading (ContentHandler), and follows consistent UX patterns.

## Installation
1. Copy `extensions/SGDEX` → `pkg:/components/SGDEX`
2. Copy `extensions/SGDEX.brs` → `pkg:/source/SGDEX.brs`
3. Add `bs_libs_required=roku_ads_lib` to manifest

---

## Core Architecture

### ComponentController
Manages the view stack (screen navigation). Automatically handles back button, focus, and transitions.

```brightscript
' In your Scene (extends BaseScene)
sub show()
    grid = CreateObject("roSGNode", "GridView")
    grid.content = GetContentHandler()
    m.top.ComponentController.callFunc("show", {view: grid})
end sub
```

### BaseScene
Extends Scene. Provides `ComponentController` and theme support.

### ContentHandler
Async content loader that runs on a Task thread. Create custom handlers by extending it.

```xml
<component name="MyContentHandler" extends="ContentHandler">
  <script type="text/brightscript" uri="MyContentHandler.brs" />
</component>
```

```brightscript
sub GetContent()
    ' m.top.content is the ContentNode to populate
    url = "https://api.example.com/feed"
    raw = m.top.content  ' the ContentNode passed in
    
    request = CreateObject("roUrlTransfer")
    request.SetCertificatesFile("common:/certs/ca-bundle.crt")
    request.InitClientCertificates()
    request.SetUrl(url)
    response = ParseJson(request.GetToString())
    
    for each item in response.items
        child = raw.createChild("ContentNode")
        child.title = item.title
        child.hdPosterUrl = item.thumbnail
        child.url = item.streamUrl
    end for
end sub
```

Assign handler to a view:
```brightscript
content = CreateObject("roSGNode", "ContentNode")
content.HandlerConfigGrid = {
    name: "MyContentHandler"
}
grid.content = content
```

### Content Manager with Custom Views
For views not built on SGDEX, use the Content Manager directly:
```brightscript
content = CreateObject("roSGNode", "ContentNode")
content.addFields({
    HandlerConfigCustom: {
        name: "MyContentHandler",
        fields: {apiUrl: "https://..."}
    }
})
```

---

## Views

### GridView
Grid of content rows with lazy loading.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `content` | ContentNode | Grid content (rows → items) |
| `rowItemFocused` | vector2d | [row, col] of focused item |
| `rowItemSelected` | vector2d | [row, col] of selected item |
| `style` | string | "standard", "hero", "zoom" |
| `posterShape` | string | "16x9", "4x3", "square", "portrait" |
| `overhang` | node | Overhang customization |

Content structure: Root ContentNode → Row ContentNodes (title field) → Item ContentNodes (title, hdPosterUrl, etc.)

### DetailsView
Content detail screen with action buttons.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `content` | ContentNode | Detail content |
| `buttons` | ContentNode | Action buttons (children with title field) |
| `buttonFocused` | integer | Focused button index |
| `buttonSelected` | integer | Selected button index |
| `isContentList` | boolean | If true, content children are browseable items |
| `currentItem` | ContentNode | Currently displayed item (RO) |

### MediaView
Video/audio player with built-in UI. Replaces manual Video node management.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `content` | ContentNode | Media content |
| `control` | string | "play", "stop", "pause", "resume" |
| `state` | string | Current playback state |
| `mode` | string | "video" or "audio" |
| `isContentList` | boolean | Playlist mode |
| `currentIndex` | integer | Current item in playlist |
| `endcardCountdownTime` | integer | Seconds before auto-play next |
| `preloadContent` | boolean | Preload next item |

### CategoryListView
Vertical list of categories, each with rows of content.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `content` | ContentNode | Categories → Rows → Items |
| `categoryFocused` | integer | Focused category |
| `categorySelected` | integer | Selected category |

### SearchView
Search interface with keyboard and results grid.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `query` | string | Current search text |
| `rowItemFocused` | vector2d | Focused result |
| `rowItemSelected` | vector2d | Selected result |
| `hintText` | string | Placeholder text |

Content handler receives query via `m.top.content.query`.

### TimeGridView
EPG-style time-based grid (program guide).

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `content` | ContentNode | Channels → Programs |
| `channelFocused` | integer | Focused channel |
| `programFocused` | roAA | Focused program info |
| `programSelected` | roAA | Selected program info |

### EntitlementView
Subscription/purchase flow.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `isSubscribed` | boolean | Subscription status (RO after handler runs) |

### ParagraphView
Full-screen text display.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `content` | ContentNode | Text content |

### SlideShowView
Image slideshow.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `content` | ContentNode | Images |
| `isContentList` | boolean | Multiple images |
| `control` | string | "play", "pause" |

---

## ButtonBar
Persistent navigation bar across views.

```brightscript
buttonBar = m.top.ComponentController.callFunc("getButtonBar")
buttonBarContent = CreateObject("roSGNode", "ContentNode")
for each label in ["Home", "Search", "Settings"]
    btn = buttonBarContent.createChild("ContentNode")
    btn.title = label
end for
buttonBar.content = buttonBarContent
buttonBar.visible = true
buttonBar.alignment = "top"
buttonBar.observeField("buttonSelected", "onButtonBarSelect")
```

---

## RAFHandler
Roku Ad Framework integration for SGDEX.

```xml
<component name="MyRAFHandler" extends="RAFHandler">
  <script type="text/brightscript" uri="MyRAFHandler.brs" />
</component>
```

```brightscript
sub ConfigureRAF(adIface as object)
    adIface.SetAdUrl(adUrl)
    adIface.SetContentId(contentId)
    adIface.SetContentGenre(genre)
    adIface.SetContentLength(duration)
end sub
```

---

## Theming

SGDEX supports comprehensive theming via the scene's `theme` field.

```brightscript
m.top.theme = {
    global: {
        textColor: "0xFFFFFFFF"
        focusRingColor: "0xFF8810FF"
        backgroundColor: "0x1A1A1AFF"
        busySpinnerColor: "0xFF8810FF"
    }
    GridView: {
        rowLabelColor: "0xCCCCCCFF"
        focusRingColor: "0xFF8810FF"
    }
    DetailsView: {
        actorsColor: "0xCCCCCCFF"
        titleColor: "0xFFFFFFFF"
    }
    MediaView: {
        trickPlayBarFilledBarBlendColor: "0xFF8810FF"
    }
}
```

Theme keys follow the pattern: `{ViewName}.{propertyName}`. Use `global` for app-wide defaults.

---

## View Stack Navigation

```brightscript
' Push a view
m.top.ComponentController.callFunc("show", {view: myView})

' Pop current view (go back)
m.top.ComponentController.callFunc("pop")

' Replace current view
m.top.ComponentController.callFunc("show", {view: newView, replace: true})
```

The ComponentController automatically:
- Manages focus when views are pushed/popped
- Handles back button to pop views
- Cleans up views when they're removed from the stack
