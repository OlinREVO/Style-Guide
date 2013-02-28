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

Braces
------

The opening brace goes on the same line as the statement that opens the block.
Having only a single `{` on a line is a waste of a perfectly good line. 
A single `}` on a line, however, isn't a waste of a line, in breaks up the
code nicely. Please add a space before the opening brace; It's not by any
means essential, but it looks pretty.

```c
int square(int x) {
    return x * x;
}
```

Don't use one-liner conditionals, unless you know that you will _never ever_
need to add to that branch. The following makes sense:

```c
if (x > y)
    return 1;
else
    return 0;
```

But this does not. Maybe you'll find you need to preprocess x one day.

```c
if (x == 17) 
    doSomething(x);
```

In `do while` loops, the `while` goes one space after the closing brace:

```c
do {
    value = readSensor();
} while (value > 255);
```

Expressions
-----------

Please keep a reasonable amount of spaces in your expressions. One on each
side of a binary operator is about right. Also, it doesn't hurt to use a bit
more than the minimum number of parenthesis, just so you, the complier, and
everyone else are clear on the order of operations.

Good:

```c
value = ((readSensor() / dt) * drift)) + calibration;
```

Hard to Read:

```c
value=readSensor()/dt*drift+calibration;
```

Names
-----

The world on an AVR is tiny. There's no need for globally unique names like
`org.olinrevo.wheelspeed.node.rpm`. Keep things short, but understandable,
like, `counter`, `value`, or `tmp`. Often, it makes sense to use
single-character names for function parameters, such as `power(b, e)`. 

If you need a long name, use lowercase for the first word, and start each 
subsequent word with a capital letter, such as, `readSensor` or 
`sendCANMessage`.

Preprocessor macros should be in all uppercase, with underscores between
words, like `FREQ_OFFSET`

Comments
--------

Comments are never _required._ But you should put a line above every function
or macro that explains what it does. Also throw a comment on the line above
any line of code that's not immediately obvious what it does. Don't just
restate the variable names, explain it.

```c
/* Setup and start the timer */
void startTimer() {
    /* Turn on Timer/Counter 1 with a 1/1024 prescalar */
    TCCR1B = (_BV(CS12) | _BV(CS10));
}
```

Datatypes
---------

Generally, stick to descriptive type names like `int`, `long int`, or `float`.
That said, we're working with microprocessors; we're close enough to the
hardware that we often do care about how many bits are in our integers. If
you're just writing math, or some other high level code, use the standard C 
types. If you're interfacing the outside world, writing bitmaps, working
with registers, or anything else where you know exactly how many bits you need
use the exact-width integers available in `inttypes.h`.

If you `#include <inttypes.h>`, you have access to the `intN_t` and `uintN_t`
types. Each of these types provides a signed integer (`intN_t`) or unsigned 
integer (`intN_t`), that is exactly N bits wide, where N is 8, 16, 32, or 64.

The AVR is an 8 bit processor. If you're working with registers, you'll be
using `uint8_t` a lot. We recommend using it for every variable that may
get written to a register. (Especially since `avr/io.h` defines most registers
as `uint8_t`).

A note on floating point numbers. The AVR almost universally lacks a floating
point processor. The AVR libc provides a nice software floating point library,
but it's relatively slow and costs nearly 3k of flash. It will be imported
automatically if you declare a `float` or `double` number. Try to avoid this
if you can. Your code will run faster and lighter if you can find a way to
make things work using integer math or fixed-point notation. Try multiplying
the units you thought needed to be fractional be a factor of 1000 or 1024.
That'll probably give you enough resolution.

Registers and Bits
------------------

If you just want to do math in a vacuum, you can probably get by without ever
using a register. But in real life, you'll need to write to and read from
registers. The AVR design philosophy is to control every peripheral on the
chip with a memory-mapped register.

The header `avr/io.h` is full of preprocessor magic to define names for every
register on the chip you're using. (It figures out which AVR you're using from
the `-mmcu` flag to the AVR GCC.) Use them. `PORTB` is much more 
understandable than `0x05`.

Often, you'll need to write to specific control bits. For registers that have
a straight-forward 1:1 mapping, like `PORTB` or `DDRB`, it generally makes
sense to read and write literally, `DDRB = 0xd3;`, or `PORTB = 1 << 4;`. For
everything else, use the `_BV()` macro and bit names from`avr/io.h`. It 
converts a bit name to a bit mask. Using names makes it super easy to see what
bits you meant to set, since you can just look the names up in your chip's
manual.

Good:

```c
TCCR1B = (_BV(CS12) | _BV(CS10));
```

No idea what this is doing:

```c
TCCR1B = 0x03;
```

Preprocessor Macros
-------------------

The AVR has extremely limited RAM, often as low as 1k. If you find yourself
writing 1-3 line convenience functions to save typing and make you code more
reasonable, consider using preprocessor macros instead. If it gets any longer
than 3 lines, use a function or a subroutine instead.

```c
#define SET_PORTB_BIT(bit) (PORTB |= (1 << bit))

SET_PORTB_BIT(7);
```

Generally, we prefer literals to constants. The pointers you would need to
access a constant are generally going to be longer (bitwise) than the constant
itself. But it gets ungainly when you need to type the same CPU frequency over
and over. So use a preprocessor macro.

```c
#define SENSOR_OFFSET 78

value = readSensor() + SENSOR_OFFSET;
```
