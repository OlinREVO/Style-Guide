C Style Guide for the AVR
=========================

Style guides are have a tendency to grow long and arbitrary, and to frustrate
developers who just want to write code. Here at REVO, we want to minimize
frustration for you, but we're working with embedded systems, and it can be a 
real pain to debug code when all you can do is blink an LED. Hopefully these
guidelines will help you write code that is easy to read and easy to debug.

Indentation
-----------

Indentation is 4 spaces. Always. No tab characters allowed. If you mix tabs
and spaces, it may look fine in your editor, but it will probably look really
silly when someone else goes to read it. 

Most editors have an option to automatically indent your code so that you
barely have to use your tab key. And most will let you configure your tab key
to output the appropriate number of spaces. We use 4 spaces because it's the
default in most editors.

The Arduino editor uses 2 space indents for some reason. If you are porting 
AVR code from an Arduino, please fix this. This isn't HTML, if you're writing
100 levels of nested code, it's going to be hard to debug.

Any new block of code (surrounded by curly braces) or a line continued from a
previous line should be indented one level, with one exception, the `switch`
statement. Only the inside of `case` blocks should be indented one level
beyond the `switch` statement, the `case` statements are indented to the same
level as the `switch`:

```c
switch (err) {
case -3:
    str = "Reactor overheated.";
    break;
case -2:
case -1:
    str = "Fault in fuel loading system.";
    break;
default:
    str = "Unkown reactor error.";
    break;
}
```
