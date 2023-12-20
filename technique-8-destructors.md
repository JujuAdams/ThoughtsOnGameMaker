# GameMaker Technique 8 - Destructors

&nbsp;

Ok, these aren't really destructors. Instead, we can simulator destructor-like behaviour by wrapping one struct in another and using weak references in a creative way. This technique isn't especially performant and has some shortcomings but may be of interest regardless.

The goal with this technique is to allow us to detect when all references to a struct has been lost and, when that happens, execute some "last breath" code before that struct is gone for good. This is handy in situations where you need to clean up some data when something gets destroyed but you don't want to manually call a destroy function. As an example, you may have a struct that creates and holds a reference to a surface. When that struct is garbage collected, you want that surface to be cleaned up otherwise you have a memeory leak.

We can achieve this behaviour by splitting up a struct into two parts. The first part acts an interface and a way to detect if all references to that interface have been lost. The second part holds state and all other functional code. When executing a method on the interface, the actual function method call is passed through to the underlying functional struct. This means there's an extra level of indirection when calling methods and accessing variables which can be inconvenient and adds extra overhead. This is the price we pay!

The final piece of the puzzle

```gml

function Struct() constructor
{
    state = new __StructInner(self);
    
    static SomeMethod = function(_value)
    {
        return state.SomeMethod(_value);
    }
}

global.trackingArray = [];

function __StructInner(_wrapperStruct) constructor
{
    __wrapperStruct = weak_ref_create(_wrapperStruct);
    array_push(global.trackingArray, self);
    
    static SomeMethod = function(_value)
    {
        show_debug_message("SomeMethod = " + string(_value));
    }
    
    static __Destroy = function()
    {
        show_debug_message("Destroyed");
    }
}

time_source_start(time_source_create(time_source_global, 1, time_source_units_frames, function()
{
    var _i = 0;
    repeat(array_length(global.trackingArray))
    {
        if (not weak_ref_alive(global.trackingArray[_i].__wrapperStruct))
        {
            array_delete(global.trackingArray, _i, 1);
            global.trackingArray[_i].__Destroy();
        }
        else
        {
            ++_i;
        }
    }
}, [], -1));
```
