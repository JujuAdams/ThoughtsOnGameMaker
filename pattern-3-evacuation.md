# GameMaker Design Pattern 3: Evacuating Global Scope

&nbsp;

Whilst you might not someone writing a library for public consumption, it's a good habit to get into to view independent sections of your code as libraries. You might want to call them "modules" or "components" or "subsystem" or just "APIs" but, at any rate, keeping your code cleanly divided makes maintenance and teamwork much much easier. There has been endless ink spilt on the topic of code encapsulation and I wouldn't pretend to have anything to add to the discussion other than "encapsulation is good". You should be encapsulating your code and spending the time to build independently operable libraries.

When writing libraries, it's often useful to hold information in global scope for convenience. It is, from time to time, also a necessity. However, this has negative side effects: Because a game and any libraries that game is using have to share the same global scope, it's easy to jumble up variables from different places. A variable called `global.cameraX` could in principle be in use by any library or come from game code itself. In the worst case, critical global variables could be accidentally overwritten by other bits of code, especially if those global variables have a commonly used name (such as `global.debugMode`). Additionally, having a bunch of variables in global scope, even if intelligently and clearly named, clogs up GameMaker's debug view. This can make it very hard for someone to find the global variables they're interested in. It's important to consider the user experience of using a library from multiple perspectives and ensuring a library doesn't make the user experience worse when debugging is part of that.

Fortunately, we can maintain variables in global scope without using global variables. This is done by creating a `static` reference to a struct returned from a globally scoped function and then storing variables inside that struct. Here's an example:

```gml
//This function returns a struct that the function itself holds statically
//This means that every call to __ExampleGlobal() will return the same struct
function __ExampleGlobal()
{
    static _global = {
        __variable: undefined,
    };
    return _global;
}

//This function sets a value in the static struct
function ExampleSet(_value)
{
    static _global = __ExampleGlobal();
    _global.__variable = _value;
}

//This function returns the value from the static struct
function ExampleGet()
{
    static _global = __ExampleGlobal();
    return _global.__variable;
}
```

This approach can be extended to include an initialization flow as well. There's a particular advantage to this method because the initialization process happen "just in time", that is to say that the initialization process happens no matter which endpoint function (`ExampleSet()` or `ExampleGet()`) is called first. This reduces the need for explicit initialization calls which is often convenient.

```gml
function __ExampleGlobal()
{
    //Instead of initializing the static to a struct, initialize to some invalid value
    static _global = undefined;

    //If our return value is valid then return it
    //This means that the first time this function is run the subsequent code is executed
    if (_global != undefined) return _global;

    //This code will only run once, and only when it needs to
    show_debug_message("The example code initialized");
    
    _global = {};
    _global.__variable = undefined;

    //Make sure you return a proper value!
    return _global;
}
```

You can also use this `static` trick in constructors too if need be, such that the "global" variables are accessible from the scope of any instance of the constructor. As a final note, you may still want to access a library's global variables when debugging for your own purposes. The easiest way to do this is to simply make a global variable that points at the static struct e.g.

```gml
function __ExampleGlobal()
{
    static _global = undefined;
    if (_global != undefined) return _global;

    show_debug_message("The example code initialized");
    
    _global = {};

    //Allow access to the static struct from the debugger
    if (debug_mode) global.exampleLibrary = _global;

    _global.__variable = undefined;

    return _global;
}
```
