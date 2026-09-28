# Desmos-supported programming language (incomplete)

## Data types
- Number: JavaScript integer, may contain decimals
- List: May contain only up to 10,000 items. All elements of a list MUST have the same type. Lists in lists are NOT supported.
- Polygon: May contain only up to 10,000 items. Polygons may not contain 3d points.
- Point: A point may be 2d or 3d. Points may contain imaginary numbers but will result in an undefined value.
- Color: may be defined using rgb, hsv, okhsv, oklab, oklch

## Operators:
Numbers:
- +
- -
- *
- / (may also use \frac{a}{b})
- \times or \cross
- a(b)
- a^b or a^{b}
- a!
Points:
- 
List:
- [a,b] operator [c,d] = [a operator c, b operator d]
- [a,b] operator c = [a operator c, b operator c]
## Variables
Variables must contain only one letter that is NOT x, y, or e. Variables may contain subscripts of any length which may only contain letters and digits. Variables may also contain the following Greek letters (but not as a subscript):
- 
Variables can be defined in a function, but a variable CANNOT be based off of another variable if in a function.
Example:
OK (equals `a+b with a=1,b=2`):
```
a = 1
b = 2
a + b
```

Not OK (equals `a+b with a=1,b=a`):
```
a = 1
b = a + 1
a + b
```

Variables may only exist in one level at a time (because the equivalent would mean a with statement within a with statement)
Not OK (equals `(a+b with b=2) with a=1`)
```
a=1
(
  b=2
  a + b
)
```
## Parametric equations

## Functions:
Recursion is allowed, but only up to 10,000 times.
## List operations:

## Constants:
\pi
\tau
\e
\infty or \infinity
## Piecewise:
All values in a piecewise function must have the same type with the following exceptions:
- 
Syntax:
{expression1:a,expression2:b,default}
If a default value is not provided, it returns undefined. Expressions are evaluated from left to right and stop on the first true statement.
Piecewise functions can also be written in if statement format:
if expression1:
  a
else if expression2: # elif (expression2):
  b
else:
  default
and
if expression1: a
else if expression2: b
else: default
is equivalent to {expression1:a,expression2:b,default}

ternary operators are supported in the following format:
value_if_true if condition else value_if_false
condition?value_if_true:value_if_false
## Syntax:

## Loops:
Syntax:
L = i for i = [1...10]
or
L = (
  for i = [1...10]:
    i
)

`for i = b` is equivalent to `for i of b` is equivalent to `for i in b`
parenthesis after `for` is ok (eg. `for (i of b)`)

## Special:
Comments are supported using #, //, /* */
Data type prefixes are supported but not required:
Number/int, Point, List<Type>/Array<Type>, Polygon, Color, Any, ListAny, ListNumber, ListPoint, ListColor, ListPolygon, List (same as List<Number>)
Language supports either indentation-based or bracket based, but they MUST be consistent

## Examples:
