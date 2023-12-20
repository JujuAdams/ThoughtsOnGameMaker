# GameMaker Technique 8 - Destructors

&nbsp;

Ok, these aren't really destructors. I clickbaited you. Instead, we can simulate destructor-like behaviour by wrapping one struct in another and using weak references in a creative way. This technique isn't especially performant and has some shortcomings but may be of interest regardless.

The goal with this technique is to allow us to detect when all references to a struct has been lost and, when that happens, execute some "last breath" code before that struct is gone for good. This is handy in situations where you need to clean up some data when something gets destroyed but you don't want to manually call a destroy function. As an example, you may have a struct that creates and holds a reference to a surface. When that struct is garbage collected, you want that surface to be cleaned up otherwise you have a memeory leak.

We can achieve this behaviour by splitting up a struct into two parts. The first part acts an interface and a way to detect if all references to that interface have been lost. The second part holds state and all other functional code. When executing a method on the interface, the actual function method call is passed through to the underlying functional struct. This means there's an extra level of indirection when calling methods and accessing variables which can be inconvenient and adds extra overhead. This is the price we pay!

The missing piece of the puzzle is how we check for when a destructor should be called. The inner functional struct should hold a weak reference to the wrapper. This means the functional struct can determine if all references to the wrapper have been lost and, if so, execute its destructor. We'll need to iterate over every functional struct to detect if wrapper structs have been lost which means (at least in the simplest implementation) iterating over a global array. You can adjust how to detect lost wrappers but the implementation below does a full sweep every frame.

Before showing an example, there's something that needs to be pointed out here: this system is only as good as GameMaker's garbage collector. If the garbage collector hasn't collected a wrapper struct then its weak reference will still appear to be alive and a destructor will not be called. This can potentially cause problems if you're relying on destructors to promptly clean up large blocks of memory (buffers, surfaces, vertex buffers etc.). Diligent use of the native `gc_collect()` may be required to reliably safeguard yourself against out-of-memory issues.

```gml

//Wrapper struct
function Struct() constructor
{
    //The wrapper instantiates an inner state-tracking struct
    state = new __StructInner(self);
    
    //We DON'T track state in the wrapper struct
    
    //Method calls need to be redirected to the state struct
    static SomeMethod = function(_value)
    {
        return state.SomeMethod(_value);
    }
}

//State-tracking struct. Should never be accessed directly!
function __StructInner(_wrapperStruct) constructor
{
    //Create a weak reference to the wrapper
    //We use this weak reference to determine when the destructor should be called
    __wrapperStruct = weak_ref_create(_wrapperStruct);
    
    //Push this struct to the tracking array (as a strong reference)
    array_push(global.trackingArray, self);
    
    //Endpoint for redirected method call from wrapper
    static SomeMethod = function(_value)
    {
        show_debug_message("SomeMethod = " + string(_value));
    }
    
    //Method to call when this state struct bites the dust
    static __Destructor = function()
    {
        show_debug_message("__Destructor");
    }
}

//Create an array to track inner state structs
global.trackingArray = [];

//Run a time source once a frame to evaluate and execute destructors
time_source_start(time_source_create(time_source_global, 1, time_source_units_frames, function()
{
    var _i = 0;
    repeat(array_length(global.trackingArray))
    {
        //If the wrapper struct for this state struct is dead ...
        if (not weak_ref_alive(global.trackingArray[_i].__wrapperStruct))
        {
            //... remove this struct from the tracking array and call its destructor method
            array_delete(global.trackingArray, _i, 1);
            global.trackingArray[_i].__Destructor();
        }
        else
        {
            ++_i;
        }
    }
}, [], -1));
```
