# Desmos-supported programming language — draft reference

## 1. Values and types

| Type | Description |
|---|---|
| `Number` or `int` | A JavaScript-style numeric value; decimals are allowed. `int` is an alias, not a separate integer-only type. |
| `Point` | A 2D or 3D point. Points with imaginary coordinates produce an undefined value. |
| `List<T>` or `Array<T>` | An ordered, homogeneous list of at most 10,000 items. `T` cannot itself be a list. |
| `Polygon` | A polygon containing at most 10,000 items. Its vertices must be 2D points. |
| `Color` | A color constructed with `rgb`, `hsv`, `okhsv`, `oklab`, or `oklch`. |
| `Any` | An unconstrained value, primarily for function parameters or intermediate results. Its use does not waive list homogeneity. |

Convenience type names:

| Name | Meaning |
|---|---|
| `List` or `ListNumber` | `List<Number>` |
| `ListPoint` | `List<Point>` |
| `ListColor` | `List<Color>` |
| `ListPolygon` | `List<Polygon>` |
| `ListAny` | A list whose element type is inferred when possible; it is *not* permission to mix element types. |

A declaration may omit its type:

```text
Number a = 3
List<Point> p = [(0, 0), (1, 0)]
c = rgb(255, 0, 0)
```

`undefined` is an evaluation result rather than a distinct type that can be declared. Operations requiring a valid value propagate undefined unless their documentation says otherwise.

### Numbers and constants

The supported named constants are `\pi`, `\tau`, `\e`, and `\infty` (also spelled `\infinity`).

Numeric literals include integers and decimals, such as `2`, `-4`, and `0.25`. `\e` denotes Euler’s number; it does not make the reserved letter `e` available as a variable name.

### Points, polygons, and colors

Points use `(x, y)` or `(x, y, z)`; coordinate access uses `.x`, `.y`, and, for 3D points, `.z`. Polygons expose an ordered `.vertices` list of 2D points. To construct a polygon, use the syntax polygon((x1,y1), (x2, y2))

Color constructors are supported, but their component ranges have not been specified. Use the target Desmos implementation’s ranges and conventions for `rgb`, `hsv`, `okhsv`, `oklab`, and `oklch`.

## 2. Names, declarations, and scope

A variable name consists of **one letter** other than `x`, `y`, or `e`. It may have a subscript of any length containing only letters and digits. The listed Greek letters are also permitted as variable names, but not as subscripts; the list of permitted Greek letters still needs to be supplied.

In plain-text source, `a_index1` represents a subscripted name. Thus `a`, `b`, and `a_index1` are valid names, while `totalForce` is not a variable name.

Inline variables may contain letters without subscripts but may not contain a digit as its first character

A variable can be declared with `name = expression` or `Type name = expression`. The `inline` modifier requests direct substitution at each use:

```text
inline Number r = distance(p_1, p_2)
```

`inline` affects compilation, not the value or scope of `r`; its expression must still be valid everywhere it is substituted.

### Local declarations (“with” expressions)

A parenthesized, multiline expression may declare variables and finish with a value:

```text
(
  a = 1
  b = 2
  a + b
)
```

Variables declared **inside** such an expression cannot depend on other variables declared at the same level:

```text
# Invalid: b depends on a declared in the same local scope.
(
  a = 1
  b = a + 1
  a + b
)
```

Dependencies between locally declared variables are allowed at the top level:

```text
a = 1
b = a + 1
a + b
```

A local variable may also depend on a variable from an enclosing top-level scope:

```text
a = 1
(
  b = a + 1
  a + b
)
```

Local declaration scopes cannot nest. This also applies inside a function: a variable declared in the function’s local expression cannot depend on another declaration in that same expression.

Function parameters and iteration variables are inputs to a local scope, rather than local declarations for this dependency rule. This permits a local definition to use its function parameter or loop iterator.

## 3. Expressions and operators

### Numeric operations

| Syntax | Meaning |
|---|---|
| `a + b`, `a - b` | Addition, subtraction |
| `a * b`, `a \times b`, `a(b)` | Multiplication |
| `a / b`, `\frac{a}{b}` | Division |
| `a \cross b` | Cross-product-style operation; its supported operand types need specification |
| `a^b`, `a^{b}` | Exponentiation |
| `a!` | Factorial |
| `a % b` | Remainder |

