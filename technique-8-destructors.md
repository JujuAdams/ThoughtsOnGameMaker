# GameMaker Technique 8 - Destructors

I was unsatisfied by the over-complex solution for [destructors](technique-8-destructors-legacy.md) in GameMaker. I had an opportunity recently in commercial work to revisit the topic and I'm happy to say I found a solution that I'm more comfortable with. I am confident is more easily applied in practice.

For the uninitiated, a "destructor" is a callback that is executed when a struct is garbage collected. We cannot build a true destructor in GML as it not a feature of the language. Instead, we need to build systems that emulate destructors. We can write this emulation by checking when a struct has been destroyed by the garbage collector and then execute a callback in response to this. We do this with "weak references". We use `weak_ref_create()` to create a weak reference for a target struct (or instance). `weak_ref_alive()` should then be called frequently to determine if the target has been unalived by the garbage collector.

Whilst the principle is straight-forward, implementation questions remain. One question is how to continuously check that weak reference without burdening the game with lots of unnecessary repeated checks. Firstly, checking every destructor every frame seems excessive. We should check every once in a while, perhaps once a second. Secondly, if you've got a couple dozen destructors then checking them every frame is unlikely to cause trouble but larger games with perhaps thousands of destructors will encounter performance problems eventually.

The solution for this first question is to lean into GameMaker's native "time source" feature. This feature allow us to run some code every now and again without needing to manage the timer or the iteration process ourselves. This is exactly what we want. We can create one time source per destructor and, in that time source, check the weak reference to the target. Why this didn't occur to me first time round I don't know but here we are. One thing to consider is what happens if we make lots of destructors all in the same step. Without adjustment, all of those destructors will execute their checking code at exactly the same time in the future. The simplest way to work around this is to spread out the time sources such that they start at different times. Even if the time sources all have the same period, the load they cause will be spread out.

The second question, one that is obvious by reading my previous attempt, is how to make it clear which structs owns what data. Once the target struct has been garbage collected, the destructor needs to know what resources it can access. The destructor cannot access the target struct directly as doing so would prevent the target from being garbage collected to begin with. Tracking where data is stored is hard enough but adding layers of indirection and making use of the kind of "clever" tricks I have tried in the past makes understanding code all that much harder.

The second question requires a compromise. The destructor has to contain the necessary and sufficient data to operate. When the target (or indeed any other part of the codebase) needs to access this data then it must **always** go through the destructor and should **never** create a permanent reference to any data that the destructor holds onto. This goes against the grain of much established GameMaker practice where simplicity and flat structures are preferred. Storing data within nested structures makes code harder to read, harder to debug, and (in GameMaker) slower to run. Nevertheless, we need to separate our data such that information essential for the destructor is stored in the destructor and only the destructor, and other data is stored in the target struct.

Here's what the `Destructor()` constructor function looks like:

```gml
/// Constructor. Creates a struct that can be used to execute code when a target (either a struct
/// or an instance) falls out of scope and is garbage collected. A method stored in the `.callback`
/// variable will be executed when the host is garbage collected. The destructor struct itself
/// should contain references to any resources that need to be destroyed/freed. The destructor must
/// not contain any references to the target otherwise the target will never be garbage collected.
/// 
/// If you need to trigger the destructor immediately then you may call the `.Terminate()` method.
/// `.Terminate()` can only be called once and subsequent calls will do nothing.
/// 
/// N.B. This function calls `random()`.
/// 
/// @param [period=1sec]
/// @param [target=self]

function Destructor(_period = 1, _target = other) constructor
{
    //Empty callback. This variable should be overwritten when building the constructor
    callback = undefined

    //Weak reference to check whether our host has been garbage collected
    __targetRef = weak_ref_create(_target);
    
    //Public method that allows for the destructor to be triggered immediately
    static Terminate = function()
    {
        time_source_destroy(__timeSource);
        
        //If the callback is valid, execute it, and then ensure the callback cannot be executed again
        if (is_callable(callback))
        {
            callback();
            callback = undefined;
        }
    }

    //Main update function. This will be called once every period (which is once every second by default)
    __timeSource = time_source_create(time_source_global, _period, time_source_units_seconds, function()
    {
        //If our host has been garbage collected then terminate the destructor
        if (not weak_ref_alive(__targetRef))
        {
            Terminate();
        }
    },
    [], -1);

    //Start the update function some random time later. This spreads out time sources so that they don't
    //stack up on one particular step if multiple destructors are created at the same time
    call_later(random(_period), time_source_units_seconds, function() { time_source_start(__timeSource) }, false);
}
```

And here's an example of use showing how the `Destructor` constructor can be used to safely manage a surface. Note that the target (the struct built by `SafeSurface()`) holds a reference to the destructor but not the other way round. It's not essential that the target holds a reference to the destructor but it is going to be typical to do so.

```gml
/// Constructor. Creates a struct that manages a surface. The surface's width and height must be
/// specified when creating the struct. If the struct falls out of scope and is garbage collected,
/// the surface will be automatically freed ensuring that no memory leaks occur. To obtain the
/// native GameMaker surface stored inside this struct, call the `GetSurface()` method. If you'd
/// like to immediately destroy the surface, please call the `.Destroy()` method. Note that after
/// calling `.Destroy()`, a further call to `.GetSurface()` will cause an error to occur.

function SafeSurface(_width, _height) constructor
{
    width  = _width;
    height = _height;
    
    //Create a destructor to hold our surface
    __destructor = new Destructor();
    with(__destructor)
    {
        surface = -1;
        
        callback = function()
        {
            //Clean up the surface when the host is garbage collected
            if (surface_exists(surface))
            {
                surface_free(surface);
                surface = -1;
            }
        }
    }
    
    static Destroy = function()
    {
        //Terminate the destructor and then unassign it to prevent the surface being recreated
        __destructor.Terminate();
        __destructor = undefined;
    }
    
    //Returns the surface, creating a new surface if necessary. An `initMethod` must be provided.
    //This method will be called if the surface needs to be created/recreated
    static GetSurface = function(_initMethod)
    {
        //We access the surface via the destructor struct
        if (not surface_exists(__destructor.surface))
        {
            destruc__destructoror.surface = surface_create(width, height);
            surface_set_target(__destructor.surface);
            _initMethod();
            surface_reset_target();
        }
        
        return __destructors.surface;
    }
}
```
