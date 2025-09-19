# Animation-in-SwiftUI

#### Table of contents
1. [Principle](#principle)
2. [Transition](#Transition)
3. [Animation Types](#animation-types)
4. [Advanced Animations](#Advanced-Animations)
5. [Canvas](#Canvas)
6. [Positioning](#Positioning)
7. [Animatable Protocol](#Animatable Protocol)

# Principle
- **State-driven animations**
	- Animations in SwiftUI happen when a **state variable changes**.
	- Flow: views re-render → SwiftUI interpolates the change → animation happens

# Explicit & Implicit Animation
- **Implicit**: animations happen _automatically_ on every state change without being directly commanded
``` swift
	.scaleEffect(isAnimating ? 1.5 : 1.0)
	.animation(.easeInOut.repeatForever(autoreverses: true),
	value: isAnimating)
```
- **Explicit**: explicitly wrap the state change inside an animation command
``` swift
	withAnimation(
	.linear(duration: 2.0) .repeatForever(autoreverses: false)
	) {
		isAnimating.toggle()
	}
```
# Animation Types
1. **Basic**
``` swift
// Linear - constant speed
.linear(duration: 1.0)

// Ease In - starts slow, ends fast  
.easeIn(duration: 1.0)

// Ease Out - starts fast, ends slow
.easeOut(duration: 1.0)

// Ease In Out - starts slow, fast in middle, ends slow
.easeInOut(duration: 1.0)
```

2. Spring Animation
``` swift
// Default spring
.spring()

// Spring with custom parameters
.spring(
    response: 0.6,      // Animation duration (0.1 - 2.0)
    dampingFraction: 0.8, // How bouncy (0.0 - 1.0)
    blendDuration: 0     // Blend with other animations
)

// Spring with mass, stiffness, damping
.interpolatingSpring(
    mass: 1.0,      // Object mass (higher = slower)
    stiffness: 100, // Spring stiffness (higher = faster)
    damping: 10,    // Damping (higher = less bounce)
    initialVelocity: 0
)
```

3. Custom timing curves
```swift
// Custom Bezier curve
.timingCurve(0.25, 0.1, 0.25, 1.0, duration: 1.0)

// Preset timing curves
.timingCurve(.easeIn, duration: 1.0)
.timingCurve(.easeOut, duration: 1.0)
.timingCurve(.easeInOut, duration: 1.0)
.timingCurve(.linear, duration: 1.0)
```

4. Animation Modifiers
```swift
// Repeating animations
.repeatForever(autoreverses: true)   // Back and forth forever
.repeatForever(autoreverses: false)  // Restart from beginning forever
.repeatCount(3, autoreverses: true)  // Repeat 3 times

// Delay
.delay(0.5)  // Wait 0.5 seconds before starting

// Speed control
.speed(2.0)  // 2x faster
.speed(0.5)  // 2x slower
```

## Animatable Properties
1. Geometric Properties
```swift
// Position & Offset
.offset(x: value, y: value)
.position(x: value, y: value)

// Size & Scale
.frame(width: value, height: value)
.scaleEffect(value)
.scaleEffect(x: value, y: value)

// Rotation & Transform
.rotationEffect(.degrees(value))
.rotation3DEffect(.degrees(value), axis: (x: 0, y: 1, z: 0))
```

2. Visual Properties
```swift
// Opacity & Visibility
.opacity(value)           // 0.0 - 1.0

// Colors
.foregroundColor(color)
.background(color)
.border(color, width: value)

// Blur & Effects
.blur(radius: value)
.brightness(value)        // -1.0 - 1.0
.contrast(value)          // 0.0 - 2.0+
.saturation(value)        // 0.0 - 2.0+
.hueRotation(.degrees(value))
```

3. Shape Properties
```swift
// Corner Radius
.cornerRadius(value)

// Clipping & Masking
.clipShape(Circle())
.mask(shape)

// Stroke & Fill (for Path/Shape)
.stroke(lineWidth: value)
.trim(from: startValue, to: endValue)  // 0.0 - 1.0
```

4. Layout Properties
```swift
// Padding & Spacing
.padding(value)
.padding(.horizontal, value)

// Aspect Ratio
.aspectRatio(value, contentMode: .fit)
```

# Transition
- Animate a view appearing/disappearing
``` .transition(.slide)
.transition(.opacity)
.transition(.scale)
.transition(.asymmetric(insertion: .slide, removal: .opacity))
  ```
- Combine with conditionals (`if show { ... }`) to animate view entry/exit

# MatchedGeometryEffect
- Used for **morphing one view into another** with smooth transitions
- Lets **two views share the same "geometry ID"**, so SwiftUI can smoothly animate changes between them
- `isSource` parameter: the one set to `false` adjusts its frame size to match the source

## Setup
1. Must define variable with `@Namespace` annotation --> type: `Namespace.ID`
2. Apply to views you want to animate

## Parameters `matchedGeometryEffect()`
1. **`id`** <span style="color: red">**required**</span>
    - Identifier that groups 2 frames to animate together
    - 1 frame can have 2 IDs --> use ternary operator (`... ? ... : ...`)
2. **`in`** <span style="color: red">**required**</span>
    - Variable that holds the `@Namespace`
3. **`properties`** <span style="color: lightblue">optional</span>
    - Type: enum `MatchedGeometryProperty` {`.frame`, `.position`, `.size`}
    - Default: `.frame`
4. **`anchor`** <span style="color: lightblue">optional</span>
    - Center point of the component
    - Type: `UnitPoint` enum (`.center`, `.top`, `.bottom`, etc.)
    - Default: `.center`
5. **`isSource`** <span style="color: red">**required**</span>
    - One of the geometry should be set to `true`
    - Source = the reference, false = the one that follows

## Example
```swift
struct CardView: View {
    @Namespace private var cardNamespace
    
    var body: some View {
        ZStack {
            Rectangle()
                .frame(width: 50, height: 80)
                .matchedGeometryEffect(id: "card", in: cardNamespace, isSource: true)
                .opacity(isExpanded ? 0 : 1)
            
            Rectangle()
                .frame(width: 300, height: 200)
                .matchedGeometryEffect(id: "card", in: cardNamespace, isSource: false)
                .opacity(isExpanded ? 1 : 0)
        }
        .onTapGesture {
            withAnimation(.spring()) {
                isExpanded.toggle()
            }
        }
    }
}
```
## Key Rules
- Same `id` for views you want to animate
- Same `namespace` for views you want to animate
- Only 1 view should be `isSource: true` at a time
- Always wrap with `withAnimation()`

# TimelineView(.animation)
- A view that updates its content based on a timeline schedule
- Automatically refreshes content at specified intervals
- Perfect for animations, clocks, timers, and live data updates
- Available from iOS 15+
## Basic syntax
```swift
TimelineView(schedule) { context in
    
}
```
**context properties:**
- context.date --> current date/time
- context.cadence --> how often updates happen
## Timeline Schedules
1. `.periodic(from:by:)`
```swift
// Updates every second
TimelineView(.periodic(from: Date(), by: 1.0)) { context in
    Text(context.date.formatted(date: .omitted, time: .standard))
}

// Updates every 0.1 seconds (smooth animation)
TimelineView(.periodic(from: Date(), by: 0.1)) { context in
    Circle()
        .rotationEffect(.degrees(context.date.timeIntervalSince1970 * 360))
}
```

2. `.everyMinute`
```swift
// Updates at the start of every minute
TimelineView(.everyMinute) { context in
    Text(context.date.formatted(.dateTime.hour().minute()))
}
```

3. `.animation`
```swift
// Updates as fast as possible (60fps-like)
TimelineView(.animation) { context in
    let seconds = context.date.timeIntervalSince1970
    Rectangle()
        .rotationEffect(.degrees(seconds * 90))
}
```

4. `.explicit(dates)`
```swift
// Updates at specific dates/times
let updateTimes = [
    Date(),
    Date().addingTimeInterval(5),
    Date().addingTimeInterval(10)
]

TimelineView(.explicit(updateTimes)) { context in
    Text("Updated at: \(context.date.formatted())")
}
```

# PhaseAnimator, KeyframeAnimator
# Animatable/VectorArithmetic

# Graphics

## Shapes
**Build in shapes**: Rectangle(), RoundedRectangle(cornerRadius: 25), UnevenRoundedRectangle(cornerRadii: .init(topLeading: 50, topTrailing: 50)), Capsule(), Ellipse(), Circle().
**Shape Modifiers**:
- `.fill(Color.blue)`
- `.stroke(Color.red, lineWidth: 2)` --> Outline 
- `.frame(width: 100, height: 100)` --> Size

### Combine shapes to create new shape
**Shape operations**:
- union
- intersection
- line intersection

**Example:**
```swift
struct CombinedShape: Shape {
    func path(in rect: CGRect) -> Path {
        let rectangle = Rectangle().path(in: rect)
        let circle = Circle().path(in: CGRect(
            x: rect.width * 0.25,
            y: rect.height * 0.25,
            width: rect.width * 0.5,
            height: rect.height * 0.5
        ))
        
        return rectangle.subtracting(circle)
    }
}
```

### Custom path
The shape types above are classes that conform to the **Shape** protocol. So we can create our own class with Shape protocol implementation, allowing us to use functions like `fill()`, `stroke()`, etc.
- Coordinate system: (0,0) is at top-left corner

**[Protocol](https://www.hackingwithswift.com/sixty/9/1/protocols) Shape Requirements**
Protocol Shape just needs 1 function implementation
```swift
func path(in rect: CGRect) -> Path
```
- `rect`: area where the drawing should fit. For example `CGRect(x: 0, y: 0, width: 200, height: 200)`
- `return`: `Path` → "raw drawing" (no color/fill/stroke, just outline)

**Example:**
```swift
struct Triangle: Shape {
func path(in rect: CGRect) -> Path {
		var path = Path()
		path.move(to: CGPoint(x: rect.midX, y: rect.minY))  // top point 
		path.addLine(to: CGPoint(x: rect.maxX, y: rect.maxY))  // bottom right 
		path.addLine(to: CGPoint(x: rect.minX, y: rect.maxY))  // bottom left 
		path.closeSubpath()  // close the path
		return path
		}
}
```
> you can also convert old path from UIKit/CoreGraphics to Path in SwiftUI

**Path methods:**
```swift
var path = Path()

// Basic drawing
path.move(to: CGPoint(x: 0, y: 0))           // Move to point
path.addLine(to: CGPoint(x: 100, y: 0))      // Draw line
path.closeSubpath()                          // Close shape

// Curves
path.addQuadCurve(
    to: CGPoint(x: 100, y: 100),
    control: CGPoint(x: 50, y: 0)
)

path.addCurve(
    to: CGPoint(x: 200, y: 100),
    control1: CGPoint(x: 150, y: 0),
    control2: CGPoint(x: 175, y: 50)
)

// Add shapes
path.addRect(CGRect(x: 0, y: 0, width: 50, height: 50))
path.addEllipse(in: CGRect(x: 0, y: 0, width: 100, height: 50))
```
## Canvas (iOS 15+)
**Why use `Canvas`?**
- For **custom graphics** that built-in shapes can’t handle.
- For **dynamic visualizations** (charts, waves, particles).
- For **game-like effects** (background animations, sprite drawing).
- For **performance**: `Canvas` is GPU-accelerated, better than stacking 1000 SwiftUI shapes
Canvas Parameters:
- `context`: `GraphicsContext` - provides drawing methods
- `size`: `CGSize` - canvas dimensions
### Basic Shapes
```swift
Canvas { context, size in
    // Fill circle
    context.fill(
        Path(ellipseIn: CGRect(x: 0, y: 0, width: 100, height: 100)),
        with: .color(.blue)
    )
    
    // Stroke rectangle
    context.stroke(
        Path(rect: CGRect(x: 50, y: 50, width: 100, height: 50)),
        with: .color(.red),
        lineWidth: 3
    )
}
```

### Transforms
```swift
Canvas { context, size in
    context.translateBy(x: 100, y: 100)
    context.scaleBy(x: 1.5, y: 1.5)
    context.rotate(by: .degrees(45))
    
    context.fill(
        Path(rect: CGRect(x: -25, y: -25, width: 50, height: 50)),
        with: .color(.purple)
    )
}
```

### Text and Images

```swift
Canvas { context, size in
    // Text
    context.draw(
        Text("Hello"),
        at: CGPoint(x: 100, y: 100),
        anchor: .center
    )
    
    // Images (must be resolved)
    if let symbol = context.resolveSymbol(id: "star") {
        context.draw(symbol, at: CGPoint(x: 50, y: 50))
    }
} symbols: {
    Image(systemName: "star.fill")
        .tag("star")
}
```
### Canvas vs Shape decision guide
- Canvas = dynamic, complex, pixel-level control
- Shape = static, reusable, SwiftUI-integrated
## Gradients
- Linear Gradient
- Radial Gradient
- Angular Gradient
- Mesh Gradient (iOS 18+)

# Positioning
## Offset
- Moves a view relative to its original position
- Did not affect the layout
- **Great for** animations or **temporary** movement
## Position
- **Sets the absolute position of the center of the view** inside its _parent’s coordinate_ space
- Affects where the view “lives” in the layout tree.
- Doesn’t reserve space on its original spot
## Inset in SwiftUI
1. Safe Area Insets
```swift
VStack {
    Text("Full Screen Content")
}
.ignoresSafeArea()
.ignoresSafeArea(.container, edges: .top)
.ignoresSafeArea(.keyboard)
```
2. Padding (edge inset)
```swift
Text("Hello")
    .padding() // 16pt all
    .padding(20) // 20pt
    .padding(.horizontal, 24)
    .padding(EdgeInsets(top: 8, leading: 16, bottom: 8, trailing: 16))
```
3. Content margin
```
.contentMargins(20)
.contentMargins(.horizontal, 24)
```
	
> **Safe area values** vary by device (iPhone: 20-59pt top, 0-34pt bottom)

>`.safeAreaInset()` modifier --> lets us place content _outside_ the device’s safe area, while automatically adjusting other views so their content remains visible.
>```
> VStack {}
	.safeAreaInset(edge: .bottom) {
        Text("Outside Safe Area")
            .font(.largeTitle)
            .foregroundStyle(.white)
            .frame(maxWidth: .infinity)
            .padding()
            .background(.indigo)
    } ```

when you do this : `Circle.inset(10)` --> it _shrinks_ the shape inward instead of cutting it of.
## Get the device screen size?
SwiftUI
with **GeometryReader** --> the **actual layout space** your view gets
```
GeometryReader { proxy in
	Text("Size: \(proxy.size.width) x \(proxy.size.height)")
}
```
UIKit
**UIScreen.main.bounds.width** --> get the whole screen size
```
Text("Screen size: \(UIScreen.main.bounds.width) x \(UIScreen.main.bounds.height)")
```

In SwiftUI, we almost never need the absolute screen size unless you’re doing something custom (games, Canvas drawing, etc.). Layouts are better left to things like `Spacer()`, `.frame()`, or `GeometryReader`.
- **`.frame()`** → kasih batasan ukuran (misalnya maxWidth `.infinity`).
- **`Spacer()`** → biarin sistem layout yang bagi ruang kosong.
- **`GeometryReader`** → dipakai kalau memang butuh tau ukuran/posisi (misalnya chart atau custom drawing).

> **Performance notes**
> Use GeometryReader too much on the screen will impacting the app performance

# Animatable Protocol
looks like this: `@Animatable`
animatable is a type that describes how to animate a property of a view.

? animatable vs animatable macro


# Geometry Reader
trade off: if we use it too much on the screen, it will impacting in app performance

# Visual Effects
--> it's a view modifier with closure
--> `content` and `proxy` is a parameter closure
```swift
.visualEffect { content, proxy in
                    content
                        .rotation3DEffect(.degrees(-proxy.frame(in: .global).minX) / 8, axis: (x: 0, y: 1, z: 0))
                }
```

--> similar like using GeometryReader, but it's not GeometryReader. GeometryReader is the older way.
return:
- `content`: the view being modified
- `geometryProxy`: a `GeometryProxy` that provides access to the frame, size, position, etc


# Scroll Effect
```swift
scrollView() { item in
	Item()
		.scrollTransation(
			axis: .horizontal | .vertical  // parameter
			) { content, phase in // parameter closure // phase has a type of `ScrollTransitionPhase`
				content
					.rotationEffect(degrees(phase.value * 2.5))
					.offset(y: phase.isIdentity ? 0 : 8)
		
		}}
```
- ScrollTransitionPhase phases: 
	- `.identity` - when the view is fully visible within the scroll container, usually with no visual effect. It doesn’t have to be just one item; multiple items can be in the identity phase at once.
	- `.topLeading` - when the view is about to enter the visible area from the top (in vertical scroll) or from the leading edge (in horizontal scroll)
	- `.bottomTrailing` - when the view is about to leave the visible area through the bottom (in vertical scroll) or the trailing edge (in horizontal scroll)
- ScrollTransitionPhase properties:
	- `.isIdentity` - a Boolean value indicating whether the phase is currently in the identity state (fully visible)
	- `.value` - a Double value ranging from `-1` to `1`, where:
	    - `-1` = topLeading phase
		- `0` = identity phase
		- `1` = bottomTrailing phase



# Custom modifier

# Common Animation Pattern
```swift
// Conditional animation
.animation(isEnabled ? .spring() : .none, value: someProperty)

// Different animations for different properties
.animation(.spring(), value: position)
.animation(.easeOut, value: opacity)

// Chained animations
withAnimation(.easeIn(duration: 0.3)) {
    firstProperty = newValue
}
DispatchQueue.main.asyncAfter(deadline: .now() + 0.3) {
    withAnimation(.easeOut(duration: 0.3)) {
        secondProperty = newValue
    }
}

// State-driven animations
@State private var animationState: AnimationState = .idle

enum AnimationState {
    case idle, loading, success, error
}

var animationValue: CGFloat {
    switch animationState {
    case .idle: return 1.0
    case .loading: return 1.2
    case .success: return 0.9
    case .error: return 1.1
    }
}
```
# Other
- Inset = the inner margin, i.e., space pushed inward. With UIEdgeInsets, you can define top, bottom, left, and right values.
  - Inset: the frame moves inward, making the visible image appear smaller.
  - Offset: you shift the drawing (like moving a painting) left, right, up, or down.
- CGRect = a fundamental structure representing a rectangle’s position and size. Defined by an origin (CGPoint containing x and y) and a size (CGSize containing width and height). Core Graphics data structures (all part of Core Graphics):
- <u>CGSize, CGRect and CGPoint, CGVector</u> is a data **structure**.
  - `CGPoint` → represents a point in a 2D coordinate system. Example: CGPoint(x: 10, y: 10)
  - `CGSize` → represents a width and height. Example: CGSize(width: 50, height: 30) (values can be negative).
  - `CGRect` → typically used to frame a UIView, with coordinates relative to its parent view.
  - `CGVector` → often used in animations, transformations, and visual effects. Example: CGVector(dx: 10, dy: 20)
```swift
public struct CGRect {  
   public var origin: CGPoint  
   public var size: CGSize  
   public init()  
   public init(origin: CGPoint, size: CGSize)  
}
```
- Usecase of CG... in swiftui:
  - GeometryReader
  - Custom drawing with Canvas
  - ScrollViewReader (use case: auto scroll when a specific view appear [use case](https://gist.github.com/nikihdyt/1082f8f7bd7394844d01a40dfded8fef))
  - Positioning elements
