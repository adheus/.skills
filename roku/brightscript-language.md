# BrightScript Language Reference

Source: https://developer.roku.com/docs/references/brightscript/language/

## Overview

BrightScript is a powerful bytecode-interpreted scripting language optimized for embedded devices. It compiles to bytecode at load time (no separate compile step). Syntax is not C-like — closer to Basic/Lua. It supports dynamic typing or declared types. Uses "interfaces" and "components" for APIs (similar to Java/.NET).

BrightScript is the "glue" connecting underlying components (network, media, UI) into applications.

---

## Types

| Type | Description |
|------|-------------|
| `Boolean` | `true` or `false` |
| `Integer` | 32-bit signed. BrightScript prefers integers (embedded CPUs may lack FPU) |
| `LongInteger` | 64-bit signed (Roku OS 7.0+) |
| `Float` | 32-bit IEEE floating point |
| `Double` | 64-bit IEEE floating point |
| `String` | Unicode character sequence. Constants are intrinsic; after expressions, becomes `roString` (reference type) |
| `Object` | Reference to a BrightScript component (roArray, roAssociativeArray, etc.) |
| `Function` | Functions/Subs are first-class. Can be stored in variables, passed as arguments |
| `Interface` | An interface in a BrightScript component |
| `Invalid` | Only value: `invalid`. Returned when reading unset array elements, etc. |
| `Dynamic` | Default — type determined at runtime by assigned value |

### Type Declaration Characters
| Char | Type | Example |
|------|------|---------|
| `$` | String | `A$` |
| `%` | Integer | `SUM%` |
| `!` | Float | `value!` |
| `#` | Double | `distance#` |
| `&` | LongInteger | `ID&` (Roku OS 7.0+) |

### Type Conversion
- Operations return the most precise operand type
- Division is NEVER integer — both integers → float result. Use `\` for integer division
- Comparisons convert to most precise type before comparing
- Integer conversion rounds down (like `INT()`)

---

## Literals

```brightscript
' Boolean
x = true
y = false

' Invalid
z = invalid

' String (double-quote to embed quotes)
s = "hello"
s2 = """quoted""" ' contains: "quoted"

' Integer
n = 255
h = &HFF        ' hex

' Float
f = 2.01
f2 = 1.23456E+30

' Double
d = 1.23456789D-12

' LongInteger
l = 9876543210&
lh = &hFEDCBA9876543210&

' Array
a = []
a = [1, 2, 3]
a = [x+5, true, ["nested"]]

