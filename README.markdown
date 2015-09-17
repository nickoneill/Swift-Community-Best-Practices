# Swift-Best-Practices

Best practices for software development with Swift.

## Preface

This document grew from an set of notes I produced while working on [SwiftGraphics][SwiftGraphics]. Most of the recommendations in this guide are definitely considered opinions and arguments could be made for other approaches. That's fine. When other approaches make sense they should be presented in addition.

These best practices do not dictate or recommend whether Swift should be used in an procedural, object-oriented or functional manner. Instead a pragmatic approach is taken. Individual recommendations might be focused on object-oriented or functional solutions as needed.

The scope of this document is mostly aimed at the Swift language and Swift standard library. That said specific recommendations on how to use Swift with Mac OS, iOS, WatchOS and TVOS might be provided if a unique Swift angle or insight can be provided. Hints & tips style recommendations on how to use Swift effectively with Xcode and LLDB might also be provided.

This is very much a work in progress. Contributions are very much appreciated in the form of pull requests or filing of issues.

Discussion can be found on the [Swift-Lang slack][slack] (in the #bestpractices channel)

### Note to Contributors

Please make sure all examples are runnable (which may not be the case for existing examples). This markdown will be converted to a Mac OS X playground.

### Golden Rules

* Apple is generally right. Defer to Apple's preferred or demonstrated way of doing things. You should follow the style of Apple's code as defined within their “[The Swift Programming Language][Swift_Programming_Language]” book wherever possible. However Apple is a large corporation and be prepared to see discrepancies in their example code.
* Never write code merely to attempt to reduce the number of keystrokes you need to type. Rely on autocompletion, autosuggestion, copy and paste, etc instead. Verbosity is often helpful to other maintainers of your code. That said, being overly verbose can bypass one of Swift's key benefits: type inference.

## Best Practices

### Naming

As per the “[Swift Programming Language][Swift_Programming_Language]” type names should be [upper camel case][Studly_caps] (example: “`VehicleController`”).

[Studly_caps]: https://en.wikipedia.org/wiki/Studly_caps

Variables and constants should be lower camel case (example “`vehicleName`”).

You should use Swift modules to namespace your code and not use Objective-C style class prefixes for Swift code (unless of course interfacing with Objective-C).

Do not use any form of [Hungarian notation][Hungarian_notation] (e.g. k for constants, m for methods), instead use short concise names and use Xcode's type Quick Help (⌥ + click) to discover a variable's type. Similarly do not use [`SNAKE_CASE`][Snake_case].

[Hungarian_notation]: https://en.wikipedia.org/wiki/Hungarian_notation
[Snake_case]: https://en.wikipedia.org/wiki/Snake_case

The only exception to this general rule are enum values, which should be uppercase (this follows Apple's "[Swift Programming Language][Swift_Programming_Language]" style):

```swift
enum Planet {
    case Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}
```

Needless contractions and abbreviations should be avoided where at all possible, you can actually type out the characters "ViewController" without any harm and rely on Xcode's autocompletion to save you typing in the future. Extremely common abbreviations such as URL are fine. Abbreviations should be represented all uppercase ("URL") or all lowercase "url" as appropriate. Use the same rule for types and variables; if url was a type it would be uppercase, if url was a variable it would be lower case.

### Comments

Comments should _not_ be used to disable code. Commented out code is dead code and pollutes your source. If you want to remove code but keep it around in case it's useful in the future you should be relying on git and/or your bug tracker.

(TODO: Add section about doc comments with link to nshipster)

### Type inference

Where possible, use Swift’s type inference to help reduce redundant type information. For example, prefer:

```swift
var currentLocation = Location()
```

to:

```swift
var currentLocation: Location = Location()
```

### Self Inference

Let the compiler infer self in all cases where it is able to. Areas where self should be explicitly used includes setting parameters in init, and non-escaping closures. For example:

```swift
struct Example {
    let name: String

    init(name: String) {
        self.name = name
    }
}
```

### Capture List Inference

Specifying parameter types inside a capture list can lead to a rather verbose code. Only specify types if needed.

```swift
let people = [
    ("Mary", 42),
    ("Susan", 27),
    ("Charlie", 18),
]

let strings = people.map() {
    (name: String, age: Int) -> String in
    return "\(name) is \(age) years old"
}
```

If at all possible remove the types if the compiler can infer them:

```swift
let strings = people.map() {
    (name, age) in
    return "\(name) is \(age) years old"
}
```

Using the numbered parameter names ("`$0`") further reduces verbosity, often eliminating capture lists completely. Only use the numbered form when the parameter names add no further information to the closure (e.g. very simple maps and filters).

Apple can and will change the parameter types of closures provided by their Swift "conversion" of Objective-C frameworks. For example, optionals are removed or changed to auto-unwrapping etc. Intentionally under-specifying your optionals and relying on Swift to infer the types, reduces the risk of the code breaking under these circumstances.

You should almost always refrain from specifying the return type. For example this capture list is completely redundant:

```swift
dispatch_async(queue) {
    () -> Void in
    print("Fired.")
}
```

(see also: http://www.russbishop.net/swift-capture-lists)

### Constants

Constants used within type definitions should be declared static within a type. For example:

```swift
struct PhysicsModel {
    static var speedOfLightInAVacuum = 299_792_458
}

class Spaceship {
    static let topSpeed = PhysicsModel.speedOfLightInAVacuum
    var speed: Double

    func fullSpeedAhead() {
        speed = Spaceship.topSpeed
    }
}
```

Making the constants static allow them to be referred to without needing instances of the type.

Constants at global level should generally be avoided except for singletons.

### Computed Properties

Use the short version of computed properties if you only need to implement a getter. For example, prefer this:

```swift
class Example {
    var age: UInt32 {
        return arc4random()
    }
}
```

to this:

```swift
class Example {
    var age: UInt32 {
        get {
            return arc4random()
        }
    }
}
```

If you add a `set` to the property then you will need to explicitly provide a `get`.

```swift
class Person {
    var age: Int {
        get {
            return Int(arc4random())
        }
        set {
            print("That's not your age.")
        }
    }
}
```

### Converting Instances

When creating code to convert instances from one type to another `init()` methods:

```swift
extension NSColor {
    convenience init(_ mood: Mood) {
        super.init(color: NSColor.blueColor)
    }
}
```
Init methods now seem to be the preferred manner to convert instances of one type to another in the Swift Standard Library.

"to" methods are another reasonable technique (although you should follow Apple's lead and use init methods):

```swift
struct Mood {
    func toColor() -> NSColor {
        return NSColor.blueColor()
  }
}
```

While you might be tempted to use a getter, e.g:

```swift
struct Mood {
    var color: NSColor {
        return NSColor.blueColor()
    }
}
```

getters should generally be limited to returning components of the receiving type. For example returning the area of a `Circle` instance is well suited to be a getter, but converting a `Circle` to a `CGPath` is better as a "to" function or an `init()` extension on `CGPath`.

### Singletons

Singletons are simple in Swift:

```swift
class ControversyManager {
    static let sharedInstance = ControversyManager()
}
```

The Swift runtime will make sure that the singleton is created and accessed in a thread-safe manner.

Singletons should generally just be accessed via "`sharedInstance`" static property unless you have a compelling reason to name it otherwise. Do not use static functions or global functions to access your singleton.

(Because singletons are so easy in Swift and because consistent naming saves you so much time you will have even more time to complain about how singletons are an anti-pattern and should be avoided at all costs. Your fellow developers will thank you.)

### Extensions for Code Organisation

Extensions should be used to help organise code.

Methods and properties that are peripheral to an instance should be moved to an extension. Note that, currently not all property types can be moved to an extension - do the best you can within this limitation.

You should use extensions to help organise your instance definitions. One good example of this is a view controller that implements table view data source and delegate protocols. Instead of mixing all that table view code into one class, put the data source and delegate methods onto extensions that adopt the relevant protocol.

Inside a single source file feel free to break down a definition into whatever extensions you feel best organise the code in question. Don't worry about methods in the main class or struct definition referring to methods or properties inside extensions. As long as it is all contained within one Swift file it is all good.

Conversely, the main instance definition should not refer to elements defined in extensions outside of the main Swift file..

### Chained Setters

Do not use chained methods as a more "convenient" replacement for simple property setters:

Prefer:

```swift
instance.foo = 42
instance.bar = "xyzzy"
```

to:

```swift
instance.setFoo(42).setBar("xyzzy")
```

Traditional setters are far easier and require far less boilerplate code than chain-able setters.

### Error Handling

Swift 2's `do`/`try`/`catch` mechanism is fantastic. Use it. (TODO: elaborate and provide examples)

### Avoid `try!`

In general prefer:

```swift
do {
    try somethingThatMightThrow()
}
catch {
    fatalError("Something bad happened.")
}
```

to:

```swift
try! somethingThatMightThrow()
```

Even though this form is far more verbose it provides context to other developers reviewing the code.

It is okay to use `try!` as a temporary error handler until a more comprehensive error handling strategy is evolved. But it is suggested you periodically sweep your code for any errant `try!` that might have snuck past your code reviews.

### Avoid `try?` where possible

`try?` is used to "squelch" errors and is only useful if you truly don't care if the error is generated. In general though, you should catch the error and at least log the failure.

### Early Returns & Guards

When possible, use `guard` statements to handle early returns or other exits (e.g. fatal errors or thrown errors).

Prefer:

```swift
guard let safeValue = criticalValue else {
    fatalError("criticalValue cannot be nil here")
}
someNecessaryOperation(safeValue)
```

to:

```swift
if let safeValue = criticalValue {
    someNecessaryOperation(safeValue)
} else {
    fatalError("criticalValue cannot be nil here")
}
```

or:

```swift
if criticalValue == nil {
    fatalError("criticalValue cannot be nil here")
}
someNecessaryOperation(criticalValue!)
```

This flattens code otherwise tucked into an `if let` block, and keeps early exits near their relevant condition instead of down in an `else` block.

Even when you're not capturing a value (`guard let`), this pattern enforces the early exit at compile time. In the second `if` example, though code is flattened like with `guard`, accidentally changing from a fatal error or other return to some non-exiting operation will cause a crash (or invalid state depending on the exact case). Removing an early exit from the `else` block of a `guard` statement would immediately reveal the mistake.

### "Early" Access Control

Even if your code is not broken up into independent modules, you should always be thinking about access control. Marking a definition as "private" or "internal" can act as lightweight documentation for your code. Anyone reading the code will know that these elements are "hands off". Conversely, marking a definition as "public" is an invite for other code to access the marked elements. It is best to be explicit and not rely on Swift's default access control level ("internal").

If your codebase grows in the future, it may end being broken down into sub-modules. Doing so on a codebase already decorated with access control information is much quicker and easier.

### "Restrictive" Access Control

It is generally better to be overly restrictive when adding access control to your code. Where it makes sense prefer "private" definitions to "internal", and prefer "internal" to "public" (note: "internal" is the default).

It is far easier to change the access control of your code to be more permissive later (along the spectrum: "private" to "internal" to "public") as needed. Code that is has too permissive access control might be used inappropriately by other code. Making code more restrictive could involve finding the inappropriate or incorrect uses and providing better interfaces. This is a trying to close the stable door after the horse has bolted style problem. An example of this could be a type exposing an internal cache publicly.

Furthermore, restricting access to code limits the "exposed surface area" and allows the code to be refactored with less chance of impacting other code. Other techniques such as "Protocol Driven Development" can also help.

## TODO Section

This is a list of headings for possible future expansion.

### Protocols & Protocol Driven Development

### Implicitly Unwrapped Optionals

### Reference vs Value Types

### Async Closures

### `unowned` vs `weak`

### Cocoa Delegates

### Immutable Structs

### Instance Initialisation

### Logging & Printing

### Computed Properties vs Functions

### Value Types and Equality

[SwiftGraphics]: https://github.com/schwa/SwiftGraphics/blob/develop/Documentation/Notes.markdown
[slack]: http://swift-lang.schwa.io
[Swift_Programming_Language]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/index.html
