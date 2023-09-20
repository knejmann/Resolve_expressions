# Resolve expressions
About the use of expressions in effects in Blackmagic Resolve
This page is a supplement to the few videos I've shared on Youtube: [Generating a counter with thousands separator/delimiter in DaVinci Resolve](https://youtu.be/Y-RYs49gtvo) and [Resolve time counter as expression](https://youtu.be/5cV6Py8-zM8)

# Expressions in Text
You add a text effect by dragging it from Effects --> Toolbox --> Titles --> Text+ to the desired place on your timeline.
Next in the Indpector right click in the Text field and choose Expression. This should add an expression box below the text box:

![image](https://github.com/knejmann/Resolve_expressions/assets/18518513/38eee478-9537-4d1e-b280-2b0f60f9fb20)

Any changes you now make to the Text field have no effect on the output. All changes must be made with the expression.

The expressions are written in the language LUA. You can find more [information on LUA online](https://www.lua.org/pil/contents.html).

A few things to be aware of. A lot of things are case sensitive - comp.RenderEnd is not the same as Comp.renderEnd.

Variables do not have prefixes like $variable in PowerShell.

Text must be enclosed in quotation marks like `""` or `''`

# Basic expressions
The default expression `Text("Custom Title")` simply outputs the text __Custom Title__

If you set the expression to `Size` your will get the current font size. `Size * 100` displays the fontsize multiplied by 100.

# Basic counting and a bit of formatting
The expression `time` gives you the frame number relative to the start of the effect. First frame is 0.

__time__ is one of the variables Resolves lets you use in expressions.

__comp.RenderEnd__ is another variable: The duration of the Text+ effect.

Those two combined with a bit of math can give us a count from 0 to 1 over the duration of the effect: `time / comp.RenderEnd`

If we want this to be in percents we just multiply by 100: `time / comp.RenderEnd * 100`. This renders the number with a lot of decimal places. If we don't want that we can round the result down with a LUA Math function: `math.floor(time / comp.RenderEnd * 100)`. Or we can use string formatting like this: `string.format("%.2f"  , time / comp.RenderEnd * 100)` to get the percent value with two decimals.

You can find more information on the format flags (like %.2f) used by string.format [here](https://faq.cprogramming.com/cgi-bin/smartfaq.cgi?answer=1048379655&id=1043284385). It uses the syntax from the programming language C.

# Multi line expressions
Expressions must be on one line and return one value by default - like the examples above.
But to do more complicated expressions - or to improve readability - we can use multiple lines by making the first character of an expression `:`.

Resolve then allows to do more 'programming' like defining our own variables and using if-then-else statements.

The final result is returned by a statement like `return(value)`

# Counter with prefix and postfix

```
: -- setup:
from = 0; -- starting value
to = 123456; -- ending value
prefix = "pre " -- text before the number
postfix = " post" -- text after the number
-- setup end
count = from + (to-from)*(time/comp.RenderEnd)
formatted = string.format("%i", count)
return(prefix .. formatted .. postfix)
```


# Counter with thousands delimiter, prefix, postfix

```
: -- setup:
from = 0; -- starting value
to = 123456; -- ending value
delim = "," -- the text inserted as a thousands separator.
prefix = "x" -- text before the number
postfix = "y" -- text after the number
-- setup end
a = math.floor(from + ((to-from)*time/comp.RenderEnd)) ; arev = string.reverse(a)
delimited = ""
count = 0
for char in arev:gmatch "%d" do
if count == 3 then
delimited = delimited .. delim
count = 0
end
delimited = delimited .. char
count = count + 1
end
return(prefix .. string.reverse(delimited) .. postfix)
```

# Counter with thousands delimiter, prefix, postfix - improved

```
: -- setup
from = 0; -- starting value
to = 123456; -- ending value
delim = "," -- the text inserted as a thousands separator.
prefix = "pre " -- text before the number
postfix = " post" -- text after the number
-- setup end
number = from + (to-from)*(time/comp.RenderEnd)
delimited = (string.format('%d', number)):reverse():gsub("(%d%d%d)","%1" .. delim):gsub(",(%-?)$","%1"):reverse()
return(prefix .. delimited .. postfix)
```


# Time counter

```
:
from = (os.time{year=2023, month=1, day=1, hour=0, min=20, sec=0}) -- starting value
to = (os.time{year=2023, month=1, day=1, hour=0, min=0, sec=0}) -- ending value
format = "%H:%M:%S" -- date format
prefix = "" -- text before the date
postfix = "" -- text after the date
-- setup end
datevalue = math.floor(from + ((to-from)*time/comp.RenderEnd))
return(prefix .. os.date(format, datevalue) .. postfix)
```
