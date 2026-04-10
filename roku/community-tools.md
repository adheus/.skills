# Roku Community Tools Reference

Source: https://github.com/rokucommunity

The RokuCommunity organization maintains the primary open-source tooling ecosystem for Roku development.

---

## BrighterScript

**Repo:** https://github.com/rokucommunity/brighterscript
**What:** A superset of BrightScript that transpiles to standard BrightScript. Write modern code, deploy to any Roku device.

### Key Language Extensions

**Classes** with inheritance and access modifiers:
```brighterscript
class Animal
    public name as string
    protected sound as string

    sub new(name as string)
        m.name = name
    end sub

    function speak() as string
        return m.name + " says " + m.sound
    end function
end class

class Dog extends Animal
    sub new(name as string)
        super(name)
        m.sound = "woof"
    end sub
end class
```

**Namespaces** — automatic prefixing, prevents naming collisions:
```brighterscript
namespace MyApp.Utils
    function formatDate(d as object) as string
        ' ...
    end function
end namespace

' Usage:
result = MyApp.Utils.formatDate(dateObj)
' Transpiles to: result = MyApp_Utils_formatDate(dateObj)
```

**Import statements** — declare dependencies in code, not XML:
```brighterscript
import "pkg:/source/utils.bs"
```

**Modern operators:**
- Ternary: `result = condition ? valueIfTrue : valueIfFalse`
- Null-coalescing: `value = possiblyInvalid ?? defaultValue`
- Template strings: `text = `Hello ${name}, you have ${count} items``

**Enum support:**
```brighterscript
enum Direction
    Up = "up"
    Down = "down"
    Left = "left"
    Right = "right"
end enum
```

### Compiler Features
- Static analysis — catches errors without running on a device
- Watch mode — real-time validation during development
- Plugin API — extend compiler with custom diagnostics/transforms
- Language Server Protocol (LSP) — powers VS Code extension
- Deploy to device — can zip and deploy after compilation

### Configuration (bsconfig.json)
```json
{
    "rootDir": "src",
    "stagingDir": "build",
    "files": ["source/**/*", "components/**/*", "manifest"],
    "autoImportComponentScript": true,
    "sourceMap": true,
    "plugins": ["@rokucommunity/bslint"]
}
```

### File Extensions
- `.bs` — BrighterScript source files
- `.brs` — Standard BrightScript (also valid BrighterScript)
- `.d.bs` — BrighterScript type definition files

---

## ropm (Package Manager)

**Repo:** https://github.com/rokucommunity/ropm
**What:** npm-based package manager for Roku. Solves the global scope collision problem.

### The Problem
All BrightScript functions in `pkg:/source/` share a single global scope. Component names must be unique across the entire project. Multiple packages with the same function/component names would collide.

### The Solution
ropm automatically rewrites function names and component references with module-specific prefixes during installation.

### Usage
```bash
npm init                    # Initialize package.json
npx ropm install logger     # Install a package
npx ropm clean              # Remove all roku_modules
npx ropm copy               # Copy without fetching from registry
npx ropm uninstall logger   # Remove a package
```

### How Packages Are Installed
```
source/roku_modules/logger/       # BrightScript source
components/roku_modules/logger/   # SceneGraph components
```

### Prefixing
- `myFunction()` → `logger_myFunction()`
- `_privateFunc()` → `_logger_privateFunc()` (underscore preserved)
- Reserved functions NEVER prefixed: `RunUserInterface`, `Main`, `RunScreenSaver`, `Init`, `OnKeyEvent`

### Configuration (package.json)
```json
{
    "ropm": {
        "rootDir": "src",
        "noprefix": ["trusted-module"]
    }
}
```

### Best Practices
- Add `roku_modules/` to `.gitignore`
- Never manually modify installed module code
- Run `ropm install` after cloning repos
- Follow semver when publishing packages

---

## bslint (Linter)

**Repo:** https://github.com/rokucommunity/bslint
**What:** BrighterScript plugin that adds linting rules for code quality.

### Setup
```json
// bsconfig.json
{
    "plugins": ["@rokucommunity/bslint"]
}
```

### Configuration (bslint.json)
```json
{
    "rules": {
        "assign-all-paths": "error",
        "unsafe-path-loop": "error",
        "unsafe-iterators": "error",
        "case-sensitivity": "warn",
        "no-print": "warn",
        "aa-comma-style": "warn",
        "no-todo": "off",
        "color-format": "warn"
    },
    "globals": ["globalFunc1"],
    "ignores": ["**/vendor/**"]
}
```

### Rule Categories

**Code Style:** function keyword preferences, conditional syntax, parentheses, AA comma style, color format validation

**Strictness:** type annotation enforcement for function args and return values

**Code Flow:** variable assignment completeness, loop safety, case sensitivity issues, unused variables, unreachable code, return value consistency

### CLI Usage
```bash
npx bslint              # Run linter
npx bslint --fix        # Auto-fix style issues
npx bslint --checkUsage # Find unused components/scripts
```

---

## Rooibos (Test Framework)

**Repo:** https://github.com/rokucommunity/rooibos
**What:** Mocha/JUnit-inspired test framework for Roku SceneGraph apps. Used in production with 10,000s of tests across many companies.

### Key Features
- Describe/it test syntax familiar to JS developers
- Assertions library
- Mocking and stubbing
- Parameterized tests
- Node-specific testing (SceneGraph component tests)
- Setup/teardown lifecycle hooks
- Test isolation

### Test Syntax
```brighterscript
@suite("Calculator Tests")
class CalculatorTests extends rooibos.BaseTestSuite

    @describe("add")
    @it("adds two positive numbers")
    function _()
        m.assertEqual(add(2, 3), 5)
    end function

    @it("handles negative numbers")
    function _()
        m.assertEqual(add(-1, 1), 0)
    end function

    @describe("divide")
    @it("divides correctly")
    @params(10, 2, 5)
    @params(9, 3, 3)
    function _(a, b, expected)
        m.assertEqual(divide(a, b), expected)
    end function
end class
```

### Assertions
```brighterscript
m.assertEqual(actual, expected)
m.assertNotEqual(actual, expected)
m.assertTrue(value)
m.assertFalse(value)
m.assertInvalid(value)
m.assertNotInvalid(value)
m.assertArrayContains(array, value)
m.assertArrayCount(array, count)
m.assertEmpty(value)
m.assertNotEmpty(value)
```

### Mocking
```brighterscript
@it("mocks a function call")
function _()
    m.expectCalled(m.myObj.getData(), {"result": "mocked"})
    ' When m.myObj.getData() is called, it returns the mocked value
end function
```

### Setup
- Requires BrighterScript compiler
- Configuration via `bsconfig.json` and test runner config
- Documentation: https://rokucommunity.github.io/rooibos
- Sample project: https://github.com/rokucommunity/rooibos-roku-sample

---

## Other Community Tools

| Tool | Description |
|------|-------------|
| **vscode-brightscript-language** | VS Code extension — syntax highlighting, IntelliSense, debugging, deploy to device |
| **roku-deploy** | npm module for zipping and deploying to Roku devices |
| **roku-debug** | Debug protocol implementation |
| **brighterscript-formatter** | Code formatter for consistent style |
| **promises** | Promise library for async patterns on Roku |
| **bsbench** | Benchmarking suite for BrightScript runtime |

### Recommended Development Setup
1. VS Code + BrightScript Language extension
2. BrighterScript compiler (transpile + static analysis)
3. bslint plugin (code quality)
4. Rooibos (testing)
5. ropm (package management)
6. roku-deploy (automated deployment to test devices)
