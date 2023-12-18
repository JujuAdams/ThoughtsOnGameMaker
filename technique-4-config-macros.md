# GameMaker Technique 4: Config Macros

&nbsp;

Macros are a way to insert values into your code by referencing a handy name rather than by reusing the same value in multiple places. For example, you could comfortably set a gravity value that is shared across many objects by using a macro. Using macros in this way allows you to keep consistency across many places in your codebase even if the value itself needs to change. Any change to the macro, which is stored in one location, is propagated across every use of that macro. Macros are very useful in this role.

Macros have another interesting property, however. When you compile your game, and use of a macro is replaced by GameMaker's precompiler with the literal content of the macro. A macro is not a variable, therefore. A variable has to be evaluated at runtime in order for your game to know what value it contains. This means looking up a value at a particular location in memory, reading the value, then using that value in a calculation. Macros are a bit different - because the literal value is strictly defined, the memory look-up doesn't need to be done. There's an even more useful side effect of this: if you use a macro in an if-statement, if could be the case that the entire if-statement can be evaluated whe compiling the game. This means the if-statement might literally not exist at runtime, leading to significant performance boosts.

We can lean into the performant nature of macros. Using macros to control configuration of a library can therefore add a lot of flexibility without the downside of potentially causing performance problems due to code branches being evaluated in performance-sensitive areas e.g. loops. Config macros are also "baked" into the game meaning that it's harder for a malicious party to edit the value of that macro after the game is made (especially if you're exporting using YYC).

Here's an example:

```gml
#macro TREE_DEBBUG  true

//Imagine we have hundreds of trees...
with(oTree)
{
    if (TREE_DEBUG) show_debug_message(string(id) + ": " + string(x) + "," + string(y));
    draw_sprite(sTree, 0, x, y);
}
```

Macros can also be used in combination with GameMaker's configuration profiles to allow you to alter behaviour.

```gml
#macro 
```
