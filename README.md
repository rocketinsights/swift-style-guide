## Table of Contents

* [Overview](#overview)
* [Naming](#naming)
  * [Idioms](#idioms)
  * [Enumerations](#enumerations) 
* [Spacing](#spacing)
* [Comments](#comments)
* [Classes and Structures](#classes-and-structures)
  * [Use of Self](#use-of-self)
  * [Computed Properties](#computed-properties)
  * [Method Declarations](#function-declarations)
* [Closure Expressions](#closure-expressions)
* [Types](#types)
  * [Constants](#constants)
  * [Optionals](#optionals)
  * [Struct Initializers](#struct-initializers)
  * [Type Inference](#type-inference)
  * [Syntactic Sugar](#syntactic-sugar)
* [Control Flow](#control-flow)
  * [Guard](#guard)
* [Credits](#credits)


## Overview
This document captures the Swift style at Virgin Pulse. Note, we also have several higher level idioms and gotchas to look out for [here](https://bitbucket.org/vpulse/ios-genesis-app/wiki/Code%20Review%20Process%20+%20Things%20to%20Look%20For)

## Naming

Use descriptive names with camel case for classes, methods, variables, etc. Class names should be capitalized, while method names and variables should start with a lower case letter.

**Preferred:**

```swift
private let maximumWidgetCount = 100

class WidgetContainer {
  var widgetButton: UIButton
  let widgetHeightPercentage = 0.85
}
```

**Not Preferred:**

```swift
let MAX_WIDGET_COUNT = 100

class app_widgetContainer {
  var wBut: UIButton
  let wHeightPct = 0.85
}
```

For methods, follow the standard Apple convention of referring to the first parameter in the method name:

```swift
class Guideline {
  func combineWithString(incoming: String, options: Dictionary?) { ... }
  func upvoteBy(amount: Int) { ... }
}
```
### Idioms
* Use **Id** and not **ID**.

### Enumerations

Use UpperCamelCase for enumeration values:

```swift
enum Shape {
  case Rectangle
  case Square
  case Triangle
  case Circle
}
```

## Spacing

* Indent using 4 spaces rather than tabs to conserve space and help prevent line wrapping. 
* Method braces and other braces (`if`/`switch`/`while` etc.) always open on the same line as the statement but close on a new line.
* `else` braces always open on the next line
* Tip: You can re-indent by selecting some code (or ⌘A to select all) and then Control-I (or Editor\Structure\Re-Indent in the menu). Some of the Xcode template code will have 4-space tabs hard coded, so this is a good way to fix that.

**Preferred:**
```swift
if user.isHappy {
  // Do something
} 
else {
  // Do something else
}
```

**Not Preferred:**
```swift
if user.isHappy
{
    // Do something
} else {
    // Do something else
}
```

* There should be exactly one blank line between methods to aid in visual clarity and organization. Whitespace within methods should separate functionality, but having too many sections in a method often means you should refactor into several methods.

## Comments

When they are needed, use comments to explain **why** a particular piece of code does something. Comments must be kept up-to-date or deleted.

Avoid block comments inline with code, as the code should be as self-documenting as possible. *Exception: This does not apply to those comments used to generate documentation.*

## Classes and Structures

### Use of Self

For conciseness, avoid using `self` since Swift does not require it to access an object's properties or invoke its methods.

Use `self` when required to differentiate between property names and arguments in initializers, and when referencing properties in closure expressions (as required by the compiler):

```swift
class BoardLocation {
  let row: Int, column: Int

  init(row: Int, column: Int) {
    self.row = row
    self.column = column
    
    let closure = {
      println(self.row)
    }
  }
}
```

### Computed Properties

For conciseness, if a computed property is read-only, omit the get clause. The get clause is required only when a set clause is provided.

**Preferred:**
```swift
var diameter: Double {
  return radius * 2
}
```

**Not Preferred:**
```swift
var diameter: Double {
  get {
    return radius * 2
  }
}
```

### Method Declarations

Keep short func declarations on one line including the opening brace:

```swift
func reticulateSplines(spline: [Double]) -> Bool {
  // reticulate code goes here
}
```

For functions with long signatures, add line breaks at appropriate points and add an extra indent on subsequent lines:

```swift
func reticulateSplines(spline: [Double], adjustmentFactor: Double,
    translateConstant: Int, comment: String) -> Bool {
  // reticulate code goes here
}
```


## Closure Expressions

Use trailing closure syntax only if there's a single closure expression parameter at the end of the argument list. Give the closure parameters descriptive names.

**Preferred:**
```swift
UIView.animateWithDuration(1.0) {
  self.myView.alpha = 0
}

UIView.animateWithDuration(1.0,
  animations: {
    self.myView.alpha = 0
  },
  completion: { finished in
    self.myView.removeFromSuperview()
  }
)
```

**Not Preferred:**
```swift
UIView.animateWithDuration(1.0, animations: {
  self.myView.alpha = 0
})

UIView.animateWithDuration(1.0,
  animations: {
    self.myView.alpha = 0
  }) { f in
    self.myView.removeFromSuperview()
}
```

For single-expression closures where the context is clear, use implicit returns:

```swift
attendeeList.sort { a, b in
  a > b
}
```


## Types

Always use Swift's native types when available. Swift offers bridging to Objective-C so you can still use the full set of methods as needed.

**Preferred:**
```swift
let width = 120.0                                    // Double
let widthString = (width as NSNumber).stringValue    // String
```

**Not Preferred:**
```swift
let width: NSNumber = 120.0                          // NSNumber
let widthString: NSString = width.stringValue        // NSString
```

In Sprite Kit code, use `CGFloat` if it makes the code more succinct by avoiding too many conversions.

### Constants

Constants are defined using the `let` keyword, and variables with the `var` keyword. Always use `let` instead of `var` if the value of the variable will not change.

**Tip:** A good technique is to define everything using `let` and only change it to `var` if the compiler complains!


### Optionals

Declare variables and function return types as optional with `?` where a nil value is acceptable.

Use implicitly unwrapped types declared with `!` only for instance variables that you know will be initialized later before use, such as subviews that will be set up in `viewDidLoad`.

When accessing an optional value, use optional chaining if the value is only accessed once or if there are many optionals in the chain:

```swift
self.textContainer?.textLabel?.setNeedsDisplay()
```

Use optional binding when it's more convenient to unwrap once and perform multiple operations:

```swift
if let textContainer = self.textContainer {
  // do many things with textContainer
}
```

For optional binding, shadow the original name when appropriate rather than using names like `unwrappedView` or `actualLabel`.

**Preferred:**
```swift
var subview: UIView?
var volume: Double?

// later on...
if let subview = subview, volume = volume {
  // do something with unwrapped subview and volume
}
```

**Not Preferred:**
```swift
var optionalSubview: UIView?
var volume: Double?

if let unwrappedSubview = optionalSubview {
  if let realVolume = volume {
    // do something with unwrappedSubview and realVolume
  }
}
```

### Struct Initializers

Use the native Swift struct initializers rather than the legacy CGGeometry constructors.

**Preferred:**
```swift
let bounds = CGRect(x: 40, y: 20, width: 120, height: 80)
let centerPoint = CGPoint(x: 96, y: 42)
```

**Not Preferred:**
```swift
let bounds = CGRectMake(40, 20, 120, 80)
let centerPoint = CGPointMake(96, 42)
```

Prefer the struct-scope constants `CGRect.infiniteRect`, `CGRect.nullRect`, etc. over global constants `CGRectInfinite`, `CGRectNull`, etc. For existing variables, you can use the shorter `.zeroRect`.

### Type Inference

Prefer compact code and let the compiler infer the type for a constant or variable, unless you need a specific type other than the default such as `CGFloat` or `Int16`.

**Preferred:**
```swift
let message = "Click the button"
let currentBounds = computeViewBounds()
var names = [String]()
let maximumWidth: CGFloat = 106.5
```

**Not Preferred:**
```swift
let message: String = "Click the button"
let currentBounds: CGRect = computeViewBounds()
var names: [String] = []
```

**NOTE**: Following this guideline means picking descriptive names is even more important than before.


### Syntactic Sugar

Prefer the shortcut versions of type declarations over the full generics syntax.

**Preferred:**
```swift
var deviceModels: [String]
var employees: [Int: String]
var faxNumber: Int?
```

**Not Preferred:**
```swift
var deviceModels: Array<String>
var employees: Dictionary<Int, String>
var faxNumber: Optional<Int>
```


## Control Flow

Prefer the `for-in` style of `for` loop over the `for-condition-increment` style.

**Preferred:**
```swift
for _ in 0..<3 {
  println("Group of owls is a parliament")
}

for (index, owl) in enumerate(owls) {
  println("\(owl) is at position #\(index)")
}
```

**Not Preferred:**
```swift
for var i = 0; i < 3; i++ {
  println("Group of owls is a parliament")
}

for var i = 0; i < owls.count; i++ {
  let owl = owls[i]
  println("\(owl) is at position #\(i)")
}
```

### Guard
The guard statement can and should be used to avoid the pyramid of doom. Exit out of methods early if input does not meet requirements.

**Preferred**
```swift
guard let raftOfOtters = raftOfOtters else { return }
// Use unwrapped `raftOfOtters` value in here
```

** Not Preferred**
```swift
if let raftOfOtters = raftOfOtters {
    // Use unwrapped `raftOfOtters` value in here
} else {
    return
}
```

## Credits

This style guide is heavily based off of Ray Wenderlich's
