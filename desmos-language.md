# Desmos-Supported Programming Language — Draft Reference

---

## 1. Values and Types

| Type | Description |
|---|---|
| `Number` or `int` | A JavaScript-style numeric value; decimals are allowed. `int` is an alias, not a separate integer-only type. |
| `Point` | A 2D `(x, y)` or 3D `(x, y, z)` coordinate tuple. Points with imaginary coordinates produce `undefined`. |
| `List<T>` or `Array<T>` | An ordered, homogeneous list of at most 10,000 items. `T` cannot itself be a list. |
| `Polygon` | A 2D planar polygon containing at most 10,000 vertices. |
| `Color` | A color constructed with `rgb`, `hsv`, `okhsv`, `oklab`, or `oklch`. |
| `Any` | An unconstrained value, primarily for function parameters or intermediate results. Its use does not waive list homogeneity. |

### Convenience Type Names

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

`undefined` is an evaluation result rather than a distinct declarable type. Operations requiring a valid value propagate `undefined` unless specified otherwise.

---

### Numbers and Constants

The supported named constants are `\pi`, `\tau`, `\e`, and `\infty` (also spelled `\infinity`).

Numeric literals include integers and decimals, such as `2`, `-4`, and `0.25`. `\e` denotes Euler’s number; the single letter `e` is reserved and cannot be used as a variable name.

---

### Points, Polygons, and Colors

- **Points**: Defined as `(x, y)` for 2D or `(x, y, z)` for 3D. Coordinate access uses `.x`, `.y`, and `.z`.
- **Polygons**: Vertices can be retrieved via the `.vertices` property, which returns a `List<Point>` of 2D points. Polygons can be constructed using any of the following signatures:
  - Sequence of point arguments: `polygon((x_1, y_1), (x_2, y_2), ...)`
  - Single list of points: `polygon([(x_1, y_1), (x_2, y_2), ...])`
  - Two parallel coordinate lists: `polygon([x_1, x_2, ...], [y_1, y_2, ...])`
- **Colors**: Component ranges and conventions follow the target Desmos implementation for `rgb(...)`, `hsv(...)`, `okhsv(...)`, `oklab(...)`, and `oklch(...)`.

---

## 2. Names, Declarations, and Scope

### Variable Naming
- A variable name consists of **one letter** other than `x`, `y`, or `e`.
- A variable name may have a subscript of any length containing only letters and digits (e.g., `a_index1`).
# - **Greek letters are permitted as base variable names, but not as subscripts (Needs review: the complete list of permitted Greek letters needs to be supplied).**
- Full-word names like `variableName` are not valid variable names.
- Inline variables may contain letters without subscripts, but their first character cannot be a digit.

### Declarations
A variable can be declared with `name = expression` or `Type name = expression`. The `inline` modifier requests direct substitution at each call site:

```text
inline Number r = distance(p_1, p_2)
```

`inline` affects compilation, not value or scope; its expression must be valid at every site where it is substituted.

---

### Local Declarations (“With” Expressions)

A parenthesized, multiline block may declare local variables and conclude with a result expression:

```text
(
  a + b
  with a = 1, b = 2
)
```

- **No intra-scope dependencies**: Variables declared *inside* a local block cannot depend on sibling variables declared in that same local scope:
  ```text
  # Invalid: b depends on a declared in the same local scope
  (
    a + b
    with a = 1, b = a + 1
  )
  ```
- **Top-level dependencies**: Top-level declarations *can* depend on prior top-level declarations:
  ```text
  a = 1
  b = a + 1
  a + b
  ```
- **Enclosing scope dependencies**: A local variable may depend on variables from an enclosing top-level scope:
  ```text
  a = 1
  (
    a + b
    with b = a + 1
  )
  ```
- Local declaration scopes cannot be nested. Function parameters and loop iteration variables act as inputs to a local scope rather than declarations, meaning local expressions may reference function arguments or iteration variables directly.

---

### Execution Model and Fault Tolerance

Statements in a document are evaluated with independent error isolation. A runtime or evaluation error on one line does not halt or invalidate independent lines before or after it:

```text
error_line
ok_line     # Executes successfully despite surrounding errors
error_line
```

This line-level fault tolerance applies solely to separate top-level expression lines in the workbook environment. It does NOT extend to multi-line sub-expressions and local blocks, where an error causes the whole block to fail.

---

## 3. Expressions and Operators

### Numeric and Comparison Operations

