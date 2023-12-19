# GameMaker Technique 6 - Multiline Macros

&nbsp;

One of the golden rules in programming is DRY - [Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). DRY can be interpreted in many ways but in the most basic sense DRY includes a basic piece of advice: "If you're using a piece of code multiple times, especially if it's spread over multiple scripts, you should ideally replace that piece of code with a reuseable function call of some sort". This keeps code organised and, if that particular function needs to change for some reason, any changes will propagate across all uses of the function. DRY programming tends to lead to easier code reuse as well (though this isn't always the case).

Rewriting portions of your code to be a function call instead seems like a solid idea on paper but, like with everything, there are downsides. A function call can be unweildy as you may need to pass several values into the function and the function may need to return several values. There is also a non-zero overhead for making function calls in GameMaker - this almost never matters but every now and again it's relevant. Inlining code largely alleviates the issue but only when using YYC.

I'd like to take a moment here to emphasise that I wouldn't use a multiline macro in place of a function call unless it's absolutely necessary. You should only use multiline macros in situations where your code is highly performance sensitive and you're trying to squeeze every drop of speed out of a routine.

At any rate, the solution that is available in many other languages is the concept of a "function macro" i.e. macros that unpack into inline code. Regretably, GameMaker doesn't have function macros. Instead, we have to make do with multiline macros. This requires you to write your code in a certain way to make sure the multiline macro is referencing the correct variables and that those variables contain the correct values but this is a minor point of friction. Consider these two functions:

```gml
function ExampleA(_value)
{
    if (not is_string(_value))
    {
        show_error("Value is not a string", true);
        return;
    }
    
    show_debug_message("ExampleA = " + _value);
}

function ExampleB(_value)
{
    if (not is_string(_value))
    {
        show_error("Value is not a string", true);
        return;
    }
    
    show_debug_message("ExampleB = " + _value);
}
```

There's some shared code between the two functions - the if-statement that checks if the input value is a string. We could replace this with a simple function call but, for the sake of example, let's do it with a multiline macro.

```gml
#macro MULTILINE_MACRO  if (not is_string(_value))\
                        {\
                            show_error("Value is not a string", true);\
                            return;\
                        }

function ExampleA(_value)
{
    MULTILINE_MACRO
    show_debug_message("ExampleA = " + _value);
}

function ExampleB(_value)
{
    MULTILINE_MACRO
    show_debug_message("ExampleB = " + _value);
}
```

Multiline macros must be written such that every line, apart from the final line, has a `\` backslash at the end. Multiline macros also can't have empty lines in them (i.e. a line that's just a backslash) so you'll need to use `;\` for an empty line.

The example above is really simple but multiline macros can get super complex if you want them to be. For some practical examples of multiline macros, see [Scribble](https://github.com/JujuAdams/Scribble/blob/master/scripts/__scribble_gen_6_build_lines/__scribble_gen_6_build_lines.gml) and [Input](https://github.com/offalynne/Input/blob/master/scripts/__input_macros/__input_macros.gml).
