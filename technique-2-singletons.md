# GameMaker Technique 2: Singletons

&nbsp;

The [singleton pattern](https://en.wikipedia.org/wiki/Singleton_pattern) is one of the most common design patterns and it's, pleasingly, one of the easiest to implement in GameMaker.

The basic idea with the singleton pattern is that one instance - and **only** one instance - of a particular object should exist. This instance should also **always** exist. You can extend this basic idea out to structs as well. The core idea is that you have a singular magical instance of an object that you can always refer to. This instance is also very commonly persistent so that it carries over between rooms.

The utility of this is probably immediately apparent but let's talk about it. Having a specific object set up to manage global functionality keeps all your code in one place and it makes it very clear in your codebase which instance is responsible for managing that functionality. This makes it easier to modularise code if you're into that but, more importantly, it makes the code plain easier to read. That the instance is persistent also means that a singleton can be used to persist state between rooms without clogging up namespace with a global variable.

Building a singleton object is pretty easy:

1. Ensure the object is set to persistent.
2. In an initialisation room/object, create an instance of the object.
3. If you're using instance deactivation, ensure that you reactivate the singleton as necessary.
4. Don't call `instance_destroy(all)`. If you absolutely have to, make sure you re-create the singleton instance.

That's it! Common use cases for singleton objects are background music control and player input handling but the uses are myriad: menu/GUI controllers, save system controllers, networking controllers ... the list is endless and, often, game-specific.

Even novice users have probably encountered the singleton pattern and I'd profer that every single commercial GameMaker game ever made uses the singleton pattern somewhere. But there are limitations. The singleton pattern is effective for handling music playback or gamepad input, stuff that reasonably is global in scope and affects everything in a game, but the singleton pattern should **never** be used outside of that or you'll run into problems as your game grows during development.

The most common over-use of the singleton pattern is when creating the player instance. Assuming that one single player instance always exists not only makes a multiplayer conversion incredibly challenging (moreso than it already is) but relying on a single player instance existing at all times is also liable to introduce severe crashes. The singleton pattern should be used sparingly and only in situations where what that singleton has to do is very specific.
