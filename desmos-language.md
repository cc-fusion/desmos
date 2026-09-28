# Desmos-supported programming language — draft reference

This document treats the rules in the prompt as requirements. A rule marked **(assumption)** fills a gap or resolves an ambiguity; it is not yet part of the confirmed specification.

The language is Desmos-like, but the example suggests a source language with features beyond native Desmos expressions. **(assumption)** A compiler or translator lowers this language into expressions that its Desmos target can evaluate. Consequently, a construct being valid *source syntax* does not necessarily mean it can be pasted into Desmos unchanged.

## 1. Values and types

| Type | Description |
|---|---|
| `Number` or `int` | A JavaScript-style numeric value; decimals are allowed. **(assumption)** `int` is an alias, not a separate integer-only type. |
| `Point` | A 2D or 3D point. Points with imaginary coordinates produce an undefined value. |
| `List<T>` or `Array<T>` | An ordered, homogeneous list of at most 10,000 items. `T` cannot itself be a list. |
| `Polygon` | A polygon containing at most 10,000 items. Its vertices must be 2D points. |
| `Color` | A color constructed with `rgb`, `hsv`, `okhsv`, `oklab`, or `oklch`. |
| `Any` | **(assumption)** An unconstrained value, primarily for function parameters or intermediate results. Its use does not waive list homogeneity. |

Convenience type names:

| Name | Meaning |
|---|---|
| `List` or `ListNumber` | `List<Number>` |
| `ListPoint` | `List<Point>` |
| `ListColor` | `List<Color>` |
| `ListPolygon` | `List<Polygon>` |
| `ListAny` | **(assumption)** A list whose element type is inferred when possible; it is *not* permission to mix element types. |

A declaration may omit its type:

```text
Number a = 3
List<Point> p = [(0, 0), (1, 0)]
c = rgb(255, 0, 0)
```

**(assumption)** `undefined` is an evaluation result rather than a distinct type that can be declared. Operations requiring a valid value propagate undefined unless their documentation says otherwise.

### Numbers and constants

The supported named constants are `\pi`, `\tau`, `\e`, and `\infty` (also spelled `\infinity`).

**(assumption)** Numeric literals include integers and decimals, such as `2`, `-4`, and `0.25`. `\e` denotes Euler’s number; it does not make the reserved letter `e` available as a variable name.

### Points, polygons, and colors

**(assumption)** Points use `(x, y)` or `(x, y, z)`; coordinate access uses `.x`, `.y`, and, for 3D points, `.z`. Polygons expose an ordered `.vertices` list of 2D points. The exact polygon-construction syntax remains unspecified.

Color constructors are supported, but their component ranges have not been specified. **(assumption)** Use the target Desmos implementation’s ranges and conventions for `rgb`, `hsv`, `okhsv`, `oklab`, and `oklch`.

## 2. Names, declarations, and scope

A variable name consists of **one letter** other than `x`, `y`, or `e`. It may have a subscript of any length containing only letters and digits. The listed Greek letters are also permitted as variable names, but not as subscripts; the list of permitted Greek letters still needs to be supplied.

**(assumption)** In plain-text source, `a_index1` represents a subscripted name. Thus `a`, `b`, and `a_index1` are valid names, while `totalForce` is not a variable name. Function-name rules are separate and have not been specified.

A variable can be declared with `name = expression` or `Type name = expression`. The `inline` modifier requests direct substitution at each use:

```text
inline Number r = distance(p_1, p_2)
```

**(assumption)** `inline` affects compilation, not the value or scope of `r`; its expression must still be valid everywhere it is substituted.

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

**(assumption)** Function parameters and iteration variables are inputs to a local scope, rather than local declarations for this dependency rule. This permits a local definition to use its function parameter or loop iterator.

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
| `a % b` | **(assumption)** Remainder, included because the example uses `%` |

**(assumption)** Comparisons use `<`, `<=`, `>`, `>=`, `=`, and `!=`; logical conditions use `and`, `or`, and `not`. Assignment `=` is distinguished from comparison `=` by context. If that ambiguity is undesirable, adopt `==` for comparison instead.

**(assumption)** Standard mathematical precedence applies: parentheses; factorial; powers; unary signs; multiplication/division/remainder; addition/subtraction; comparisons; logical operators. Parenthesize expressions whose interpretation matters.

### Point operations

The example uses point subtraction, point addition, multiplication by a number, and summation of point values:

```text
p_1 - p_2
p_1 + p_2
p_1 * 0.5
```

**(assumption)** These are coordinate-wise operations on points of matching dimension. Division of a point by a nonzero number is also coordinate-wise. Other point operations, including `\cross`, require an explicit definition before use.

### List operations

For a binary operator supported by the element type, a list operation is element-wise:

```text
[a, b] + [c, d]  # [a+c, b+d]
[a, b] * c       # [a*c, b*c]
```

**(assumption)** Two lists must have equal lengths. A scalar is broadcast across a list. Broadcasting does not create nested lists, and an unsupported element-level operation remains invalid.

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

**(assumption)** Conditions yield truth values usable by piecewise expressions; boolean values are not otherwise a declared data type. The same branch-type rule applies to all three forms.

## 4. Lists, indexing, and iteration

A list literal is `[value_1, value_2, ...]`. Every element must have the same type, and lists cannot contain lists.

