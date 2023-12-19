# GameMaker Technique 5 - Fluent Interface

&nbsp;

A "fluent interface" is a way of chaining together method calls to configure an entity. Fluent interfaces are commonly used to implement a so-called ["Builder" design pattern](https://en.wikipedia.org/wiki/Builder_pattern), a design pattern which is common in the native GameMaker API. For example, let's take a look at the vertex buffer builder:

```gml
//Start filling out a vertex buffer
vertex_begin(buffer, format);

//Top-left corner
vertex_position(buffer, x, y);
vertex_color(buffer, c_white, 1);
vertex_texcoord(buffer, 0, 0);

//Top-right corner
vertex_position(buffer, x + 32, y);
vertex_color(buffer, c_white, 1);
vertex_texcoord(buffer, 1, 0);

//Bottom-left corner
vertex_position(buffer, x, y + 32);
vertex_color(buffer, c_white, 1);
vertex_texcoord(buffer, 0, 1);

//etc.

//Close the vertex buffer
vertex_end(buffer);
```

There's nothing inherently wrong with this syntax but it's pretty long-winded to write. In some situations, it's more convenient to have a compressed syntax to build entities for use. We can rewrite GameMaker's native vertex buffer API to use a fluent interface, and it'd look something like this:

```gml
VertexBegin(buffer, format)
.Position(x,    y   ).Color(c_white, 1).Texcoord(0, 0) //Top-left corner
.Position(x+32, y   ).Color(c_white, 1).Texcoord(1, 0) //Top-right corner
.Position(x,    y+32).Color(c_white, 1).Texcoord(0, 1) //Bottom-left corner
.End();
```

This is much more compact and, at least for me, way easier to read. The trick here is that `VertexBegin()` returns a struct and each following method call returns the struct itself. This enables methods to be chained together using the same scope for each method call. Here's how we might make this fluent interface:

```gml
function VertexBegin(_buffer, _format)
{
    //Start building out the vertex buffer
    vertex_begin(_buffer, _format);
    
    //Define a constructor. We instantiate this constructor further down
    var _inner = function(_buffer) constructor
    {
        //Keep track of what buffer we're building
        __buffer = _buffer;
        
        static Position = function(_x, _y)
        {
            //Poke position data into the vertex buffer
            vertex_position(__buffer, _x, _y);
            
            //Return ourselves so that the next method in the chain is executed in the same scope
            return self;
        }
        
        static Color = function(_color, _alpha)
        {
            vertex_color(__buffer, _color, _alpha);
            return self;
        }
        
        static Texcoord = function(_u, _v)
        {
            vertex_texcoord(__buffer, _u, _v);
            return self;
        }
        
        static End = function()
        {
            //Close the vertex buffer, we're done!
            vertex_end(__buffer);
            
            //Prevent further methods from writing to the vertex buffer
            __buffer = undefined;
            
            //Explicitly prevent further chained methods after .End()
            return undefined;
        }
    }
    
    //Instantiate the above constructor and return it for use
    return new _inner(_buffer);
}
```

For a practical example, see [Scribble](https://www.github.com/jujuadams/Scribble). Scribble's usage is looser than the above vertex buffer exampleand includes a layer of caching. However, the basic technique remains the same: methods should return `self` so that further methods can be called in the same scope.
