# SceneGraph Reference

Source: https://developer.roku.com/docs/references/scenegraph/

SceneGraph is Roku's UI framework. Apps are built as component trees using XML markup + BrightScript.

---

## XML Elements

### `<component>`
Defines a custom SceneGraph component.
```xml
<component name="MyScreen" extends="Group">
  <interface>
    <!-- public fields -->
  </interface>
  <script type="text/brightscript" uri="MyScreen.brs" />
  <children>
    <!-- child nodes -->
  </children>
</component>
```
- `name` — unique component name
- `extends` — parent component (Group, Task, Scene, ContentNode, etc.)
- Components inherit all fields and behavior from parent

### `<interface>`
Declares public fields and functions.
```xml
<interface>
  <field id="title" type="string" value="default" onChange="onTitleChanged" />
  <field id="items" type="array" />
  <field id="selectedIndex" type="integer" value="-1" alwaysNotify="true" />
  <function name="doSomething" />
</interface>
```
- `onChange` — callback function when field value changes
- `alwaysNotify` — notify observers even if value doesn't change
- Field types: `string`, `integer`, `float`, `boolean`, `vector2d`, `color`, `uri`, `node`, `array`, `assocarray`, `time`, `nodeuri`

### `<script>`
Links BrightScript code to the component.
```xml
<script type="text/brightscript" uri="MyComponent.brs" />
<script type="text/brightscript" uri="pkg:/source/utils.brs" />
<script type="text/brightscript">
<![CDATA[
  sub init()
    ' inline script
  end sub
]]>
</script>
```
Multiple script tags allowed. All scripts share the same scope within the component.

### `<children>`
Declares child nodes created when the component is instantiated.
```xml
<children>
  <Label id="titleLabel" text="Hello" font="font:MediumBoldSystemFont" />
  <Poster id="bgImage" width="1920" height="1080" />
  <LayoutGroup id="layout" layoutDirection="vert" itemSpacings="[20]">
    <Label id="item1" text="First" />
    <Label id="item2" text="Second" />
  </LayoutGroup>
</children>
```

---

## Component Functions

### `init()`
Called when a component is created. Use for initialization:
```brightscript
sub init()
    m.titleLabel = m.top.findNode("titleLabel")
    m.top.observeField("focusedChild", "onFocusChange")
    m.titleLabel.text = "Welcome"
end sub
```
- Runs on the **render thread**
- `m.top` refers to the component's top-level node
- `m.top.findNode(id)` finds child nodes by their `id` attribute
- Set up field observers here

### `onKeyEvent(key as String, press as Boolean) as Boolean`
Handles remote control key presses:
```brightscript
function onKeyEvent(key as string, press as boolean) as boolean
    handled = false
    if press
        if key = "OK"
            handleSelect()
            handled = true
        else if key = "back"
            handleBack()
            handled = true
        else if key = "options"
            showOptions()
            handled = true
        end if
    end if
    return handled  ' true = consumed, false = bubble up
end function
```

**Key names:** `OK`, `back`, `up`, `down`, `left`, `right`, `options`, `play`, `pause`, `rewind`, `fastforward`, `replay`

**Important:** Return `true` to consume the event, `false` to let it bubble up to parent components. For the `back` key, returning `false` eventually exits the app.

---

## Key Nodes

### Scene (extends Group)
The root node of every app. Only one Scene per app.
```xml
<component name="MainScene" extends="Scene">
```
Fields: `backgroundURI` (string), `backgroundColor` (color), `backExitsScene` (boolean), `dialog` (node — set to show a dialog)

### ContentNode (extends Node)
Universal data container. All lists, grids, and Video nodes consume ContentNode trees.
```brightscript
content = CreateObject("roSGNode", "ContentNode")
content.title = "My Video"
content.url = "https://example.com/video.m3u8"
content.streamFormat = "hls"
content.description = "A description"
content.hdPosterUrl = "https://example.com/poster.jpg"

' For lists/grids, create child ContentNodes
parent = CreateObject("roSGNode", "ContentNode")
for i = 0 to 9
    child = parent.createChild("ContentNode")
    child.title = "Item " + i.toStr()
end for
```

Common ContentNode fields: `title`, `description`, `url`, `streamFormat`, `hdPosterUrl`, `sdPosterUrl`, `releaseDate`, `rating`, `categories`, `playStart`, `length`, `bookmarkPosition`

### Task (extends Node)
Runs code on a separate thread. **Critical for network requests and heavy computation.**