' Associative Array
aa = {}
aa = {key1: "value", key2: 55}
aa = {"Key With Spaces": 1001}  ' string keys for mixed case/spaces
```

---

## Operators (by precedence, highest first)

| Operator | Description |
|----------|-------------|
| `()` | Function call / grouping |
| `.` | Dot operator (member access, AA lookup — always case insensitive) |
| `[]` | Array/AA access (evaluates at runtime, case sensitive for AA) |
| `?.` `?@` `?[` `?(` | Optional chaining (Roku OS 11.0+) — returns `invalid` if left side is invalid |
| `^` | Exponentiation (right associative) |
| `-` `+` | Unary negation/positive |
| `*` `/` `MOD` `\` | Multiply, divide, modulo, integer divide |
| `-` `+` | Add, subtract, string concatenation |
| `<<` `>>` | Bitshift |
| `<` `>` `=` `<>` `<=` `>=` | Comparison (string comparisons are CASE SENSITIVE) |
| `NOT` | Logical/bitwise NOT |
| `AND` | Logical/bitwise AND (short-circuit for Boolean) |
| `OR` | Logical/bitwise OR (short-circuit for Boolean) |

### Important: `=` is both assignment and comparison
```brightscript
a = 5              ' assignment
if a = 5 then ...  ' comparison
```
No `==` operator exists. Assignment cannot occur inside expressions.

### Dot operator on Associative Arrays
```brightscript
aa.newkey = "value"      ' always creates lowercase key "newkey"
aa["NewKey"] = "value"   ' preserves case "NewKey"
aa.AddReplace("NewKey", "value")  ' also preserves case
```

### Optional Chaining (Roku OS 11.0+)
```brightscript
x = array?[3]?.foo?.bar?()  ' returns invalid if any part is invalid
' CANNOT use for assignment: array?[12] = x  — NOT supported
' invalid is NOT false: IF aa?.foo THEN — THROWS if aa is invalid
' Correct: IF aa?.foo <> invalid THEN ...
```

### Increment/Decrement (Roku OS 7.1+)
```brightscript
x++ ' increment
x-- ' decrement
x += 2 : x -= 1 : x *= 3 : x /= 2 : x \= 2
x <<= 8 : x >>= 4
```

---

## Program Statements

### Variable Assignment
```brightscript
a$ = "a string"
b1 = 1.23
x = x - z1
```

### DIM (array creation shortcut)
```brightscript
Dim array[5]         ' same as CreateObject("roArray", 6, true)
Dim c[5, 4, 6]      ' multi-dimensional
item = c[1, 2, 3]   ' same as c[1][2][3]
```

### IF / ELSEIF / ELSE / END IF
```brightscript
' Single line
if x > 127 then print "out of range"

' Block form
if type(msg) = "roVideoPlayerEvent" then
    if msg.isFullResult()
        return 9
    end if
else if type(msg) = "roUniversalControlEvent" then
    HandleButton(msg.GetInt())
elseif msg = invalid then
    return 6
end if
```
`THEN` is optional in block form.

### FOR / END FOR
```brightscript
For i = 10 To 1 Step -1
    print i
End For

' EXIT FOR to break early
' CONTINUE FOR to skip to next iteration (Roku OS 7.1+)
```

### FOR EACH / END FOR
```brightscript
aa = {joe: 10, fred: 11, sue: 9}
For Each n In aa
    Print n; aa[n]
End For
' Works on: roList, roArray, roAssociativeArray, roMessagePort
' Safe to delete entries during enumeration
```

### WHILE / END WHILE
```brightscript
while condition
    ' ...
    if done then exit while
    if skip then continue while
end while
```

### TRY / CATCH / END TRY (Roku OS 9.4+)
```brightscript
TRY
    result = 1 / 0
CATCH e
    print "Error: "; e.message
    print "Line: "; e.backtrace[0].line_number
END TRY
```
Exception object fields: `number` (int), `message` (string), `backtrace` (array of {filename, line_number, function}), `rethrown` (bool).

### THROW (Roku OS 9.4+)
```brightscript
THROW "Cannot calculate negative factorial"
' Or throw an exception object:
THROW {number: &h28, message: "custom error"}
```

### FUNCTION / SUB / END FUNCTION
```brightscript
Function add(a as Integer, b as Integer) As Integer
    Return a + b
End Function

' Sub = Function with Void return
Sub doSomething()
    ' ...
End Sub

' Default parameters
Function add2(a as Integer, b = 5 as Integer) As Integer
    Return a + b
End Function

' Anonymous functions
myfunc = Function(a, b)
    Return a + b
End Function

' Parameter types: Integer, Float, Double, Boolean, String, Object, Dynamic, Function
' Return types: above + Void
```

### The `m` Pointer ("this")
When a function is called from an Associative Array, `m` refers to that AA:
```brightscript
obj = {
    value: 10,
    getValue: function()
        return m.value  ' m points to obj
    end function
}
print obj.getValue()  ' prints 10
```
If not called from an AA, `m` is the module-global AA (same as `GetGlobalAA()`).

### PRINT
```brightscript
print "value: "; x          ' semicolon: no space
print "col1", "col2"        ' comma: tab to next 16-char zone
? "shortcut"                ' ? is alias for print
```

### Other
```brightscript
END          ' terminate execution
STOP         ' invoke debugger
GOTO label   ' branch to label
RETURN expr  ' return from function
REM comment  ' or ' comment
```

---

## Scope Rules

- No traditional global variables (except `global` object and `GetGlobalAA()`)
- Named functions are global scope; anonymous functions are local scope
- Local variables have function scope
- Block statements (FOR, WHILE, IF) do NOT create separate scope
- `m` in a non-method function = GetGlobalAA() (module-global AA)

---

## Component Architecture

### Intrinsic vs Object Types
- **Intrinsic** (Integer, Float, Double, String, Boolean, Invalid, Function): passed by value (copied)
- **Object** (roArray, roAssociativeArray, BrightScript Components): passed by reference (reference counted)

```brightscript
a = [1, 2, 3]
b = a         ' b is a reference to the SAME array
a[0] = 99     ' b[0] is also 99 now
```

### Reference Counting & Garbage Collection
Objects are freed when reference count reaches zero (immediate, deterministic). Circular references are cleaned by mark-and-sweep GC (runs after script ends or via `RunGarbageCollector()`).

### Threading Model
BrightScript runs single-threaded per script. Components may run async operations (roUrlTransfer, roVideoPlayer) that post events to a `roMessagePort`. In SceneGraph, Task nodes run on separate threads — communicate via field observation only.

### Events
```brightscript
port = CreateObject("roMessagePort")
obj.SetMessagePort(port)
while true
    msg = Wait(0, port)  ' 0 = wait forever
    if type(msg) = "roSomeEvent" then
        ' handle event
    end if
end while
```

---

## Runtime Functions

| Function | Description |
|----------|-------------|
| `CreateObject(name, [params])` | Create a BrightScript component. Returns invalid on failure |
| `Type(variable, [version])` | Returns type name as string |
| `GetGlobalAA()` | Returns the module-global associative array |
| `Box(x)` | Returns object version of intrinsic type |

---

## Global Utility Functions

| Function | Description |
|----------|-------------|
| `Sleep(ms)` | Pause execution (1000 = 1 second) |
| `Wait(timeout, port)` | Wait for message on port. 0 = forever. Returns event or invalid |
| `GetInterface(obj, ifname)` | Get a specific interface from an object |
| `FindMemberFunction(obj, name)` | Find which interface provides a function |
| `UpTime(0)` | System uptime in seconds (float) |
| `ListDir(path)` | List directory contents |
| `ReadAsciiFile(path)` | Read file as string (UTF-8/UTF-16) |
| `WriteAsciiFile(path, text)` | Write string to file (UTF-8) |
| `CopyFile(src, dst)` | Copy a file |
| `MoveFile(src, dst)` | Rename/move a file |
| `DeleteFile(file)` | Delete a file |
| `CreateDirectory(dir)` | Create a directory |
| `MatchFiles(path, pattern)` | Wildcard file search |
| `ParseJson(str, [flags])` | Parse JSON → BrightScript objects. Use "i" flag for case-insensitive AAs. Use "d" flag for double-precision floats (Roku OS 14.6+) |
| `FormatJson(obj, [flags])` | BrightScript objects → JSON string |
| `Tr(source)` | Translate string via locale XLIFF file |
| `StrToI(str)` | String to integer |
| `RunGarbageCollector()` | Force garbage collection |
| `Substitute(str, arg0, ...)` | Replace {0}, {1}, etc. in string |

---

## Global String Functions

| Function | Description |
|----------|-------------|
| `UCase(s)` / `LCase(s)` | Upper/lower case |
| `Len(s)` | String length |
| `Left(s, n)` / `Right(s, n)` | First/last n characters |
| `Mid(s, p, [n])` | Substring from position p (1-based) |
| `Instr(start, text, sub)` | Find substring (1-based, 0 if not found) |
| `Asc(s)` / `Chr(n)` | Character ↔ Unicode value |
| `Str(f)` / `StrI(i)` | Number → string (leading blank for positive) |
| `Val(s)` / `Val(s, radix)` | String → number |
| `String(n, s)` | Repeat string n times |
| `StringI(n, ch)` | Repeat character (by Unicode value) n times |

**Note:** Global string functions use 1-based indexing. The `ifStringOps` methods (`.Instr()`, `.Mid()`) use 0-based indexing.

---

## Global Math Functions

All trig functions use radians, not degrees. Multiply by 0.01745329 to convert degrees → radians.

| Function | Description |
|----------|-------------|
| `Abs(x)` | Absolute value |
| `Int(x)` | Largest integer ≤ x |
| `Fix(x)` | Truncate toward zero |
| `Cint(x)` | Round to integer (round up from midpoints) |
| `Csng(x)` / `Cdbl(x)` | Convert to float/double |
| `Sgn(x)` | Sign: -1, 0, or 1 |
| `Exp(x)` / `Log(x)` | Natural exp / natural log |
| `Sqr(x)` | Square root |
| `Sin(x)` / `Cos(x)` / `Tan(x)` / `Atn(x)` | Trig (radians) |
| `Rnd(0)` | Random float 0–1 |
| `Rnd(n)` | Random integer 1–n |

---

## XML Support

BrightScript has built-in XML support via `roXMLElement` and `roXMLList`.

```brightscript
rsp = CreateObject("roXMLElement")
rsp.Parse(ReadAsciiFile("tmp:/data.xml"))

' Dot operator: get child elements (case insensitive)
photos = rsp.photos.photo  ' roXMLList

' @ operator: get attribute (case insensitive)
perPage = rsp.photos@perpage  ' string "100"

' GetText(): get element text content
title = booklist.book.GetText()

' Namespaced elements
elements = xml.GetNamedElements("media:thumbnail")
attrs = element.GetAttributes()["xmlns:media"]
```

---

## Reserved Words

STOP, RUN, END, FOR, TO, STEP, EXIT, NEXT, WHILE, ENDWHILE, FUNCTION, ENDFUNCTION, SUB, ENDSUB, AS, RETURN, PRINT, GOTO, DIM, REM, TRUE, FALSE, AND, OR, NOT, IF, THEN, ELSE, ELSEIF, ENDIF, EACH, IN, TYPE, LIBRARY, MOD, INVALID, BOOLEAN, INTEGER, LONGINTEGER, FLOAT, DOUBLE, STRING, OBJECT, INTERFACE, DYNAMIC, VOID, LET, LINE_NUM, THROW, TRY, CATCH, ENDTRY, CONTINUE

---

## Error Handling (Roku OS 9.4+)

### Common Error Codes
| Code | Name | Description |
|------|------|-------------|
| `&hFF` | ERR_OKAY | No error |
| `&hFC` | ERR_NORMAL_END | Normal termination |
| `&hE2` | ERR_VALUE_RETURN | Value returned |
| `&hE9` | ERR_USE_OF_UNINIT_VAR | Uninitialized variable |
| `&h14` | ERR_DIV_ZERO | Division by zero |
| `&h18` | ERR_TM | Type mismatch |
| `&hF4` | ERR_RO2 | Member function not found |
| `&hF1` | ERR_WRONG_NUM_PARAM | Wrong parameter count |
| `&h28` | ERR_USER | User-thrown error |

### Exception Object Structure
```brightscript
' e.number    — error code (integer)
' e.message   — error description (string)
' e.backtrace — array of {filename, line_number, function}
' e.rethrown  — boolean
```
