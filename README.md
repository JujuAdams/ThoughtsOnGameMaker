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

## GameMaker Design Pattern 1: Singletons

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

The most common over-use of the singleton pattern is when creating the player instance. Assuming that one single player instance always exists not only makes a multiplayer conversion incredibly challenging (moreso than it already is) but relying on a single player instance existing at all times is also liable to introduce severe crashes.