```xml
<component name="DataTask" extends="Task">
  <interface>
    <field id="input" type="assocarray" />
    <field id="output" type="assocarray" />
  </interface>
  <script type="text/brightscript" uri="DataTask.brs" />
</component>
```

```brightscript
sub init()
    m.top.functionName = "execute"  ' REQUIRED — name of function to run
end sub

sub execute()
    ' Runs on TASK THREAD — cannot access SceneGraph nodes
    ' Communicate via m.top fields only
    url = m.top.input.url
    request = CreateObject("roUrlTransfer")
    request.SetCertificatesFile("common:/certs/ca-bundle.crt")
    request.InitClientCertificates()
    request.SetUrl(url)
    response = request.GetToString()
    m.top.output = ParseJson(response)
end sub
```

Control a task: `m.task.control = "run"` / `"stop"` / `"done"`

**Key rules:**
- Set `functionName` in `init()` before the task runs
- Cannot create or access SceneGraph nodes from Task thread
- Communicate results via field observation from the render thread
- Use synchronous network calls inside Task (they run on their own thread)

### Timer (extends Node)
Fires callbacks at regular intervals.
```xml
<Timer id="refreshTimer" />
```
```brightscript
sub init()
    m.timer = m.top.findNode("refreshTimer")
    m.timer.duration = 30        ' seconds
    m.timer.repeat = true        ' fire repeatedly
    m.timer.observeField("fire", "onTimerFire")
    m.timer.control = "start"
end sub

sub onTimerFire(event as object)
    ' called every 30 seconds
end sub
```

Fields: `duration` (float, seconds), `repeat` (boolean), `fire` (field to observe), `control` ("start"/"stop"/"none")

### Label (extends Group)
Displays text.
```xml
<Label id="title"
    text="Hello World"
    font="font:LargeBoldSystemFont"
    color="0xFFFFFFFF"
    width="400"
    height="50"
    horizAlign="center"
    vertAlign="center"
    wrap="true"
    maxLines="2"
/>
```

Key fields: `text`, `font`, `color`, `width`, `height`, `horizAlign` ("left"/"center"/"right"), `vertAlign` ("top"/"center"/"bottom"), `wrap` (boolean), `maxLines` (integer), `lineSpacing` (float), `truncateOnDelimiter` (string), `ellipsizeOnBoundary` (boolean), `numLines` (RO — actual line count after wrapping)

### Poster (extends Group)
Displays images.
```xml
<Poster id="background"
    uri="pkg:/images/bg.png"
    width="1920"
    height="1080"
    loadDisplayMode="scaleToFit"
/>
```

Key fields: `uri` (string), `width`/`height` (float), `loadDisplayMode` ("noScale"/"scaleToFit"/"scaleToFill"/"scaleToZoom"/"limitSize"), `loadWidth`/`loadHeight` (int — texture size), `loadStatus` (RO: "none"/"loading"/"ready"/"failed"), `bitmapWidth`/`bitmapHeight` (RO), `blendColor` (color)

### Rectangle (extends Group)
Draws a colored rectangle.
```xml
<Rectangle id="overlay" width="1920" height="1080" color="0x000000AA" />
```

Key fields: `width`/`height` (float), `color` (color), `blendColor` (color)

### LayoutGroup (extends Group)
Arranges children in a line.
```xml
<LayoutGroup id="row"
    layoutDirection="horiz"
    horizAlignment="left"
    vertAlignment="center"
    itemSpacings="[10]"
>
    <Poster ... />
    <Label ... />
</LayoutGroup>
```

Fields: `layoutDirection` ("vert"/"horiz"), `horizAlignment` ("left"/"center"/"right"), `vertAlignment` ("top"/"center"/"bottom"), `itemSpacings` (array of floats — spacing between children), `addItemSpacingAfterChild` (boolean)

### RowList (extends Group)
Displays scrollable rows of items. Used for content grids/carousels.

Key fields: `content` (ContentNode), `rowItemFocused` (vector2d [row, col]), `rowItemSelected` (vector2d), `numRows` (int), `itemSize` (vector2d), `rowHeights` (array), `rowSpacings` (array), `rowFocusAnimationStyle` ("fixedFocusWrap"/"floatingFocus"/"fixedFocus"), `showRowLabel` (array of boolean), `drawFocusFeedback` (boolean), `focusBitmapUri` (uri)

### MarkupGrid (extends Group)
Flexible grid layout with custom item components.

Key fields: `content` (ContentNode), `numColumns` (int), `numRows` (int), `itemSize` (vector2d), `itemSpacing` (vector2d), `drawFocusFeedback` (boolean), `focusBitmapUri` (uri), `itemComponentName` (string — custom component to render each item), `currFocusRow` (RO int), `currFocusColumn` (RO int), `itemFocused` (int), `itemSelected` (int), `animateToItem` (int)

