Over at Broxcorp we have a template project that we use for every new prototype / project. A few of these are libraries I write and maintain, these are linked, but most are bespoke to the studio. Each component is its own API that we can slice out of the project if we're not using it or we can import if we need it.



Here's a list:

Better Shader API - Shader push/pop stack. Allows us to more easily manage shaders when there's a lot going on
Bezier Curves - We used to use this for animation curves. Largely defunct now
Debug Output - Various [ICODE]show_debug_message()[/ICODE] calls that behave in different ways
Distortion - Frag shader screen distortion. Used for shockwaves etc. A creative use was the wind effects in Skies Of Chaos
Dynamo - Live datafile reload tool. Can also be used to live reload simple scripts that define variables. We used this for balance adjustments and shader tweaking
Hitflash - Hitflash shader with an added struct-based controller. We found we were tracking a lot of hitflash states in different places so it made sense to centralise it
Input - Very useful for its unified keyboard / gamepad / virtual button handling
Kawase - Fast approximation of Gaussian blur. Used for fake depth-of-field effects and fake soft shadows
Low Spec Detector - Mobile performance varies wildly. We use some basic device stat checks to predict what devices are likely to cause us problems and then we turn off expensive graphics features to compensate (like Kawase above!)
Lyre - Localisation system designed for fast iteration when working with a localisation team
Mobile Haptics - Maximum compatibility, minimum viable product, haptics system for mobile
Palette Swap Shader - Non-indexed palette swap. Has tolerance values so it works better on antialiased art. An evolution of the system I wrote for Swords Of Ditto
PictureFrame - Camera and resolution handler. GameMaker's view/camera/app surf size/GUI size handling is painful so we tried to make it easier for ourselves. More work to be done in this area I think
Pills - Draws pill shapes. We have a particular "house style" for our UI and pill shapes are a big part of that!
Pop Caption - Draws comic book-style pop-up text. Example animation here, look out for the "Damn" and "HEALTH" text
Post-processing - Common post-processing shader effects all in one place. Includes vignette, tinting, chromatic aberration, and per-pixel distortion
PRNG - Replacement for GameMaker's internal RNG. Allows us to more precisely control randomisation which is critical for networking
Recolour - Tinting shader that operates on the perceived brightness of the input image. Looks more natural than HSV-based tinting
Resolution Library - List of common device resolutions. Useful for testing
Save System - Combination of db and a bespoke async file i/o library
Scribble - Handles most text drawing. Skies Of Chaos was localised into 33 languages, including Thai and Arabic, and GameMaker's native text solution didn't cover what we needed
SNAP - Converts between data representations. Not used all that much but helpful nonetheless
Squash & Stretch - Struct-based state machine for squash-and-stretch-style effects
Texan - TEXture mANager. Essential for Skies Of Chaos to keep memory within budget
Transforms - Easier API for world matrix transforms. Very useful for doing little animations on menu UI
Vinyl - Audio manager with runtime configuration and mixing
This is a lot, and we haven't even talked about networking or the Netflix SDK extension or various bits of tech art or delta timing or ...



Your game doesn't need all of this. I'm wary about these sorts of threads because everyone's workflow is different and it's easy to come away with this foreboding sense that you need to solve every problem before making a game. Nothing is further than the truth. Case in point, I've also had the opportunity to work with JW on games like Minit and Disc Room and the code he shares between projects and prototypes is the following:

Menu system - A keyboard/gamepad-driven menu with no animations
If he can make excellent games without shared code so can you. (Apart from UI code apparently, because UI sucks.)



For your very first game you probably should largely write the code yourself. Understanding why a library or component or someone else's technique is useful does genuinely require a bit of hands-on experience first. Build up your own library of useful components that you like to use, sure, but also go out and learn from other people. It's a lot faster to learn from other's mistakes. Game development is a collaborative process, even if you're not working on the same game and on the same team, and even if you're not working in the same engine. Listen to what's happening out there and you'll make life a lot easier.
