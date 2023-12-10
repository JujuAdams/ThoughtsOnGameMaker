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

I've found in my professional work that most every game is improved by a clear and organised initialisation flow. Having the initialization flow spread out across multiple frames is a safer way to program. On console, any long loading times will constitute a compliance fail and the platform holder will refuse to release your game. On PC, a long hang may cause the OS to think that your game has locked up and may try to terminate the application. Players on mobile tend to be sensitive to load times and many will shut your app is it's taking too long (either out of boredom or because they think the game has crashed).

To describe it simply, game initialization should be executed as a finite state machine that advances through a list of states. This finite state machine should be executed by a single instance of an initialization object in a separate room that is the first room to be visited. At a high level, a basic initialization flow might look like this:

1. Initialize settings
2. Start asynchronous load of settings
3. Wait for settings to finish loading
4. Apply window/fullscreen setting
5. Move to the next room (typically a main menu)

Seems easy enough. As mentioned above, this process should be handled by a specific object dedicated to the task. Let's call that object `oAppInitialize`. An instance of this object should be created in the first room in the game e.g. `rAppInitialize`. This ensures that the app isn't accidentally reinitialized (though advanced users may well want to figure out ways to reinitialize their game for the occasional situation where that's helpful).

The meat of this design pattern is the simple finite state machine in `oAppInitialize`. Example code follows:

```gml
/// Create Event
initPhase = 0;
initPhaseTimer = 0;

loadBuffer = undefined;
loadID = undefined;
loadState = 0; //-1 = error, 0 = pending, 1 = success



/// Step Event
if (initPhaseTimer > 0)
{
    --initPhaseTimer;
}
else switch(initPhase)
{
    case 0:
        global.settings = {
            fullscreen: true,
        };
        
        initPhase = 1;
    break;
    
    case 1:
        loadBuffer = buffer_create(1024, buffer_grow, 1);
        loadID = buffer_load_async(buffer, filename, 0, -1);
        loadState = 0;
        
        initPhase = 2;
    break;

    case 2:
        if (loadState != 0)
        {
            if (loadState == 1)
            {
                global.settings.fullscreen = buffer_read(loadBuffer, buffer_bool);
            }
            
            buffer_delete(loadBuffer);
            initPhase = 3;
        }
    break;

    case 3:
        window_set_fullscreen(global.settings.fullscreen);
        
        initPhaseTimer = 10;
        initPhase = 4;
    break;

    case 4:
        room_goto_next();
    break;
}



/// Async Save/Load Event
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
