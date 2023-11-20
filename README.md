<h1 align="center">Thoughts on GameMaker</h1>

<p align="center">Idle thoughts on the game engine I use for my job</p>

&nbsp;

### Versioning

This is the versioning scheme I prefer for open source libraries: `major.minor.patch`. For example, the latest Scribble version at the time of writing is `8.7.0`: major = 8, minor = 7, patch = 0.

- Major version for breaking changes
- Minor version for backwards compatible changes (e.g. a user can update to that version without writing new code but might not be able to downgrade)
- Patch version for changes that fix bugs and don't otherwise change the API

Examples:

- New API feature = minor version
- Hotfix for dumb mistake = patch version
- SDL database update = patch version
- Removal of an API feature = major version
- Adding a new optional argument = typically a minor version, sometimes a major version depending on new default behaviour
