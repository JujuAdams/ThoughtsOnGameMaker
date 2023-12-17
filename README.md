<h1 align="center">Thoughts on GameMaker</h1>

<p align="center">Idle thoughts on the game engine I use for my job</p>

&nbsp;

## Versioning

This is the versioning scheme I prefer for open source libraries: `major.minor.patch`. For example, the latest Scribble version at the time of writing is `8.7.0`: major = 8, minor = 7, patch = 0.

- Major version is incremented for breaking changes
- Minor version is incremented for backwards compatible changes (e.g. a user can update to that version without writing new code but might not be able to downgrade)
- Patch version is incremented for changes that fix bugs and don't otherwise change the API

Examples:

- New API feature = minor version
- Hotfix for dumb mistake = patch version
- SDL database update = patch version
- Removal of an API feature = major version
- Adding a new optional argument = typically a minor version, sometimes a major version depending on new default behaviour

&nbsp;

## Pull Requests

I'll only accept pull requests that fulfill the following requirements:

- **Consulted** — Check whether the PR is, in principle, acceptable with the repo maintainer before spending your time making one.
- **Small** — The PR should contain as few changes as possible. If it's a big change, it's harder to review.
- **Specific** — The PR should address only the specific feature or bug and not contain multiple fixes for multiple things at the same time.
- **Solution-oriented** — The PR should cover a known issue. PRs that "tidy up" a project are unhelpful.
- **Commented** — Comment your code, folks.
- **No IDE / GameMaker updates** — The PR should not require updating the GameMaker IDE or using a different runtime than the library is built for.

I hate writing documentation. Special love is given to people who fix typos or broken links in documentation, or generally improve documentation at all.

&nbsp;

## GameMaker Design Pattern 1: Application Initialization

There's a lot to do when a game boots up. Loading savedata, initializing global state-tracking variables, loading and applying game settings, import localisation, position the window or setting fullscreen mode, showing a short intro video, fetching texture pages, establishing connections to analytics services, the list goes on and on. It's often the case that these steps are asynchronous and some actions rely on the result of previous steps. In some cases, initialization might take so long that it's necessary to show a loading bar.

I've found in my professional work that most every game is improved by a clear and organised initialisation flow. Having the initialization flow spread out across multiple frames is a safer way to program. On console, any long loading times will constitute a compliance fail and the platform holder will refuse to release your game. On desktop PC, a long hang may cause the OS to think that your game has locked up and may try to terminate the application. Players on mobile tend to be sensitive to load times and many will shut your app is it's taking too long (either out of boredom or because they think the game has crashed).

To describe it simply, game initialization should be executed as a finite state machine that advances through a list of states. This finite state machine should be executed by a single instance of an initialization object in a separate room that is the first room to be visited. At a high level, a basic initialization flow might look like this:

1. Initialize settings
2. Start asynchronous load of settings
3. Wait for settings to finish loading
4. Apply window/fullscreen setting
5. Move to the next room (typically a main menu)