**(assumption)** Indexing uses `list[index]`, starts at **1**, and returns undefined for an invalid or out-of-range index. The example’s `[1...count(pos)]` is the reason for choosing 1-based indexing.

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

**(assumption)** `[a...b]` is an inclusive integer range, and iterations preserve source order. A typed iterator can be written `for Point p in points`. Each loop body evaluates to one list element.

Nested loops occur in the example:

```text
[
  for i in [1...3]:
    for j in [1...2]:
      i + j
]
```

**(assumption)** Nested loops **flatten** into one list in outer-loop-major order. They do not produce a list of lists, which would violate the data-type restriction. Every resulting element must have the same type, and the final list must have at most 10,000 elements.

**(assumption)** `count(list)` returns its number of items; `total(list)` sums numeric elements or adds point elements coordinate-wise. Other list functions—including `rangeExcept` and `closestVerticesIdx` in the example—must be defined by the program or separately documented as built-ins.

## 5. Functions and recursion

Functions are supported, and recursion is limited to 10,000 calls.

**(assumption)** Function definitions use a name, parameters, and an expression-valued body:

```text
distanceSquared(p_1, p_2):
  (p_1.x - p_2.x)^2 + (p_1.y - p_2.y)^2
```

The last expression is the result; no `return` keyword is required. Parameter and result type annotations are optional **(assumption)**.

A function body may use local declarations subject to the scope rules in §2:

```text
midpoint(p_1, p_2):
  (
    a = (p_1.x + p_2.x) / 2
    b = (p_1.y + p_2.y) / 2
    (a, b)
  )
```

Here neither `a` nor `b` depends on the other. **(assumption)** Exceeding the recursion limit evaluates to undefined. A precise rule is still needed for whether the limit counts stack depth or total calls.

## 6. Parametric equations

Parametric equations are supported, but their original syntax was not provided. One possible syntax is:

```text
(x(t), y(t)) for t = [0, 2\pi]
```

**(assumption)** A parametric expression produces a 2D point as a parameter varies over a specified domain; 3D parametric expressions may use `(x(t), y(t), z(t))` if supported by the target. The notation above is illustrative rather than confirmed grammar. It also requires an explicit decision about whether `x`, `y`, and `t` have special roles, given the variable-name restrictions.

## 7. Blocks, comments, and source style

Comments may use `#`, `//`, or `/* ... */`:

```text
# One line
// Another line
/* Multiple
   lines */
```

A file may use indentation-based or bracket-based block syntax, but it must use one style consistently. **(assumption)** Parentheses used solely to group an expression or to create the documented local-declaration expression do not count as switching block styles.

The exact delimiters for *bracket-based statement blocks* have not been specified. **(assumption)** Until they are, use indentation for functions, conditionals, and loops, while retaining ordinary parentheses, square brackets, and braces for expressions, lists, and piecewise expressions respectively.

## 8. Geometry example: intended meaning versus valid code

The supplied collision example illustrates the intended workflow—iterate over polygons, inspect vertices, find a nearby edge, and compute forces—but is not yet a valid reference program. In particular:

- `getBoundingBox(pos)` inside `for Polygon currPos in pos` probably means `getBoundingBox(currPos)`.
- `totalForce`, `currPt`, `segmentPt`, and `distanceRatio` are not valid variable names under the one-letter rule. Give them one-letter names, optionally with subscripts.
- The proposed `getData` result, `[segmentPt, (ratio, 0)]`, **is** a homogeneous `List<Point>` if both points are 2D. Its second element’s `.x` component can carry the ratio. **(assumption)** This point-as-record encoding is intentional; a dedicated record or tuple type does not currently exist.
- `getDistanceRatio` and `getData` appear to refer to the same operation but have different names.
- `total(...)` must receive a homogeneous list. If each collision contribution is a 2D point, the no-collision contribution should also be `(0, 0)`, not `0`.
- `vel[i] - totalForce` computes a value; it does not, by itself, update `vel[i]`. **(assumption)** State changes are expressed by defining a new list of velocities, rather than mutating list elements.
- `pos.vertices` is invalid if `pos` is a `List<Polygon>`; use the current polygon, such as `pos[i].vertices`.
- `bodyIndex` in the nested-loop sketch has no defined source.
- `polygon.vertices[index]`, `%`, and the 1-based indexing rule need to agree at the final vertex: `(index % count(vertices)) + 1` wraps a 1-based index to its successor; `(index + 1) % count(vertices)` does not.

Names such as `collisionAABB`, `ptInPolygon`, `closestPointOnSegment`, `safeDivide`, and `getBoundingBox` have not been established as language built-ins. **(assumption)** Treat them as user-defined functions until a built-in library is specified.

## 9. Decisions still needed

This draft deliberately does not invent definitive answers for the following:

1. Which Greek letters are legal variable names, and how their names are written in source.
2. Polygon-construction syntax and whether `.vertices` is writable.
3. The exact permitted operand/type combinations for `\cross`, factorial, and point operations.
4. Whether numeric equality is `=`, `==`, or both, and the complete comparison/logical syntax.
5. The exception(s) to the same-type piecewise-branch rule.
6. The exact grammar and semantics of parametric equations.
7. Bracket-based *statement block* syntax.
8. Which geometry, list, and color functions are built-ins and their signatures.
9. Whether list indices and ranges are definitively 1-based and inclusive.
10. Whether the language has mutable state, or whether simulations must define a new state from the old state.