| Syntax | Meaning |
|---|---|
| `a + b`, `a - b` | Addition, subtraction |
| `a * b`, `a \times b`, `a(b)` | Multiplication |
| `a / b`, `\frac{a}{b}` | Division |
| `a^b`, `a^{b}` | Exponentiation |
| `a!` | Factorial (numeric types only) |
| `a % b` | Remainder / modulo |
| `<`, `<=`, `>`, `>=` | Relational comparisons |
| `=`, `==` | Equality comparison (both forms are supported) |
| `and`, `or` | Logical conjunction and disjunction |

- **Logical NOT**: The `not` operator is **not supported**. Conditions must be expressed using inverted comparison operators.
- **Factorials**: The factorial operator `!` is strictly numeric; applying `!` to a `Point` or other non-numeric type produces an evaluation error.
- Standard mathematical operator precedence applies.

---

### Point Operations

- **Addition and Subtraction**: `p_1 + p_2` and `p_1 - p_2` perform coordinate-wise operations on points of matching dimension (2D with 2D, 3D with 3D).
- **Scalar Multiplication and Division**: `p * s` and `p / s` multiply or divide each coordinate by the scalar value `s`.
- **Cross Product (`\cross`)**:
  - `Point3D \cross Point3D`: Supported; computes the 3D vector cross product and yields a 3D `Point`.
  - `Point2D \cross Point2D`: **Not supported**; results in an evaluation error.
  - `Point2D \cross Number`: Supported; behaves identically to multiplication.
  - `Point3D \cross Number`: **Not supported**; results in an evaluation error.

---

### List Operations

Binary operations on lists are applied element-wise:

```text
[a, b] + [c, d]  # [a + c, b + d]
[a, b] * c       # [a * c, b * c]
```

- When operating on two lists of unequal length, the result is truncated to the length of the shorter list.

---

### Piecewise Expressions and Conditions

Piecewise expressions evaluate conditions from left to right:

```text
{condition_1: value_1, condition_2: value_2, default_value}
```

Equivalent block and ternary syntaxes:

```text
# Statement / Block style
if condition_1:
  value_1
else if condition_2:
  value_2
else:
  default_value

# Ternary styles
value_if_true if condition else value_if_false
condition ? value_if_true : value_if_false
```

- **Type Consistency Exception**: All branches must evaluate to the same data type, with one exception: **`undefined` is compatible with any type** and may appear as an alternative branch value.
- **Evaluation Semantics**: All branches in a piecewise expression are evaluated concurrently. Non-selected branches with runtime errors or undefined values are will invalidate the entire expression.

---

### Actions (State Updates)

Actions perform state mutations (such as ticker updates or click handlers) using either `->` or `\to`:

```text
variable -> expression
variable \to expression
```

Actions update the assigned state variable when an event or action trigger occurs.

---

## 4. Lists, Indexing, and Iteration

- **List Literals**: Defined as `[value_1, value_2, ...]`. Elements must be homogeneous; nested lists are forbidden.
- **Indexing**: 1-based (`list[1]` is the first element). Out-of-bounds indices evaluate to `undefined`.
- **Ranges**: `[a...b]` produces an inclusive range from integer `a` to `b`.

### List Comprehensions

List comprehensions produce a single-dimension list:

```text
L = i for i = [1...10]
```

Equivalent block syntax:

```text
L = (
  for i = [1...10]:
    i
)
```

- Variations: `for i = values`, `for i of values`, and `for i in values` are interchangeable. Parentheses around loop clauses are optional: `for (i of values)`.
- Typed iterators are supported: `for Point p in points`.
- Nested loops do **not** flatten and cannot produce lists of lists (which would violate type homogeneity rules).

---

## 5. Functions and Recursion

Functions are defined with an identifier, parameter list, and an expression body. The final evaluated expression serves as the return value:

```text
distanceSquared(p_1, p_2):
  (p_1.x - p_2.x)^2 + (p_1.y - p_2.y)^2
```

- **Recursion**: Recursive calls are permitted, with a maximum recursion depth limit of **10,000 calls**. Exceeding this limit returns `undefined`.
- **Local Scopes in Functions**: A function body may use a local declaration block (§2):

```text
midpoint(p_1, p_2):
  (
    (a, b)
    with a = (p_1.x + p_2.x) / 2,
    b = (p_1.y + p_2.y) / 2
  )
```

---

## 6. Built-in Functions and Standard Library

### Elementary, Exponential, and Logarithmic

