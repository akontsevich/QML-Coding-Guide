# Table of Contents

- [Introduction](#introduction)
- [Code Style](#code-style)
    + [Scalability](#scalability)
        * [Scalability techniques overview](#scalability-techniques-overview)
        * [Items sizes proportional to default font sizes](#items-sizes-proportional-to-default-font-sizes)
        * [Text fields font size](#text-fields-font-size)
        * [Item stretchable to inner text size metrics](#item-stretchable-to-inner-text-size-metrics)
        * [Position Elements With Anchors](#position-elements-with-anchors)
        * [Positioners vs Layouts](#positioners-vs-Layouts)
        * [Load components on demand](#load-components-on-demand)
    + [QML Object Declarations](#qml-object-declarations)
    + [Signal Handler Ordering](#signal-handler-ordering)
    + [Property Initialization Order](#property-initialization-order)
    + [Function Ordering](#function-ordering)
    + [Animations](#animations)
    + [Specifying IDs for Objects](#specifying-ids-for-objects)
    + [Property Assignments](#property-assignments)
    + [Import Statements](#import-statements)
    + [Full Example](#full-example)
    + [Check for best practice compliance with `qmllint`](#check-for-best-practice-compliance-with-qmllint)
- [Properties](#properties)
    + [Use types](#use-types)
    + [Required properties](#required-properties)
    + [Avoid parent and other generic properties](#avoid-parent-and-other-generic-properties)
    + [Use qualified property lookup](#use-qualified-property-lookup)
- [Bindings](#bindings)
    + [Prefer Bindings over Imperative Assignments](#prefer-bindings-over-imperative-assignments)
    + [Making `Connections`](#making-connections)
    + [Use `Binding` Object](#use-binding-object)
    + [KISS It](#kiss-it)
    + [Avoid Unnecessary Re-Evaluations](#avoid-unnecessary-re-evaluations)
- [C++ Integration](#c-integration)
    + [Avoid Context Properties](#avoid-context-properties)
    + [Use Singleton for Common API Access](#use-singleton-for-common-api-access)
    + [Prefer Instantiated Types Over Singletons For Data](#prefer-instantiated-types-over-singletons-for-data)
    + [Watch Out for Object Ownership Rules](#watch-out-for-object-ownership-rules)
- [Performance and Memory](#performance-and-memory)
    + [Reduce the Number of Implicit Types](#reduce-the-number-of-implicit-types)
- [Signal Handling](#signal-handling)
    + [Try to Avoid Using connect Function in Models](#try-to-avoid-using-connect-function-in-models)
    + [When to use Functions and Signals](#when-to-use-functions-and-signals)
- [JavaScript](#javascript)
    + [Use Arrow Functions](#use-arrow-functions)
    + [Use the Modern Way of Declaring Variables](#use-the-modern-way-of-declaring-variables)
- [States and Transitions](#states-and-transitions)
    + [Don't Define Top Level States](#dont-define-top-level-states)
- [Visual Items](#visual-items)
    + [Distinguish Between Different Types of Sizes](#distinguish-between-different-types-of-sizes)
    + [Be Careful with a Transparent `Rectangle`](#be-careful-with-a-transparent-rectangle)


# Introduction
Coding guide is based on my experience working on big Qt/QML projects, 
some good examples and common sense first of all, as well as on the following 
sources, best practices and recommendations:

  - Original [Furkanzmc/QML-Coding-Guide](https://github.com/Furkanzmc/QML-Coding-Guide) content
  - Qt best practices:
    + [Best Practices for QML and Qt Quick](https://doc.qt.io/qt-6/qtquick-bestpractices.html)
    + [Performance Considerations And Suggestions](https://doc.qt.io/qt-6/qtquick-performance.html)
    + [Scalability](https://doc.qt.io/qt-6/scalability.html)
    + [QML Application Structuring Approaches](https://wiki.qt.io/QML_Application_Structuring_Approaches) (some info is actual, some outdated)
    + [10 Tips to Make Your QML Code Faster and More Maintainable | KDAB](https://www.kdab.com/10-tips-to-make-your-qml-code-faster-and-more-maintainable/)
    + [Best Practices in Writing Applications in QML | User Interface | #QtWS21 - YouTube](https://www.youtube.com/watch?v=mImptIBmWW0)
  - [QGroundControl Coding Style](https://github.com/mavlink/qgroundcontrol/blob/master/CodingStyle.qml)

This write-up summarizes best practices towards good user experience, 
UI look-and-feel, scalability, performance, much fewer errors, extendability and 
maintenance. They are to be applied early in the development cycle in order to 
avoid technical debt and _costly_ refactoring later. Also document includes
justifications for this or that technical decisions.

# Code Style

## Scalability

### Scalability techniques overview
When we develop applications for several different mobile device platforms, 
we may face the following challenges:

  - Mobile device platforms support devices with varying screen configurations: 
    size, aspect ratio, orientation, and density.
  - Different platforms have different UI conventions and you need to meet the 
    users' expectations on each platform.
    
You need to consider scalability when:

  - You want to deploy your application to more than one device platform, 
  such as Android and iOS, or more than one device screen configuration.
  - Your want to be prepared for new devices that might appear on the market 
  after your initial deployment.

So since display resolutions improve, a scalable application UI becomes more and 
more important. While Qt provides only some 
[general recommendations](https://doc.qt.io/qt-6/scalability.html) 
without good code examples for them, some of advices are very suboptimal: 
one of the approaches is to maintain several copies of the UI for different screen resolutions, and load the appropriate one depending on the available resolution. 
This adds significant development and maintenance overhead.

Considering scalability feature for the UI may significantly impact on the overal 
application architecture and components design so should be taken into account 
on very early development stage to avoid further technical debt and refactoring.

There is no unique, universal or the best aprroach organizing scalability while 
Qt Quick allows to develop applications that can run on different types of devices, 
screen sizes, aspect ratio, orientation, DPI, etc. Application can cope with 
different screen configurations. However, there is always a certain amount 
of fixing and polishing needed to create an optimal user experience for each 
target platform.

[Good scalable UI organization example](https://github.com/mavlink/qgroundcontrol/blob/master/CodingStyle.qml) provided by QGroundControl - 
Cross-platform ground control station for drones (Android, iOS, Mac OS, Linux, 
Windows). There are a lot of [QML controls](https://github.com/mavlink/qgroundcontrol/blob/master/src/QmlControls/) to learn scalability and positioning from.

### Items sizes proportional to default font sizes
So for the above type of scalability organization, items sizes and 
layouts/positioners spacing should be proportional to default font sizes. 
Example:

```qml
Item {
    // Property binding to item properties
    width:  AppConstantsSingleton.defaultFontPixelHeight * 10 
    // No hardcoded sizing. All sizing must be relative to default font size
    height: AppConstantsSingleton.defaultFontPixelHeight * 20
}
```

### Text fields font size
A `Text` QML type attempts to determine how much room is needed and set the 
width and height properties accordingly, unless they are explicitly set. This 
fact could be used in developing scalable UI, which will be shown in sections below.

Since we rely on font pixel size there, hence for consistency in scalable 
interfaces in bindings need to use `Text.font.pixelSize` everywhere, which 
gives us exact predictive Text box height in pixels for sizes supervision, and 
avoid using conflicting `pointSize` property which gives different pixel sizes 
dependent according to screen size, DPI, etc. QML always warn if we try to set 
both in a control.

```qml
    Text {
        anchors.centerIn: parent
        text: "Hello world!"
        font.pixelSize: AppConstantsSingleton.defaultFontPixelHeight
    }
```

### Item stretchable to inner text size metrics
Another type of scaling example where Item scales to child Text item size:

```qml
    Rectangle {
        color:  "green"
        width:  t_metrics.tightBoundingRect.width
        height: t_metrics.tightBoundingRect.height

        Text {
            id:     a_text
            text:   "r"
            color:  "white"
            anchors.centerIn: parent
        }

        TextMetrics {
            id:     t_metrics
            font:   a_text.font
            text:   a_text.text
        }
    }
```

### Text fits into an Item
Of course there could be an opposite situation where need to fit text into
parent control predefined size. For this purpose 
[`Text.fontSizeMode` property](https://doc.qt.io/qt-6/qml-qtquick-text.html#fontSizeMode-prop) 
could be used. This property specifies how the font size of the displayed text 
is determined. The font size of fitted text has a minimum bound specified by 
the `minimumPointSize` or `minimumPixelSize` property and maximum bound specified 
by either the `font.pointSize` or `font.pixelSize` properties.

```qml
Text { text: "Hello"; fontSizeMode: Text.Fit; minimumPixelSize: 10; font.pixelSize: 72 }
```
However this approach has limitation as if the text does not fit within the 
item bounds with the minimum font size the text will be elided as per the 
elide property.

Other way is to calculate font size relative to parent item:

```qml
    Rectangle {
        anchors.centerIn: parent
        width: root.width * 0.5
        height: root.height * 0.5
        color: 'green'

        Text {
            id: field
            anchors.centerIn: parent
            width: parent.width
            height: parent.height
            text: "Size me!"
            color: 'white'
            font.pixelSize: field.width / 10 // resize relative to parent
        }
    }
```
**Note.** And again we specify font size in pixels above (`font.pixelSize`).

### Position Elements With Anchors
If the layout is dynamic, the most performant and efficient way to specify 
the layout is to use anchors rather than bindings to position items 
relative to each other. Positioning with bindings (by assigning binding 
expressions to the `x`, `y`, `width` and `height` properties of visual objects, 
rather than using anchors) is relatively slow, although it allows maximum 
flexibility.

And vice versa: if the layout is not dynamic, the most performant way to specify 
the layout is via static initialization of the x, y, width and height properties. 

### Positioners vs Layouts
As long as items should be proportional to the default font size, 
[Positioner](https://doc.qt.io/qt-6/qml-qtquick-positioner.html)s 
([Row](https://doc.qt.io/qt-6/qml-qtquick-row.html), 
[Column](https://doc.qt.io/qt-6/qml-qtquick-column.html), 
[Grid](https://doc.qt.io/qt-6/qml-qtquick-grid.html), 
[Flow](https://doc.qt.io/qt-6/qml-qtquick-flow.html)) 
considered preferable comparing to layouts as positioners manage only items 
position &ndash; not their size. Positioners stretch their size according to 
content items, which is suitable, for example, for dynamic scrollable pages, 
lists, etc where we do not know parent size before hand and want parent size 
rises according to content and do not want items to be resizeable as they 
already scales according to font sizes.

**Example**. Stretchable 2x2 Grid which prints map parameters titles and their values

```qml
    Grid {
        columns: 2
        columnSpacing: ScreenTools.defaultFontPixelHeight * 2

        Label {
            text: qsTr("Tile Count:")
            font.pixelSize: ScreenTools.smallFontPixelHeight
        }
        Label {
            text: QGroundControl.mapEngineManager.tileCountStr
            font.pixelSize: ScreenTools.smallFontPixelHeight
            color: _tooManyTiles ? "red" : qgcPal.text
        }

        Label {
            text: qsTr("Size (est.):")
            font.pixelSize: ScreenTools.smallFontPixelHeight
        }
        Label {
            text: QGroundControl.mapEngineManager.tileSizeStr
            font.pixelSize: ScreenTools.smallFontPixelHeight
            color: _tooManyTiles ? "red" : qgcPal.text
        }
    }
```

### Load components on demand
To implement scalable applications using Qt Quick load components on demand by 
using a `Loader` is suggested as well. This also provides code reusing and 
universalism and some useful side effect like for QML bindings:

  - few times(!!!) less coding: single `Loader` used for many Items (pages), 
    pages could be loaded via a model.
  - much simpler logic
  - automatic destroying/hiding of dependent objects/items
  - etc.
  
**Example**. Loader with model
```qml
    ListView {
        id: settingsPager
        anchors {
            top: parent.top;
            bottom: parent.bottom
            horizontalCenter: parent.horizontalCenter;
            topMargin: ScreenTools.defaultFontPixelHeight
        }
        width: parent.width * 0.9
        spacing: ScreenTools.defaultFontPixelHeight * 2
        clip: true
        indicatorEnabled: false

        model: ["qrc:/qml/Controls/UnitsSettings.qml",
                "qrc:/qml/Controls/MissionParameters.qml",
                "qrc:/qml/Controls/GeneralSettings.qml"]

        delegate: Item {
            id: element
            width: settingsPager.width
            height: sectionsLoader.height

            Loader {
                id: sectionsLoader
                anchors.top: parent.top;
                width: parent.width
                source: modelData
            }
        }
    }
```

## QML Object Declarations
This section provides details about how to format the order of properties, signals,
and functions to make things easy on the eyes and quickly switch to related code block.

Though in Qt documentation and examples, [QML object attributes](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html) are always structured in the [following order](https://doc.qt.io/qt-6/qml-codingconventions.html#qml-object-declarations) I found it inconsistent as 
`id` is also a property and we got a gap in declaration: as all object 
properties initialization should go in one logical block. Also it is not quite 
suitable for scalable interfaces as there object properties modifies standart 
and actually defines new item (object) behavior as stretching is one of the main 
function (primary) of a control in such UI, therefore first look on a control 
should show how it works. Qt guys also mention 
[these conventions are wrong and outdated](https://github.com/Furkanzmc/QML-Coding-Guide/issues/4#issue-524411345) 
and use natural ordering without such breaks in declaration blocks.


For consistency [QML object attributes](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html)
should be structured in the following order:

- id (and `objectName` if required for unit or Squish testing)
- properties:
    + object property initializations without a gap with `id`
    + custom object properties and property aliases that define our interface.
    + private properties with undercored names if necessary
    + attached properties 
- Signal declarations
- Signal handlers
- Attached signal handlers
- JavaScript functions which are "public" interfaces of an object (control)
- Child objects
  + Visual Items
  + Qt provided non-visual items
  + Custom non-visual items
- States
- Transitions
- `QtObject` for encapsulating private members[1](https://bugreports.qt.io/browse/QTBUG-11984)
  + "private" (internal) JavaScript functions 

The main purpose for this order is to make sure that the most intrinsic properties of a type is
always the most visible one in order to make the interface easier to digest at a first glance. 
So I prefer such an attributes order where 
[QGC coding style](https://github.com/mavlink/qgroundcontrol/blob/master/src/QmlControls/QGCButton.qml) 
is a very good organization example.

## Signal Handler Ordering

When handling the signals attached to an `Item`, make sure to always leave
`Component.onCompleted` to the last line.

```qml
// Wrong
Item {
    Component.onCompleted: {
    }
    onSomethingHappened: {
    }
}

// Correct
Item {
    onSomethingHappened: {
    }
    Component.onCompleted: {
    }
}
```

This is because it mentally makes for a better picture because
`Component.onCompleted` is expected to be fired when the components construction
is complete.

------

If there are multiple signal handlers in an `Item`, then the ones with least amount
of lines may be placed at the top. As the implementation lines increases, the handler
also moves down. The only exception to this is `Component.onCompleted` signal, it
is always placed at the bottom.

```qml
// Wrong
Item {
    onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
    }
    onSomethingHappened: {
        // Line 1
        // Line 2
    }
}

// Correct
Item {
    onSomethingHappened: {
        // Line 1
        // Line 2
    }
    onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
    }
}
```

## Property Initialization Order

The first property assignment must always be the `id` of the component. If you
want to declare custom properties for a component, the declarations are always
after properties assignments.

```qml
// Wrong
Item {
    someProperty: false
    property int otherProperty: -1
    id: myItem
}

// Correct
Item {
    id: myItem
    someProperty: false

    property int otherProperty: -1
}
```

There's also a bit of predefined order for property assignments. The order goes
as follows:

- id
- x, y or anchors
- width
- height

The goal here is to put the most obvious and defining properties at the top for
easy access and visibility. 

If there are also property assignments along with signal handlers, make sure to
always put property assignments above the signal handlers.

```qml
// Wrong
Item {
    onOtherEvent: {
    }
    someProperty: true
    onSomethingHappened: {
    }
    x: 23
    y: 32
}

// Correct
Item {
    x: 23
    y: 32
    someProperty: true
    
    onOtherEvent: {
    }
    onSomethingHappened: {
    }
}
```

It is usually harder to see the property assignments If they are mixed with
signal handlers. That's why we are putting the assignments above the signal
handlers.

### Function Ordering

Although there are no private and public functions in QML, you can provide a
similar mechanism by wrapping the properties and functions that are only supposed
to be used internally in `QtObject `.

Private function implementations are always put at the very bottom of the file 
(Sometimes this is also suggested by Qt in their examples). For public we
prioritize putting the declarations at the top of the file. While if the large 
function may significantly reduce the readability of the QML document modern 
editors may collapse it if necessary. Ideally, you shouldn't have any
functions at all and strive to rely on declarative properties of your component 
as much as possible.

### Animations

When using any subclass of `Animation`, especially nested ones like
`SequentialAnimation`, try to reduce the number of properties in one line.
More than 2-3 assignments on the same line becomes harder to reason with after
a while. Or maybe you can keep the one line assignments to whatever line length
convention you have set up for your project.

Since animations are harder to imagine in your mind, you will benefit from
keeping the animations as simple as possible.

```qml
// Bad
NumberAnimation { target: root; property: "opacity"; duration: root.animationDuration; from: 0; to: 1 }

// Depends on your convention. The line does not exceed 80 characters.
PropertyAction { target: root; property: "visible"; value: true }

// Good.
SequentialAnimation {
    PropertyAction {
        target: root
        property: "visible"
        value: true
    }

    NumberAnimation {
        target: root
        property: "opacity"
        duration: root.animationDuration
        from: 0
        to: 1
    }
}
```

### Specifying IDs for Objects

If an object does not need to be accessed for a functionality, avoid setting the `id` property.
This way you'll be less likely to run into duplicate `id` problem and reduces the 
code which is always suitable in big QML components. Also, having an id for an object puts additional cognitive stress because it now means that there's additional relationships that we
need to care for.

If you want to mark the type with a descriptor but you don't intend to reference
the type, you can use `objectName` instead or just plain old comments.

Make sure that the top most component in the file always has `root` as its `id`.
Qt will make unqualified name look up deprecated in QML 3, so it's better to
start giving IDs to your components now and use qualified look up.

See [QTBUG-71578](https://bugreports.qt.io/browse/QTBUG-71578) and
[QTBUG-76016](https://bugreports.qt.io/browse/QTBUG-76016) for more details
on this.

### Property Assignments

When assigning grouped properties, always prefer the dot notation If you are only
altering just one property. Otherwise, always use the group notation.

```qml
Image {
    anchors.left: parent.left // Dot notation
    sourceSize { // Group notation
        width: 32
        height: 32
    }
}
```

When you are assigning the component to a `Loader`'s `sourceComponent` in different
places in the same file, consider using the same implementation. For example, in
the following example there are two instances of the same component. If both of
those `SomeSpecialComponent` are meant to be identical it is a better idea to
wrap `SomeSpecialComponent` in a `Component`.

```qml
// BEGIN bad.
Loader {
    id: loaderOne
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}

Loader {
    id: loaderTwo
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}
// END bad.

// BEGIN good.
Loader {
    id: loaderOne
    sourceComponent: specialComponent
}

Loader {
    id: loaderTwo
    sourceComponent: specialComponent
}

Component {
    id: specialComponent

    SomeSpecialComponent {
        text: "Some Component"
    }
}
// END good.
```

This ensures that whenever you make a change to `specialComponent` it will take
effect in all of the `Loader`s. In the bad example, you would have to duplicate
the same change.

When in a similar situation without the use of `Loader`, you can use inline
components.

```qml
component SomeSpecialComponent: Rectangle {

}
```

### Import Statements

As a general rule, you should always prefer C++ over JavaScript to do heavy lifting. If there are
cases where you justify having a separate JavaScript file, keep these in mind.

If you are importing a JavaScript file, make sure to not include the same
module in both the QML file and the JavaScript file. JavaScript files share the
imports from the QML file so you can take advantage of that. If the JavaScript
file is meant as a library, this does not apply.

If you are not making use of the imported module in the QML file, consider moving
the import statement to the JavaScript file. But note that once you import something
in the JavaScript file, the imports will no longer be shared. For the complete
rules see [here](https://doc.qt.io/qt-6/qtqml-javascript-imports.html#imports-within-javascript-resources).

`Qt.include()` is [deprecated](https://doc.qt.io/qt-6/qml-qtqml-qt-obsolete.html#include-method)
and should not be used.

As a general rule, you should avoid having unused import statements.

#### Import Order

When importing other modules, use the following order;

- Qt modules
- Third party modules
- Local C++ module imports
- QML folder imports

### Full Example

```qml
// First Qt imports
import QtQuick 
import QtQuick.Controls 
// Then custom imports
import my.library 

Item {
    // ----- Property Declarations
    id: root
    anchors.top: parent.top // If a single assignment, dot notation can be used.
    x: 0
    y: 0
    z: 0
    width: 100
    height: 100
    enabled: true
    layer.enabled: true

    // Required properties should be at the top.
    required property int radius: 0

    property int radius: 0
    property alias color rect.color

    // ----- Then attached properties 
    Layout.fillWidth: true
    Drag.active: false
    
    // ----- Signal declarations
    signal clicked()
    signal doubleClicked()
    
    // ----- Public JavaScript functions
    function collapse() {

    }

    function setCollapsed(value: bool) {
        if (value === true) {
        }
        else {
        }
    }

    // ----- Signal handlers
    onWidthChanged: { // Always use curly braces.

    }
    
    // onCompleted and onDestruction signal handlers are always the last in
    // the order.
    Component.onCompleted: {

    }
    
    Component.onDestruction: {

    }

    // ----- Then attached signal handlers
    Drag.onActiveChanged: {

    }

    // ----- Visual children.
    Rectangle {
        id: rect
        anchors: { // For multiple assignments, use group notation.
            top: parent.top
            left: parent.left
            right: parent.right
        }
        height: 50
        color: "red"
        layer: {
            enabled: true
            samples: 4
        }
    }

    Rectangle {
        width: parent.width
        height: 1
        color: "green"
    }

    // ----- Qt provided non-visual children

    Timer {

    }

    // ----- Custom non-visual children

    MyCustomNonVisualType {

    }

    // ----- States and transitions.
    states: [
        State {

        }
    ]
    
    transitions: [
        Transitions {

        }
    ]

    QtObject {
        id: privates

        property int diameter: 0
    }

    // Private JavaScript functions or move this to the `privates` object
    function _doSomethingPrivate(x) {   
        ...                             
    }
}
```

## Check for best practice compliance with `qmllint`
[`qmllint`](https://doc.qt.io/qt-6/qtqml-tooling-qmllint.html) 
is a tool shipped with Qt, that verifies the syntatic validity of QML 
files. It also warns about some QML anti-patterns. qmllint warns about:

- Unqualified accesses of properties
- Usage of signal handlers without a matching signal
- Usage of with statements in QML
- Issues related to compiling QML code
- Unused imports
- Deprecated components and properties
- And many other things

# Properties

##  Use types
In order for `qmlcachegen` to generate efficient code for your bindings, it needs 
to know the type for properties. Avoid using `property var` wherever possible 
and use concrete types. This may be built-in types like `int`, `double`, or 
`string`, or any declaratively-defined custom type. Sometimes you want to be 
able to use a type as a property type in QML but don't want the type to be 
creatable from QML directly. For this, you can register them using the 
`QML_UNCREATABLE` macro.

```qml
property var size: 10 // bad
property int size: 10 // good

property var thing // bad
property MyThing thing // good
```

## Required properties
Use **required** properties to avoid undefined behavior errors during runtime. 
They are especially useful for delegates.

```qml
Rectangle {
    id: root
    required property string fontName

    Text {
        anchors: centerIn.parent
        font.family: root.fontName
        text: root.fontName
    }
}
```

Also it is always better to specify the required property type which guarantees
that object will exists on compile time and not lost in objects hierarchy 
tree. In example below, this also secures main window id potentially change 
if `mainWindow` id is called directly from the button.

```qml
    Button {
        required property ApplicationWindow mainWindow
        text: qsTr("Enter fullscreen")
        onClicked: mainWindow.showFullScreen()
    }
```

## Avoid parent and other generic properties
`qmlcachegen` can only work with the property types it knows at compile time. 
It cannot make any assumptions about which concrete subtype a property will 
hold at runtime. This means that, if a property is defined with type `Item`, 
it can only compile bindings using properties defined on `Item`, not any of its 
subtypes. This is particularly relevant for properties like `parent` or 
`contentItem`. For this reason, avoid using properties like these to look up 
items when not using properties defined on `Item` (properties like `width`, 
`height`, or `visible` are okay) and use look-ups via IDs instead.

```qml
    Item {
        id: thing

        property int size: 10

        Rectangle {
            width: parent.size // bad, Item has no 'size' property
            height: thing.height // good, lookup via id

            color: parent.enabled ? "red" : "black" // good, Item has 'enabled' property
        }
    }
```

## Use qualified property lookup
QML allows you to access properties from objects several times up in the parent 
hierarchy without explicitly specifying which object is being referenced. 
This is called an unqualified property look-up and generally considered bad 
practice since it leads to brittle and hard to reason about code. 
`qmlcachegen` also cannot properly reason about such code. So, it cannot properly 
compile it. You should only use qualified property lookups:

```qml
Item {
    id: root
    property int size: 10

    Rectangle {
        width: size // bad, unqualified lookup
        height: root.size // good, qualified lookup
    }
}
```

# Bindings

Bindings are a powerful tool when used responsibly. Bindings are evaluated
whenever a property it depends on changes and this may result in poor performance
or unexpected behaviors. Even when the binding is simple, its consequence can be
expensive. For instance, a binding can cause the position of an item to change
and every other item that depends on the position of that item or is anchored to
it will also update its position.

So consider the following rules when you are using bindings.

## Prefer Bindings over Imperative Assignments

See the related section on [Qt Documentation](https://doc.qt.io/qt-6/qtquick-bestpractices.html#prefer-declarative-bindings-over-imperative-assignments).

The official documentation explains things well, but it is also important to
understand the performance complications of bindings and understand where the
bottlenecks can be.

If you suspect that the performance issue you are having is related to
excessive evaluations of bindings, then use the QML profiler to confirm your
suspicion and only after this opt-in to use imperative option.

Refer to the [official documentation](https://doc.qt.io/qtcreator/creator-qml-performance-monitor.html)
on how to use QML profiler.

## Making `Connections`

A `Connections` object is used to handle signals from arbitrary `QObject` derived
classes in QML. One thing to keep in mind when using connections is the default
value of `target` property of the `Connections` is its parent if not explicitly
set to something else. If you are setting the target after dynamically creating
a QML object, you might want to set the `target` to `null` otherwise you might
get signals that are not meant to be handled.

Also note that using a `Connections` object will incur a slight performance/memory penalty since
it's another allocation that has to be done. If you are concerned about this you can use
`QtObject.connect` method, but [be careful](#try-to-avoid-using-connect-function-in-models) of
the pitfalls of this solution.

```qml
// Bad
Item {
    id: root
    onSomethingHappened: {
        // Set the target of the Connections.
    }

    Connections {
        // Notice that target is not set so it's implicitly set to root.
        function onWidthChanged(width) {
            // Do something. But since Item also has a width property we may
            // handle the change for root until the target is set explicitly.
        }
    }
}

// Good
Item {
    id: root
    onSomethingHappened: {
        // Set the target of the Connections.
    }

    Connections {
        target: null // Good. Now we won't have the same problem.
        function onWidthChanged(width) {
            // Do something. Only handles the changes for the intended target.
        }
    }
}
```

## Use `Binding` Object

`Binding`'s `when` property can be used to enable or disable a binding expression
depending on a condition. If the binding that you are using is complex and does
not need to be executed every time a property changes, this is a good idea to
reduce the binding execution count.

Using the same example above, we can rewrite it as follows using a `Binding` object.

```qml
Rectangle {
    id: root

    Binding on color {
        when: mouseArea.pressed
        value: mouseArea.pressed ? "red" : "yellow"
    }

    MouseArea {
        id: mouseArea
        anchors.fill: parent
    }
}
```

Again, this is a really simple example to get the point out. In a real life
situation, you would not get more benefit from using `Binding` object in this
case unless the binding expression is expensive (e.g It changes the item's
`anchor` which causes a whole chain reaction and causes other items to be
repositioned.).

## KISS It

A property binding expression will be re-evaluated if any of the properties it 
references are changed. As such, binding expressions should be kept as simple as 
possible ([KISS principle](https://en.wikipedia.org/wiki/KISS_principle)).
QML supports optimization of binding expressions. Optimized bindings do not require
a JavaScript environment hence it runs faster. The basic requirement for optimization
of bindings is that the type  information of every symbol accessed must be known at
compile time. So, avoid accessing `var` properties. You can see the full list of prerequisites
of optimized bindings [here](https://doc.qt.io/qt-6/qtquick-performance.html#property-bindings).
If you need to disable or enable bindings, prefer using 
[Binding objects](#use-binding-object). Or use a boolean flag
to enable a binding, e.g `visible: privates.bindingEnabled ? root.count > 0 : false`.

## Avoid Unnecessary Re-Evaluations

If you have a loop or process where you update the value of the property, you may
want to use a temporary local variable where you accumulate those changes and only
report the last value to the property. This way you can avoid triggering re-evaluation
of binding expressions during the intermediate stages of accumulation.

Here's a bad example straight from Qt documentation:

```qml
import QtQuick

Item {
    id: root
    width: 200
    height: 200

    property int accumulatedValue: 0

    Component.onCompleted: {
        const someData = [ 1, 2, 3, 4, 5, 20 ];
        for (let i = 0; i < someData.length; ++i) {
            accumulatedValue = accumulatedValue + someData[i];
        }
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```

And here is the proper way of doing it:

```qml
import QtQuick

Item {
    id: root
    width: 200
    height: 200

    property int accumulatedValue: 0

    Component.onCompleted: {
        const someData = [ 1, 2, 3, 4, 5, 20 ];
        let temp = accumulatedValue;
        for (let i = 0; i < someData.length; ++i) {
            temp = temp + someData[i];
        }

        accumulatedValue = temp;
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```

Also note that `list` type doesn't have a change signal associated with adding/moving/removing
elements from it. If you are using a `list` type to store sequential data, make sure that the
places where this property is used does not do expensive things (e.g populating a view).

# C++ Integration

QML can be extended with C++ by exposing the `QObject` classes using the `Q_OBJECT`
macro or custom data types using `Q_GADGET` macro.
It always should be preferred to use C++ to add functionality to a QML application.
But it is important to know which is the best way to expose your C++ classes, and
it depends on your use case.

## Avoid Context Properties

Context properties are registered using

```cpp
rootContext()->setContextProperty("someProperty", QVariant());
```

Context properties always takes in a `QVariant`, which means that whenever you access the property
it is re-evaluated because in between each access the property may be changed as
`setContextProperty()` can be used at any moment in time.

Context properties are expensive to access, and hard to reason with. When you are writing QML code,
you should strive to reduce the use of contextual variables (A variable that doesn't exist in the
immediate scope, but the one above it.) and global state. Each QML document should be able to run
with QML scene provided that the required properties are set.

See [QTBUG-73064](https://bugreports.qt.io/browse/QTBUG-73064).

## Use Singleton for Common API Access

There are bound to be cases where you have to provide a single instance for a
functionality or common data access. In this situation, resort to using a singleton
as it will have a better performance and be easier to read. Singletons are also
a good option to expose enums to QML.

```cpp
class MySingletonClass : public QObject
{
public:
    static QObject *singletonProvider(QQmlEngine *qmlEngine, QJSEngine *jsEngine)
    {
        if (m_Instance == nullptr) {
            m_Instance = new MySingletonClass(qmlEngine);
        }

        Q_UNUSED(jsEngine);
        return m_Instance;
    }
};

// In main.cpp
qmlRegisterSingletonType<SingletonTest>("MyNameSpace", 1, 0, "MySingletonClass",
                                        MySingletonClass::singletonProvider);
```

You should strive to not use singletons for shared data access. Reusable components are especially
a bad place to access singletons. Ideally, all QML documents should rely on the customization
through properties to change its content.

Let's imagine a scenario where we are creating a paint app where we can change the currently
selected color on the palette. We only have one instance of the palette, and the data from this is
accessed throughout our C++ code. So we decided that it makes sense to expose it as a singleton to
QML side.

```qml
// ColorViewer.qml
Row {
    id: root

    Rectangle {
        color: Palette.selectedColor
    }

    Text {
        text: Palette.selectedColorName
    }
}
```

With this code, we bind our component to `Palette` singleton. Who ever wants to use our `ColorViewer`
they won't be able to change it so they can show some other selected color.

```qml
// ColorViewer_2.qml
Row {
    id: root

    property alias selectedColor: colorIndicator.color
    property alias selectedColorName: colorLabel.color

    Rectangle {
        id: colorIndicator
        color: Palette.selectedColor
    }

    Text {
        id: colorLabel
        text: Palette.selectedColorName
    }
}
```

This would allow the users of this component to set the color and the name from outside, but we
still have a dependency on the singleton.

```qml
// ColorViewer_3.qml
Row {
    id: root

    property alias selectedColor: colorIndicator.color
    property alias selectedColorName: colorLabel.color

    Rectangle {
        id: colorIndicator
    }

    Text {
        id: colorLabel
    }
}
```

This version allows you to de-couple from the singleton, enable it to be resuable in any context
that wants to show a selected color, and you could easily run this through `qmlscene` and inspect
its behavior.

## Prefer Instantiated Types Over Singletons For Data

Instantiated types are exposed to QML using:

```cpp
// In main.cpp
qmlRegisterType<ColorModel>("MyNameSpace", 1, 0, "ColorModel");
```

Instantiated types have the benefit of having everything available to you to understand and digest
in the same document. They are easier to change at run-time without creating side effects, and easy
to reason with because when looking at a document, you don't need to worry about any global state
but the state of the type that you are dealing with at hand.

```qml
// ColorsWindow.qml
Window {
    id: root

    Column {
        Repeater {
            model: Palette.selectedColors
            
            delegate: ColorViewer {
                selectedColor: color
                selectedColorName: colorName

                required property color color
                required property string colorName
            }
        }
    }
}
```

The code above is a perfectly valid QML code. We'll get our model from the singleton, and display it
with the reusable component we created in earlier. However, there's still a problem here. `ColorsWindow`
is now bound to the model from `Palette` singleton. And If I wanted to have the user select two
different sets of colors, I would need to create another file with the same contents and use that.
Now we have 2 components doing basically the same thing. And those two components need to be
maintained.

This also makes it hard to prototype. If I wanted to see two different versions of this window with
different colors at the same time, I can't do it because I'm using a singleton. Or, If I wanted to
pop up a new window that shows the users the variants of a color set, I can't do it because the data
is bound to the singleton.

A better approach here is to either use an instantiated type or expect the model as a property.

```qml
// ColorsWindow.qml
Window {
    id: root

    property PaletteColorsModel model

    Column {
        Repeater {
            model: root.model
            // Alternatively
            model: PaletteColorsModel { }
            
            delegate: ColorViewer {
                selectedColor: color
                selectedColorName: colorName

                required property color color
                required property string colorName
            }
        }
    }
}
```

Now, I can have the same window up at the same time with different color sets because they are not
bound to a singleton. During prototyping, I can provide a dummy data easily by adding
`PaletteColorElement` types to the model, or by requesting test dataset with something like:

```qml
PaletteColorsModel {
    testData: "prototype_1"
}
```

This test data could be auto-generated, or it could be provided by a JSON file. The beauty is that
I'm no longer bound to a singleton, that I have the freedom to instantiate as many of these windows
as I want.

There may be cases where you actually truly want the data to be the same every where. In these
cases, you should still provide an instantiated type instead of a singleton. You can still access
the same resource in the C++ implementation of your model and provide that to QML. And you would
still retain the freedom of making your data easily pluggable in different context and it would
increase the re-usability of your code.

```cpp
class PaletteColorsModel
{
    explicit PaletteColorsModel(QObject* parent = nullptr)
    {
        initializeModel(MyColorPaletteSingleton::instance().selectedColors());
    }
};
```

## Watch Out for Object Ownership Rules

When you are exposing data to QML from C++, you are likely to pass around custom
data types as well. It is important to realize the implications of ownership when
you are passing data to QML. Otherwise you might end up scratching your head trying
to figure out why your app crashes.

If you are exposing custom data type, prefer to set the parent of that data to the
C++ class that transmits it to QML. This way, when the C++ class gets destroyed
the custom data type also gets destroyed and you won't have to worry about releasing
memory manually.

There might also be cases where you expose data from a singleton class without a
parent and the data gets destroyed because QML object that receives it will take
ownership and destroy it. And you will end up accessing data that doesn't exist.
Ownership is **not** transferred as the result of a property access. For data
ownership rules see [here](https://doc.qt.io/qt-6/qtqml-cppintegration-data.html#data-ownership).

To learn more about the real life implications of this read [this blog post](https://www.embeddeduse.com/2018/04/02/qml-engine-deletes-c-objects-still-in-use/).

# Performance and Memory

Most applications are not likely to have memory limitations. But in case you are
working on a memory limited hardware or you just really care about memory allocations,
follow these steps to reduce your memory usage.

## Reduce the Number of Implicit Types

If a type defines custom properties, that type becomes an implicit type to the JS
engine and additional type information has to be stored.

```qml
Rectangle { } // Explicit type because it doesn't contain any custom properties

Rectangle {
    // The deceleration of this property makes this Rectangle an implicit type.
    property int meaningOfLife: 42
}
```

You should follow the advice from the [official documentation](http://doc.qt.io/qt-6/qtquick-performance.html#avoid-defining-multiple-identical-implicit-types)
and split the type into its own component If it's used in more than one place.
But sometimes, that might not make sense for your case. If you are using a lot of
custom properties in your QML file, consider wrapping the custom properties of
types in a `QtObject`. Obviously, JS engine will still need to allocate memory
for those types, but you already gain the memory efficiency by avoiding the
implicit types. Additionally, wrapping the properties in a `QtObject` uses less
memory than scattering those properties to different types.

Consider the following example:

```qml
Window {
    Rectangle { id: r1 } // Explicit type. Memory 64b, 1 allocation.

    // Implicit type. Memory 128b, 3 allocations.
    Rectangle { id: r2; property string nameTwo: "" }

    QtObject { // Implicit type. Memory 128b, 3 allocations.
        id: privates
        property string name: ""
    }
}
```

In this example, the introduction of a custom property to added additional 64b
of memory and 2 more allocations. Along with `privates`, memory usage adds up to
256b. The total memory usage is 320b.

You can use the QML profiler to see the allocations and memory usage for each
type. If we change that example to the following, you'll see that both memory
usage and number of allocations are reduced.

```qml
Window {
    Rectangle { id: r1 } // Explicit type. Memory 64b, 1 allocation.

    Rectangle { id: r2 } // Explicit type. Memory 64b, 1 allocation.

    QtObject { // Implicit type. Memory 160b, 4 allocations.
        id: privates

        property string name: ""
        property string nameTwo: ""
    }
}
```

In the second example, total memory usage is 288b. This is really a minute
difference in this context, but as the number of components increase in a
project with memory constrained hardware, it can start to make a difference.

# Signal Handling

Signals are a very powerful mechanism in Qt/QML. And the fact that you can
connect to signals from C++ makes it even better. But in some situations, If you
don't handle them correctly you might end up scratching your head.

## Try to Avoid Using connect Function in Models

You can have signals in the QML side, and the C++ side. Here's an example for
both cases.

QML Example.

```qml
// MyButton.qml
import QtQuick.Controls

Button {
    id: root

    signal rightClicked()
}
```

C++ Example:

```cpp
class MyButton
{
    Q_OBJECT

signals:
    void rightClicked();
};
```

The way you connect to signals is using the syntax

```qml
item.somethingChanged.connect(function() {})
```

When this method is used, you create a function that is connected to the
`somethingChanged` signal.

Consider the following example:

```qml
// MyItem.qml
Item {
    id: root

    property QtObject customObject

    objectName: "my_item_is_alive"
    onCustomObjectChanged: {
        customObject.somethingChanged.connect(() => {
            console.log(root.objectName)
        })
    }
}
```

This is a perfectly legal code. And it would most likely work in most scenarios.
But, if the life time of the `customObject` is not managed in `MyItem`, meaning
if the `customObject` can keep on living when the `MyItem` instance is destroyed,
you run into problems.

The connection is created in the context of `MyItem`, and the function naturally
has access to its enclosing context. So, as long as we have the instance of
`MyItem`, whenever `somethingChanged` is emitted we'd get a log saying
`my_item_is_alive`.

Here's a quote directly from [Qt documentation](https://doc.qt.io/qt-6/qml-qtquick-listview.html):

> Delegates are instantiated as needed and may be destroyed at any time. They
> are parented to `ListView`'s `contentItem`, not to the view itself. State
> should never be stored in a delegate.

So you might be making use of an external object to store state. But what If
`MyItem` is used in a `ListView`, and it went out of view and it was destroyed
by `ListView`?

Let's examine what happens with a more concrete example.

```qml
ApplicationWindow {
    id: root
    width: 640
    height: 480

    property list<QtObject> myObjects: [
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        }
    ]

    ListView {
        anchors {
            top: parent.top
            left: parent.left
            right: parent.right
            bottom: btn.top
        }
        // Low enough we can resize the window to destroy buttons.
        cacheBuffer: 1
        model: root.myObjects.length
        delegate: Button {
            id: self
            text: "Button " + index
  
            readonly property string name: "Button #" + index

            onClicked: {
                root.myObjects[index].somethingHappened()
            }
            Component.onCompleted: {
                root.myObjects[index].somethingHappened.connect(() => {
                    // When the button is destroyed, this will cause the following
                    // error: TypeError: Type error
                    console.log(self.name)
                })
            }
            Component.onDestruction: {
                console.log("Destroyed #", index)
            }
        }
    }

    Button {
        id: btn
        anchors {
            bottom: parent.bottom
            horizontalCenter: parent.horizontalCenter
        }
        text: "Emit Last Signal"
        
        onClicked: root.myObjects[root.myObjects.length - 1].somethingHappened()
    }
}
```

In this example, once one of the buttons is destroyed we still have the object
instance. And then object instance still contains the connection we made in
`Component.onCompleted`. So, when we click on `btn`, we get an error:
`TypeError: Type error`. But once we expand the window so that the button is
created again, we don't get that error. That is, we don't get that error for the
newly created button. But the previous connection still exists and still causes
error. But now that a new one is created, we end up with two connections on the
same object.

This is obviously not ideal and should be avoided. But how do you do it?

The simplest and most elegant solution (That I have found) is to simply use a
`Connections` object and handle the signal there. So, If we change the code to
this:

```qml
delegate: Button {
    id: self
    text: "Button " + index

    readonly property string name: "Button #" + index

    onClicked: root.myObjects[index].somethingHappened()

    Connections {
        target: root.myObjects[index]
        function onSomethingHappened {
            console.log(self.name)
        }
    }
}
```

Now, whenever the delegate is destroyed so is the connection. This method can
be used even for multiple objects. You can simply put the `Connections` in a
`Component` and use `createObject` to instantiate it for a specific object.

```qml
Item {
    id: root
    onObjectAdded: {
        cmp.createObject(root, {"target": newObject})
    }

    Component {
        id: cmp

        Connections {
            target: root.myObjects[index]
            function onSomethingHappened() {
                console.log(self.name)
            }
        }
    }
}
```

## When to use Functions and Signals

When coming from imperative programming, it might be very tempting to use signals
very similar to functions. Resist this temptation. Especially when communicating
between the C++ layer of your application, misusing signals can be very confusing
down the line.

Let's first clearly define what a signal should be doing. Here's how
[Qt](https://doc.qt.io/qt-6/signalsandslots.html#signals) defines it.

> Signals are emitted by an object when its internal state has changed in some
> way that might be interesting to the object's client or owner. 

This means that whatever happens in the signal handler is a reaction to an
internal state change of an object. The signal handler should not be changing
something else in the same object.

See the following example. We have a `ColorPicker` component that we want to use
to show the user a message when the color is picked. As far as component design
goes, the fact that the customer sees a message is not `ColorPicker`'s job.
Its job is to present a dialog and change the color it represents.

```qml
// ColorPicker.qml
Rectangle {
    id: root

    signal colorPicked()

    ColorDialog {
        onColorChanged: {
            root.color = color
            root.colorPicked()
        }
    }
}

// main.qml
Window {
    ColorPicker {
        onColorPicked: {
            label.text = "Color Changed"
        }
    }

    Label {
        id: label
    }
}
```

The above example is pretty straightforward, the signal handler only reacts to
a change and does something with that information after which the `ColorPicker`
object is not affected.

```qml
// ColorPicker.qml
Rectangle {
    id: root

    signal colorPicked(color pickedColor)

    ColorDialog {
        onColorChanged: {
            root.colorPicked(color)
        }
    }
}

// main.qml
Window {
    ColorPicker {
        onColorPicked: {
            color = pickedColor
            label.text = "Color Changed"
        }
    }

    Label {
        id: label
    }
}
```

In this example, the signal handler not only reacts to an internal state but it
also changes it. This is a very simple example, and it'll be easy to spot an
error. However complex your application is, you will always benefit from
making the distinction clear. Otherwise what you think to be a function at first
glance might end up being a signal and it loses its semantics of an internal
state change.

Here's a general principle to follow:

1. When communicating up, use signals.
2. When communicating down, use functions.

### Communicating with C++ Using Signals

When you have a model that you use in the QML side, it's very possible that you
are going to run into cases where something that happens in the QML side needs
to trigger an action in the C++ side.

In these cases, prefer not to invoke any C++ signals from QML side. Instead,
use a function call or better a property assignment. The C++ object then should
make the decision whether to fire a signal or not.

If you are using a C++ type instantiated in QML, the same rules apply. You should
not be emitting signals from QML side.

# JavaScript

It is the prevalent advice that you should avoid using JavaScript as much as possible
in your QML code and have the C++ side handle all the logic. This is a sound advice
and should be followed, but there are cases where you can't avoid having JavaScript
code for your UI. In those cases, follow these guidelines to ensure a good use of
JavaScript in QML.

## Use Arrow Functions

Arrow functions were introduced in ES6. Its syntax is pretty close to C++ lambdas
and they have a pretty neat feature that makes them most comfortable to use
when you are using the `connect()` function to create a binding. If there's no
block within the arrow function, it has an implicit return statement.

Let's compare the arrow function version with the old way.

```qml
Item {
    property int value: -1

    Component.onCompelted: {
        // Arrow function
        root.value = Qt.binding(() => root.someOtherValue)
        // The old way.
        root.value = Qt.binding(function() { return root.someOtherValue })
    }
}
```

The arrow function version is easier on the eyes and cleaner to write.
For more information about arrow functions, head over to the [MDN Blog](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## Use the Modern Way of Declaring Variables

With ES6, there are 3 ways of delcaring a variable: `var`, `let`, and `const`.

You should leverage `let` and `const` in your codebase and avoid using `var`.
`let` and `const` enables a scope based naming wheras `var` only knows about one
scope.

```qml
Item {
    onClicked: {
        const value = 32;
        let valueTwo = 42;
        {
            // Valid assignment since we are in a different scope.
            const value = 32;
            let valueTwo = 42;
        }
    }
}
```

Much like in C++, prefer using `const` If you don't want the variable to be assigned.
But keep in mind that `const` variables in JavaScript are not immutable. It just
means they can't be reassigned, but their contents can be changed.

```javascript
const value = 32;
value = 42; // ERROR!

const obj = {value: 32};
obj.value = 42; // Valid.
```

See the MDN posts on [const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
and [let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)

# States and Transitions

States and transitions are a powerful way to create dynamic UIs. Here are some things to keep in
mind when you are using them in your projects.

## Don't Define Top Level States

Defining states at the top-level of a reusable component can cause breakages if the user of your
components also define their own states for their specific use case. 

```qml
// MyButton.qml
Rectangle {
    id: root
    color: "red"

    property alias text: lb.text
    property alias hovered: ma.containsMouse

    MouseArea {
        id: ma
        anchors.fill: parent
        hoverEnabled: true
    }

    Label {
        id: lb
        anchors.centerIn: parent
    }
    
    states: [
        State {
            when: ma.containsMouse

            PropertyChanges {
                target: root
                color: "yellow"
            }
        }
    ]
}

// MyItem.qml
Item {
    MyButton {
        id: btn
        text: "Not Hovering"
        // The states of the original component are not actually overwritten.
        // The new state is added to the existing states.
        states: [
            State {
                when: btn.hovered

                PropertyChanges {
                    target: btn
                    text: "Hovering"
                }
            }
        ]
    }
}
```
When you assign a new value to `states` or any other `QQmlListProperty`, the new value does not
overwrite the existing one but adds to it. In the example above, the new state is added to the
existing list of states that we already have in `MyButton.qml`. Since we can only have one active
state in an item, our hover state will be messed up.

In order to avoid this problem, create your top-level state in a separate item or use a
`StateGroup`.

```qml
Rectangle {
    id: root
    color: "red"

    property alias text: lb.text
    property alias hovered: ma.containsMouse


    MouseArea {
        id: ma
        anchors.fill: parent
        hoverEnabled: true
    }

    Label {
        id: lb
        anchors.centerIn: parent
    }

    // another item
    Item {
        states: [
            State {
                when: ma.containsMouse

                PropertyChanges {
                    target: root
                    color: "yellow"
                }
            }
        ]
    }

    // A State group or
    StateGroup {
        states: [
            State {
                when: ma.containsMouse

                PropertyChanges {
                    target: root
                    color: "yellow"
                }
            }
        ]
    }

}
```

With this change, the button will both change its color and text when the mouse is hovered above it.


# Visual Items

Visual items are at the core of QML, anything that you see in the window (or don't see because of
transparency) are visual items. Having a good understanding of the visual items, their relationship
to each other, sizing, and positioning will help you create a more robust UI for your application.

## Distinguish Between Different Types of Sizes

When thinking about geometry, we think in terms of `x`, `y`, `width` and `height`. This defines
where our items shows up in the scene and how big it is. `x` and `y` are pretty straightforward but
we can't really say the same about the size information in QML.

There's 2 different types of size information that you get from various visual items:

1. Explicit size: `width`, `height`
2. Implicit size: `implicitWidth`, `implicitHeight`

A good understanding of these different types is important to building a reusable library of
components.

### Explicit Size

It's in the name. This is the size that you explicitly assign to an `Item`. By default, `Item`s do
not have an explicit size and its size will always be `Qt.size(0, 0)`.

```qml
// No explicit size is set. You won't see this in your window.
Rectangle {
    color: "red"
}

// Explicit size is set. You'll see a yellow rectangle.
Rectangle {
    width: 100
    height: 100
    color: "yellow"
}
```

### Implicit Size

Implicit size refers to the size that an `Item` occupies by default to display itself properly.
This size is not set automatically for any `Item`. You, as a component designer, need to make a
decision about this size and set it to your component.

The other thing to note is that [Qt internally
knows](https://github.com/qt/qtdeclarative/blob/dev/src/quick/items/qquickitem.h#L418) if it has an
explicit size or not. So, when an explicit size is not set, it will use the implicit size.

```qml
// Even though there's no explicit size, it will have a size of Qt.size(100, 100)
Rectangle {
    implicitWidth: 100
    implicitHeight: 100
    color: "red"
}
```

-----

Whenever you are building a reusable component, never set an explicit size within the component but
instead choose to provide a sensible implicit size. This way, the user of your components can freely
manipulate its size and when they need to return to a default size, they can always default to the
implicit size so they don't have to store a different default size for the component. This feature
is also very useful if you want to implement a resize-to-fit feature.

When a user is using your component, they may not bother to set a size for it.

```qml
CheckBox {
    text: "Check Me Out"
}
```

In the example above, the check box would only be visible If there was a sensible implicit size for
it. This implicit size needs to take into account its visual components (the box, the label etc.) so
that we can see the component properly. If this is not provided, it's difficult for the user of your
component to set a proper size for it.

## Be Careful with a Transparent `Rectangle`

`Rectangle` should never be used with a transparent color except when you need to draw a border.
This is especially true if you are using a `Rectangle` as part of a delegate that's supposed to be
created in a batch.

Drawing transparent/translucent content takes more time because translucency requires blending.
Opaque content is optimized better by the renderer.

In order to avoid paying the penalty, look for ways that you can defer the use of a transparent
`Rectangle`. Maybe you can show it on hover, or during certain events and set it to invisible when
it's no longer needed. Alternatively, you can put the `Rectangles` in an asynchronous `Loader`.

Here's a sample QML code to demonstrate the difference between using an opaque rectangle and a
transparent one when it comes to the creation time of these components.

```qml
Window {
    visible: true

    Row {
        Button {
            text: "Rect"
            onClicked: {
                console.time("Rect")
                rprect.model = rprect.model + 10000
                console.timeEnd("Rect")
            }
        }

        Button {
            text: "Transparent"
            onClicked: {
                console.time("Transparent")
                rptrans.model = rptrans.model + 10000
                console.timeEnd("Transparent")
            }
        }

        Button {
            text: "Transparent Loader"
            onClicked: {
                console.time("Transparent Loader")
                rploader.model = rploader.model + 10000
                console.timeEnd("Transparent Loader")
            }
        }

        Button {
            text: "Translucent"
            onClicked: {
                console.time("Translucent")
                rptransl.model = rptransl.model + 10000
                console.timeEnd("Translucent")
            }
        }

        Button {
            text: "Reset"
            onClicked: {
                rprect.model = 0
                rptrans.model = 0
                rptransl.model = 0
                rploader.model = 0
            }
        }
    }

    Repeater {
        id: rptrans
        model: 0
        delegate: Rectangle {
            width: 10
            height: 10
            color: "transparent"
        }
    }

    Repeater {
        id: rptransl
        model: 0
        delegate: Rectangle {
            width: 10
            height: 10
            opacity: 0.5
            color: "red"
        }
    }

    Repeater {
        id: rprect
        model: 0
        delegate: Rectangle {
            width: 10
            height: 10
            color: "red"
        }
    }

    Repeater {
        id: rploader
        model: 0
        // This will speed things up. You can defer the creation of the rectangle to when it makes
        // sense and since it's asynchronous it won't block the UI thread.
        delegate: Loader {
            asynchronous: true
            sourceComponent: Rectangle {
                width: 10
                height: 10
                opacity: 0.5
                color: "transparent"
            }
        }
    }
}
```

When you run this example for the first time and create solid rectangles, you'll notice that the
creation is pretty fast. If you close it and run it again, but this time create transparent or
translucent ones you'll see that the time reported does not actually differ that much from the
solid rectangle.

The real problem starts presenting itself when you are creating new transparent items when there's
already rectangles on the scene. Try creating first the solid ones and then the transparent ones.
You'll see that the time difference is very noticeable.

Please note that this will not matter that much when you are drawing a few rectangles here and
there. The problem will present itself when you are using translucency in the context of a delegate
because there can potentially be creating thousands of these rectangles.

See also: [Translucent vs Opaque](https://doc.qt.io/qt-6/qtquick-performance.html#translucent-vs-opaque)
