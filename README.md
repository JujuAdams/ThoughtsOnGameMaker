<h1 align="center">Thoughts on GameMaker</h1>

<p align="center">Idle thoughts on the game engine I use for my job</p>

&nbsp;

## GML Techniques

Over the years I've found I've been using and re-using particular GML techniques to solve problems. Below is a list of links to pages that explain them:

- [Technique 1 - Initialization](technique-1-initialization.md)
- [Technique 2 - Singletons](technique-2-singletons.md)
- [Technique 3 - Evacuating global namespace](technique-3-evacuation.md)
- [Technique 4 - Config macros](technique-4-config-macros.md)
- [Technique 5 - Fluent interface](technique-5-fluent.md)
- [Technique 6 - Multiline macros](technique-6-multiline.md)
- [Technique 7 - Pseudo-objects](technique-7-pseudo-objects.md)
- [Technique 8 - Destructors](technique-8-destructors.md)

&nbsp;

## Code Reuse

Code reuse is good actually. Everything should be an API. [Read more here.](code-reuse.md)

&nbsp;

## Formatting

You shouldn't care what formatting someone uses in GameMakerLand unless you're on the same project, at which point the first person to write a line of code wins and you should do what they do. If you must know, I use the following formatting rules:

1. Spaces, not tabs. I want to have complete control over how code looks.
2. Custom asset ordering in the asset browser. I want to have complete control over how scripts are introduced to the user.
3. K&R style for curly brackets (curly brackets start on the new line)
4. camelCase for variables and assets, PascalCase for functions/static methods, SCREAMING_SNAKE_CASE for constants/macros
5. The actual API and internal code should use American spellings wherever relevant. UK spelling for constants etc. should not be supported. Comments are fair game for Britishisms, however.
6. Local (`var`) variables should be prefixed with a single `_` underscore. Private variables, i.e. variables that should not be tampered with or accessed, should be prefixed with two `__` underscores. Woe betide the developer who accesses private variables and expects everything to go smoothly.

You might find some older libraries use a different formatting standard. Time makes fools of us all.

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