Seems easy enough. As mentioned above, this process should be handled by a specific object dedicated to the task. Let's call that object `oAppInitialize`. An instance of this object should be created in the first room in the game e.g. `rAppInitialize`. This ensures that the app isn't accidentally reinitialized (though advanced users may well want to figure out ways to reinitialize their game for the occasional situation where that's helpful).

The meat of this design pattern is the simple finite state machine (FSM) in `oAppInitialize`. Example code follows:

```gml
/// Create Event

//Variables to track FSM state
initPhase = 0;
initPhaseTimer = 0;

//Variables to track settings buffer state
loadBuffer = undefined;
loadID = undefined;
loadState = 0; //-1 = error, 0 = pending, 1 = success



/// Step Event

//We can force a pause in execution of the FSM if we want
//This is handy when changing window state (see below)
if (initPhaseTimer > 0)
{
    --initPhaseTimer;
}
else switch(initPhase)
{
    case 0:
        //Initialize our settings to a known good and valid state
        global.settings = {
            fullscreen: true,
        };

        //Execute the next phase in the next step ...
        initPhase = 1;
    break;
    
    case 1:
        //Start an async buffer load for the game settings
        //We're using an async load here for compatibility with consoles
        loadBuffer = buffer_create(1024, buffer_grow, 1);
        loadID = buffer_load_async(buffer, filename, 0, -1);
        loadState = 0;
        
        initPhase = 2;
    break;

    case 2:
        //Only execute code if we've received an async event for the buffer load
        //(See the "Async Save/Load Event" below)
        if (loadState != 0)
        {
            //If we've successfully loaded the buffer, read our fullscreen state into our settings
            if (loadState == 1)
            {
                global.settings.fullscreen = buffer_read(loadBuffer, buffer_bool);
            }

            //Always clean up your memory!
            buffer_delete(loadBuffer);

            //Only proceed to the next phase when the async event fires
            initPhase = 3;
        }
    break;

    case 3:
        //If we're on desktop PC, set our fullscreen state based on what's in our settings
        //(In a real game you'd probably use a function for this)
        if ((os_type == os_windows) || (os_type == os_macosx) || (os_type == os_linux))
        {
            window_set_fullscreen(global.settings.fullscreen);
        }

        //Force a 10 frame pause. This is recommended in the GameMaker manual to ensure the
        //fullscreen transition happens smoothly
        initPhaseTimer = 10;
        initPhase = 4;
    break;

    case 4:
        //We done! On to the next room
        room_goto_next();
    break;
}



/// Async Save/Load Event

//If this async event is for the settings buffer load, set our load state based on the
//success of the operation (-1 for an error, 1 for a success)
if (async_load[? "id"] == loadID)
{
    loadState = async_load[? "status"]? 1 : -1;
}
```

&nbsp;

## GameMaker Design Pattern 2: Singletons

The [singleton pattern](https://en.wikipedia.org/wiki/Singleton_pattern) is one of the most common design patterns and it's, pleasingly, one of the easiest to implement in GameMaker.

The basic idea with the singleton pattern is that one instance - and only one - instance of a particular object should exist. This instance should also **always** exist. You can extend this basic idea out to structs as well. The core idea is that you have a singular magical instance of an object that you can always refer to. This instance is also very commonly persistent so that it carries over between rooms.

The utility of this is probably immediately apparent but let's talk about it. Having a specific object set up to manage global functionality keeps all your code in one place and it makes it very clear in your codebase which instance is responsible for managing that functionality. This makes it easier to modularise code if you're into that but, more importantly, it makes the code plain easier to read. That the instance is persistent also means that a singleton can be used to persist state between rooms without clogging up namespace with a global variable.

Building a singleton object is pretty easy:

1. Ensure the object is set to persistent.
2. In an initialisation room/object, create an instance of the object.
3. If you're using instance deactivation, ensure that you reactivate the singleton as necessary.
4. Don't call `instance_destroy(all)`. If you absolutely have to, make sure you re-create the singleton instance.

That's it! Common use cases for singleton objects are background music control and player input handling but the uses are myriad: menu/GUI controllers, save system controllers, networking controllers ... the list is endless and, often, game-specific.

Even novice users have probably encountered the singleton pattern and I'd profer that every single commercial GameMaker game ever made uses the singleton pattern somewhere. But there are limitations. The singleton pattern is effective for handling music playback or gamepad input, stuff that reasonably is global in scope and affects everything in a game, but the singleton pattern should **never** be used outside of that or you'll run into problems as your game grows during development.

The most common over-use of the singleton pattern is when creating the player instance. Assuming that one single player instance always exists not only makes a multiplayer conversion incredibly challenging (moreso than it already is) but relying on a single player instance existing at all times is also liable to introduce severe crashes. The singleton pattern should be used sparingly and only in situations where what that singleton has to do is very specific.

&nbsp;

## GameMaker Design Pattern 3: Evacuating Global Scope

Whilst you might not someone writing a library for public consumption, it's a good habit to get into to view independent sections of your code as libraries. You might want to call them "modules" or "components" or "subsystem" or just "APIs", but at any rate, keeping your code cleanly divided makes maintenance and teamwork much much easier. There has been endless ink spilt on the topic of code encapsulation and I wouldn't pretend to have anything to add to the discussion other than "encapsulation is good".

GameMaker, regrettably, can make good practice hard, however, and the ease by which we can stuff more and more variables into global scope allows bad habits. When writing libraries, it's often useful to hold information in global scope for convenience and, from time to time, necessity. But this has negative side effects: Because a game and any libraries necessarily share the same global scope, it's easy to mix up variables from different components. A variable called `global.cameraX` could be in use by any library or come from game code itself. In the worst case, critical global variables could be accidentally overwritten by other bits of code, especially if those global variables have a commonly used name (such as `global.debugMode`). Additionally, having a bunch of variables in global scope, even if intelligently and clearly named, clogs up GameMaker's debug view. This can make it very hard for anyone using a library to find the global variables they're interested in. It's important to consider the user experience of using a library from multiple perspectives and ensuring a library doesn't make the user experience worse when debugging is part of that.

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