| Function | Description |
|---|---|
| `exp(x)` | Exponential function ($e^x$) |
| `ln(x)` | Natural logarithm (base $e$) |
| `log(x)` | Common logarithm (base 10) |
| `loga(x)` / `log_a(x)` / `\log_{a}(x)` | Logarithm with base $a$ |
| `ceil(x)`, `floor(x)`, `round(x)` | Ceiling, floor, and round-to-nearest |
| `sign(x)` | Signum function (-1, 0, or 1) |
| `mod(a, b)` | Modulo / remainder |
| `gcd(a, b)`, `lcm(a, b)` | Greatest common divisor, least common multiple |
| `distance(p_1, p_2)` | Euclidean distance between two points |

---

### Trigonometric and Hyperbolic Functions

- **Direct Trigonometric**: `sin`, `cos`, `tan`, `csc`, `sec`, `cot`
- **Inverse Trigonometric**: `arcsin` (or `\sin^{-1}`), `arccos` (or `\cos^{-1}`), `arctan` (or `\tan^{-1}`), `arccsc` (or `\csc^{-1}`), `arcsec` (or `\sec^{-1}`), `arccot` (or `\cot^{-1}`)
- **Hyperbolic**: `sinh`, `cosh`, `tanh`, `csch`, `sech`, `coth`

---

### Calculus and Iterated Operators

| Syntax | Description |
|---|---|
| `d/dx f(x)` or `\frac{d}{dx} f(x)` | Derivative with respect to $x$ |
| `f'(x)` | Prime derivative notation |
| `\int` or `\integral` | Definite or indefinite integral |
| `\sum` or `\summation` | Iterated summation over an index range |
| `\prod` or `\product` | Iterated product over an index range |

---

### Discrete Mathematics and Combinatorics

| Function | Description |
|---|---|
| `nPr(n, r)` | Permutations: $\frac{n!}{(n - r)!}$ |
| `nCr(n, r)` | Combinations: $\frac{n!}{r!(n - r)!}$ |
| `a!` | Factorial of a non-negative integer |

---

### List Aggregations and Array Operations

- **Aggregations**: `count(L)`, `total(L)`, `mean(L)`, `median(L)`, `min(L)`, `max(L)`, `quartile(L, q)`, `quantile(L, p)`, `stdev(L)`, `stdevp(L)`, `var(L)`, `varp(L)`, `cov(L_1, L_2)`, `covp(L_1, L_2)`, `mad(L)`, `corr(L_1, L_2)`, `spearman(L_1, L_2)`, `stats(L)`
- **Transformations**: `repeat(item, count)`, `join(L_1, L_2, ...)`, `sort(L)`, `shuffle(L)`, `unique(L)`

---

### Probability Distributions and Simulation

- **Distributions**:
  - Continuous: `normaldist(...)`, `tdist(...)`, `chisqdist(...)`, `uniformdist(...)`
  - Discrete: `binomialdist(...)`, `poissondist(...)`, `geodist(...)`, `discretedist(...)`
- **Distribution Operators**:
  - `pdf(dist, x)`: Probability density / mass function evaluated at $x$
  - `cdf(dist, x)`: Cumulative distribution function evaluated at $x$
  - `inversecdf(dist, p)`: Quantile value for cumulative probability $p$
  - `random(dist, [n])`: Generates one or $n$ random samples from the distribution

---

### Hypothesis Testing and Statistical Inference

- **Test Constructors**: `ztest(...)`, `ttest(...)`, `zproptest(...)`, `chisqtest(...)`, `chisqgof(...)`
- **Statistical Results and Properties**:
  `null`, `p`, `pleft`, `pright`, `score`, `dof`, `stderr`, `conf`, `lower`, `upper`, `estimate`
# Needs review: Confirm whether these statistical fields are standalone accessor functions (e.g., `p(T)`) or properties accessed directly off the test object (e.g., `T.p`, `T.dof`).

---

### Geometry Transformations

The following geometry-specific transformation functions operate on points and polygons

- `translate(...)`
- `reflect(...)`
- `dilate(...)`
- `rotate(...)`

---

### Audio and Visualizations

- **Plots**: `histogram(...)`, `dotplot(...)`, `boxplot(...)`
- **Audio Generation**: `tone(frequency)` generates an audio tone at a specified frequency in Hertz.

---

## 7. Comments and Source Style

Comments can be formatted using shell, C-style line, or block delimiters:

```text
# Single line comment
// Alternative single line comment
/* Multi-line
   block comment */
```

## other things

syntax list[list<x]
regressions
tables + actions incompatability
no implicit `if x` where x is a number
syntax [0,2...100]
inline variables can reference other inline variables
recursive definition triggers an error, eg. a=b, b=a
variables violating containing more than one letter should show a warning, not an error
