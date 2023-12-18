# GameMaker Design Pattern 1: Application Initialization

&nbsp;

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