Comparisons use `<`, `<=`, `>`, `>=`, `=`, or `==`; logical conditions use `and` and `or`. Assignment `=` is distinguished from comparison `=` by context.

Standard mathematical precedence applies: parentheses; factorial; powers; unary signs; multiplication/division/remainder; addition/subtraction; comparisons; logical operators. Parenthesize expressions whose interpretation matters.

### Point operations

The example uses point subtraction, point addition, multiplication by a number, and summation of point values:

```text
p_1 - p_2
p_1 + p_2
p_1 * 0.5
```

These are coordinate-wise operations on points of matching dimension. Division of a point by a nonzero number is also coordinate-wise. Other point operations, including `\cross`, require an explicit definition before use.

### List operations

For a binary operator supported by the element type, a list operation is element-wise:

```text
[a, b] + [c, d]  # [a+c, b+d]
[a, b] * c       # [a*c, b*c]
```

Operations between two lists of different lengths will trim the list to the shortest length.

### Piecewise expressions and conditions

A piecewise expression tests conditions from left to right and returns the first matching value:

```text
{condition_1: value_1, condition_2: value_2, default_value}
```

Without a default, it returns undefined when no condition matches. All branch values must have the same type, except for any exceptions that are to be specified.

The equivalent statement-style form is:

```text
if condition_1:
  value_1
else if condition_2:
  value_2
else:
  default_value
```

`elif` is an alias for `else if`. Ternary forms are also supported:

```text
value_if_true if condition else value_if_false
condition ? value_if_true : value_if_false
```

Conditions yield truth values usable by piecewise expressions; boolean values are not otherwise a declared data type. The same branch-type rule applies to all three forms.

## 4. Lists, indexing, and iteration

A list literal is `[value_1, value_2, ...]`. Every element must have the same type, and lists cannot contain lists.

Indexing uses `list[index]`, starts at **1**, and returns undefined for an out-of-range index. 

Iteration constructs produce lists:

```text
L = i for i = [1...10]
```

The block form is equivalent:

```text
L = (
  for i = [1...10]:
    i
)
```

`for i = values`, `for i of values`, and `for i in values` are equivalent; parentheses around the iterator clause are allowed, as in `for (i of values)`.

`[a...b]` is an inclusive integer range, and iterations preserve source order. A typed iterator can be written `for Point p in points`. Each loop body evaluates to one list element.

Nested loops do not flatten into one list. They do not produce a list of lists, which would violate the data-type restriction.

`count(list)` returns its number of items; `total(list)` sums numeric elements or adds point elements coordinate-wise. 

## 5. Functions and recursion

Functions are supported, and recursion is limited to 10,000 calls.

Function definitions use a name, parameters, and an expression-valued body:

```text
distanceSquared(p_1, p_2):
  (p_1.x - p_2.x)^2 + (p_1.y - p_2.y)^2
```

The last expression is the result; no `return` keyword is required. Parameter and result type annotations are optional.

A function body may use local declarations subject to the scope rules in §2:

```text
midpoint(p_1, p_2):
  (
    a = (p_1.x + p_2.x) / 2
    b = (p_1.y + p_2.y) / 2
    (a, b)
  )
```

Here neither `a` nor `b` depends on the other. Exceeding the recursion limit evaluates to undefined.

## 6. Blocks, comments, and source style

Comments may use `#`, `//`, or `/* ... */`:

```text
# One line
// Another line
/* Multiple
   lines */
```

## Other things

Polygon.vertices returns a List<Point>
\cross between two points cannot be used on 2d points. It can be used on 3d points
\cross between a point and number can be used on a 2d point (same as dot) but not a 3d point
polygons can be constructed like:
polygon((a,b),(c,d))
polygon([(a,b),(c,d)])
polygon([a,c],[b,d])
You cannot take the Factorial of any point
Numeric equality can be represented as either = or ==
Exception to same-type piecewise branch: undefined values can be mixed with any value
The not operator is not supported
All paths in a piecewise expression is evaluated at the same time. If an error occurs in a path that is not called, it will not execute
The ok_line can still execute in this example
```
error_line
ok_line
error_line
```
