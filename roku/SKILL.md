---
name: roku
description: Roku BrightScript and SceneGraph development reference. Use when writing, debugging, or reviewing BrightScript/SceneGraph code for Roku apps. Covers language syntax, SceneGraph nodes, Video playback, SGDEX, and community tools.
user-invocable: true
argument-hint: [topic or question]
paths: "**/*.brs,**/*.xml,**/manifest"
---

# Roku / BrightScript Development Skill

You are assisting with Roku app development using BrightScript and SceneGraph. Use the reference files below to provide accurate, idiomatic guidance.

## Reference Files

Use these references when answering questions or writing code:

- [BrightScript Language Reference](brightscript-language.md) — Types, operators, control flow, functions, scope, threading, error handling, global functions (ParseJson, FormatJson, Wait, Sleep, ReadAsciiFile, etc.)
- [SceneGraph Reference](scenegraph-reference.md) — XML elements, component functions (init, onKeyEvent), key nodes (Task, Timer, ContentNode, Label, Poster, RowList, MarkupGrid, LayoutGroup, Scene)
- [Video Node Reference](video-node.md) — Complete Video node spec: playback fields, DRM, ads (RAF/SSAI), trickplay, captions, buffering, error codes. Critical for OTT streaming apps.
- [SGDEX Reference](sgdex.md) — SceneGraph Developer Extensions: GridView, MediaView, DetailsView, ContentHandler, ComponentController, theming, content management
- [Community Tools](community-tools.md) — BrighterScript (language superset), ropm (package manager), bslint (linter), Rooibos (test framework)

## Key Principles

1. **BrightScript is NOT JavaScript.** It uses `end if` / `end for` / `end function` block terminators. No curly braces. Case insensitive identifiers. `m` is the `this` pointer for AA method calls. Strings compare case-sensitively. `=` is both assignment and comparison (no `==`). Division is never integer — use `\` for integer division.

2. **SceneGraph threading model.** The render thread runs component `init()` and field observers. Task nodes run on separate threads. You CANNOT access SceneGraph nodes from a Task thread — communicate via field observation. Use `observeField` / `observeFieldScoped` for cross-thread communication.

3. **Field observation is the primary event mechanism.** Unlike traditional event loops, SceneGraph uses `observeField("fieldName", "callbackFunctionName")` to react to state changes. The callback receives an `roSGNodeEvent`.

4. **ContentNode is the universal data container.** All content in SceneGraph flows through ContentNode trees. Set metadata as fields on ContentNode children. The Video node, lists, grids all consume ContentNode hierarchies.

5. **Memory matters on embedded devices.** Roku devices have limited memory. Avoid large arrays, prefer lazy loading, clean up references. Use `roTextureManager` for image caching. The garbage collector runs reference counting (immediate) plus mark-and-sweep (manual via `RunGarbageCollector()`).

6. **roUrlTransfer for network calls.** Use `AsyncGetToString()` / `AsyncPostFromString()` for non-blocking requests in Task nodes. Always set a message port and handle `roUrlEvent` responses. Synchronous methods (`GetToString()`) block the thread.

## Common Patterns

### Creating a Task for async work
```xml
<!-- MyTask.xml -->
<component name="MyTask" extends="Task">
  <interface>
    <field id="requestUrl" type="string" />
    <field id="response" type="assocarray" />
  </interface>
  <script type="text/brightscript" uri="MyTask.brs" />
</component>
```
```brightscript
' MyTask.brs
sub init()
  m.top.functionName = "execute"
end sub

sub execute()
  request = CreateObject("roUrlTransfer")
  request.SetCertificatesFile("common:/certs/ca-bundle.crt")
  request.InitClientCertificates()
  request.SetUrl(m.top.requestUrl)
  result = request.GetToString()
  m.top.response = ParseJson(result)
end sub
```

### Observing fields
```brightscript
sub init()
  m.myTask = m.top.findNode("myTask")
  m.myTask.observeField("response", "onResponse")
  m.myTask.requestUrl = "https://api.example.com/data"
  m.myTask.control = "run"
end sub

sub onResponse(event as object)
  response = event.getData()
  ' process response
end sub
```

### Video playback
```brightscript
sub playVideo(url as string, title as string)
  videoNode = m.top.findNode("videoPlayer")
  content = CreateObject("roSGNode", "ContentNode")
  content.url = url
  content.title = title
  content.streamFormat = "hls"
  videoNode.content = content
  videoNode.control = "play"
end sub
```

## If the topic is: $ARGUMENTS
Focus your answer on the requested topic using the reference files above.