### Scene (fields)
| Field | Type | Description |
|-------|------|-------------|
| `backgroundURI` | string | Background image URI |
| `backgroundColor` | color | Background color |
| `backExitsScene` | boolean | Whether back button exits |
| `dialog` | node | Set to display a dialog overlay |
| `currentDesignResolution` | AA | RO — `width`, `height` of design resolution |

---

## Key BrightScript Components

### roUrlTransfer
HTTP client for network requests. Use inside Task nodes.
```brightscript
request = CreateObject("roUrlTransfer")
request.SetCertificatesFile("common:/certs/ca-bundle.crt")
request.InitClientCertificates()
request.SetUrl("https://api.example.com/data")
request.AddHeader("Content-Type", "application/json")

' Synchronous (use in Task only)
response = request.GetToString()
result = request.PostFromString(jsonBody)

' Asynchronous
port = CreateObject("roMessagePort")
request.SetMessagePort(port)
request.AsyncGetToString()
msg = Wait(5000, port)  ' 5 second timeout
if type(msg) = "roUrlEvent"
    code = msg.GetResponseCode()
    body = msg.GetString()
end if
```

### roSGNode
Interface for creating and manipulating SceneGraph nodes from BrightScript.
```brightscript
node = CreateObject("roSGNode", "ContentNode")
node.addField("customField", "string", false)
node.setField("customField", "value")
child = node.createChild("ContentNode")
node.appendChild(existingNode)
node.removeChild(child)
count = node.getChildCount()
child = node.getChild(0)
```

### roRegistry / roRegistrySection
Persistent key-value storage (survives app restarts).
```brightscript
section = CreateObject("roRegistrySection", "UserData")
section.Write("token", "abc123")
section.Flush()  ' MUST call to persist

token = section.Read("token")
section.Delete("token")
section.Flush()
```

### roDeviceInfo
Device information (model, firmware, display, network).
```brightscript
di = CreateObject("roDeviceInfo")
model = di.GetModel()
version = di.GetVersion()
displaySize = di.GetDisplaySize()  ' {w: 1920, h: 1080}
uuid = di.GetRIDA()               ' Roku ID for Advertisers
locale = di.GetCurrentLocale()     ' "en_US"
connType = di.GetConnectionType()  ' "WiFiConnection" or "WiredConnection"
```

### roChannelStore
In-app purchases and subscription management.
```brightscript
store = CreateObject("roChannelStore")
port = CreateObject("roMessagePort")
store.SetMessagePort(port)
store.GetCatalog()  ' async — wait for roChannelStoreEvent
store.GetPurchases()
store.DoOrder()
```

### roMessagePort
Event message queue. Central to the Roku event model.
```brightscript
port = CreateObject("roMessagePort")
object.SetMessagePort(port)
msg = Wait(0, port)        ' block until message
msg = Wait(5000, port)     ' 5 second timeout, returns invalid on timeout
```

### roArray
Dynamic array.
```brightscript
a = CreateObject("roArray", 10, true)  ' initial size 10, auto-resize
a.Push("item")
a.Pop()
a.Peek()
a.Count()
a.Clear()
a.Append(otherArray)
a.Sort()    ' sort in place
a.Reverse()
item = a[0]
```

### roAssociativeArray
Key-value dictionary. The core data structure in BrightScript.
```brightscript
aa = CreateObject("roAssociativeArray")
aa.AddReplace("key", "value")
aa.Lookup("key")         ' returns value or invalid
aa.DoesExist("key")      ' boolean
aa.Delete("key")
aa.Count()
aa.Clear()
aa.Keys()                ' array of keys
aa.Items()               ' array of {key, value}
aa.Append(otherAA)
aa.SetModeCaseSensitive()  ' default is case insensitive
```

---

## Focus Model

SceneGraph uses a hierarchical focus model. Only one node has focus at a time.

```brightscript
' Set focus
m.myNode.setFocus(true)

' Check focus
if m.myNode.hasFocus() then ...

' Observe focus changes
m.top.observeField("focusedChild", "onFocusChange")

sub onFocusChange(event as object)
    focusedNode = event.getData()
end sub
```

**Key rules:**
- A node must be in the scene tree to receive focus
- Setting focus on a node removes focus from the previously focused node
- `focusedChild` field on any node changes when focus moves within its subtree
- Use `ifSGNodeFocus` interface: `setFocus(true/false)`, `hasFocus()`, `isInFocusChain()`
