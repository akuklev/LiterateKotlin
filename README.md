We have a dream of making Kotlin a programming language suitable for every purpose in any context. Unfortunately, Kotlin in its current form is poorly suited for literate programming and lags far behind Python when it comes to illustrating ideas in tutorials and research papers. In this memo, we draft a Kotlin flavor for literate programming and academic/educational use instead of ad hoc pseudocode.

When writing a computer science research paper or an educational tutorial, it's fine to spend days polishing code snippets for optimal readability, conciseness, and typographic perfection. Such applications value readability over writeability, expressiveness over simplicity, principled considerations over practical concerns, and the avoidance of boilerplate and visual clutter at almost any cost. This seems to contradict one of the cornerstones of Kotlin: a remarkable balance between readability and writeability, expressiveness and simplicity, orderliness and pragmatism, innovation and conservatism. But it turns out that the necessary changes, while fairly radical, are limited to syntax and default behavior. Literate Kotlin, the flavor of Kotlin presented in this memo, can be seen as an alternative interface to the same underlying language.

The first part of the memo is devoted to syntax and appearance. The second part suggests some adjustments to the default behavior. In the third part, we discuss desirable extensions that we believe will also benefit Kotlin itself in the long run.

# What's literate programming anyway?
In 1984, Donald Knuth introduced literate programming, a practice of working not just on the source code but on a well-written and well-structured expository paper from which the source code can be extracted. The ultimate result should be the expository paper, which carefully walks through all the nooks and crannies of the source code, explaining the ideas and documenting the reasoning behind certain decisions. It is both an essay interspersed with code snippets and a source code interleaved by accompanying text: code and text are equally important.

Existing programming languages treat accompanying text as a second-class citizen, as “comments” bashfully fenced with freakish digraphs like `/* … */`. Markup languages used for writing computer sciense research papers (mainly (La)TeX) and tutorials (mainly HTML and Markdown) take the opposite approach, treating code snippets as second-class citizens. We propose a balanced approach treating treating code and text on par. Before we can present it, we need to explain our treatment of blocks and literals.

# Part I: Syntax and appearence

## Basic structure: blocks, literals, comments

### Multiline blocks

We propose to restrict utilising braces for inline blocks only, and use off-side rule for multiline blocks. Indentation-based structure sticks out above everything else, so it should take precedence over comments, quoted literals and brackets. **This massively speeds up incremental parsing: blocks can be recognised instantly without prior parsing and processed independently.** 

We propose to fix block indentation to two whitespaces once and for all, any other indent (1 or >2) continues the previous line:

```Kotlin
fun example(files : List<File>,
            target : File)
  ...
  return something + somethingElse + somethingOther +
   yetSomething + rest
```

IDEs should provide visual reading aid in case of consequent dedents by displaying end marks (`■`).

```Kotlin
fun main(args : List<String>)
  for (arg in args)
    println(arg)
  ■
```

At the end of large indentation regions, labeled end marks (e.g. `■ main`) should be used.

### Unquoted literals
In Kotlin, trailing functional arguments enjoy special syntax: `a.map({ println(it) })` is simply `a.map { println(it) }`.
Trailing textual arguments deserve special syntax too. Unquoted literals are opened by `~` (with no whitespace before and a whitespace or an indent after) and closed by the next line or dedent. Line breaks can be `\`-escaped, `\{...}`-syntax used for type-based JSR 430-like safe interpolation.

```Kotlin
fun greet(name : String)
  println~ Hello, \{name}!
```

Those would also work nicely with key-value lists:

```Kotlin
address: Address
  house:~
    Olaf Taanensen 
    Tordenskiolds 24
  city:~ Oslo
```


### Instead of comments: treating code and text equally

Our proposal from the first section implies mandatory indentation for all non-inline blocks. Thus, all remaining unindented lines are top-level definitions (`class …`, `object …`, …) and directives (`package …`, `import …`). These necessarily begin with an annotation or a keyword. Annotations readily begin with an `@`, and it won't be too much pain to prepend `@` to top-level keywords: `@import` already looks familiar from CSS, `@data class` and `@sealed class` make perfect sense anyway: most modifier keywords are nothing but inbuilt annotations.

This way, every code line either starts with an `@`, or is an indented line following a code line (with possibly one or more blank lines in between). Let us require the compiler to skim all the lines that do not meet this specification. These other lines now can be used for accompanying text written “as is” without fencing. We suggest using (La)TeX hybrid-mode Markdown (`\usepackage[hybrid]{markdown}`): it has excellent readability while providing the whole power of (La)TeX, the de facto standard for writing technical and scientific papers.

Freely interleaving the code and accompanying text, without fencing either, is the perfect fit for literate programming. The very same file can either be fed into a Kotlin compiler to produce a binary or into a Markdown/TeX processor to produce a paper.

Sometimes it is still desirable to comment a single line. Since at least 1958, em-dashes ` — ` surrounded by whitespaces were used for single-line comments to separate code and text. It seems to be a typographically perfect solution, but PC standard keyboard layout lacks em-dash. Ada, SQL, Eiffel, Elm, Haskell, Lua, SQL and several other languages use double dash `--` as an ASCII substitute for
em-dashes, but this is incompatible with C-style decrement operator. We propose to use either the real em-dash (which is present in the standard MacOS keyboard layout and can be accessed via Compose+`---` on Linux) or a single backtick surrounded by whitespaces.

## Plain text notebooks

Jypiter style notebooks can be seen as an interactive form of literate programming. The expository paper can (and should) contain runnable code samples to illustrate usages of the code being explained and test cases for each non-trivial function. These should be optimally displayed as runnable, editable, debbugable blocks with rich (visual, animated, interactive) output, that's what notebooks are build from. Since we see such blocks as an element of literate programming, we want to provide plain-text syntax for them:
```Kotlin
@run sampleFunction(1, 3)

@run 1 + 2 + 3
@expect 6

@run `Named sample`:
  val a = 1 + 2
  a + 3

@run(collapsed: true, autoexec: false)
  someLenghtyComputation()
```

## Advanced typography 

## Compliance with mathematical notation
To be compliant with standard mathematical notation, we should display the multiplication operator as `·`, comparison operators as `≤`, `≥`, `=`, `≠`, logical operators as `¬`, `∧`, `∨`, and use `↦` in lambda-expressions, e.g. `{x ↦ x + 1}` . 

As it is customary in mathematics to have whitespaces around relation symbols and symbolic symmetric binary operators such as `+` (except for `a·b`, `a..b`, and `a..<b`), we require that for all such operators including the typing relation as in `n : Int`. We want to allow `//` and `#` both as infix operators and as end-of-line comment markers. It is indeed possible if we require end-of-line comments to be separated from the preceding text by at least two whitespaces or to start directly at the indentation level of the previous line.

The assignment operator should be then displayed as `≔` when introducing a fresh name
(e.g. `val a ≔ 5`) and for default argument values, or by an immediate colon `key: value` for named arguments and other “key-value” cases.

In mathematics and functional programming, it's fairly common to use the right pointing black triangle for inverse application, i.e. `x ▸ f ≔ f(x)`. Thus we propose to display `x.let f` as `x ▸ f` and `x?.let f` as `x?▸ f`.

In Kotlin, the method invocation `method(args)` is a complex syntactic entity, supporting optional arguments, named arguments, and variable number of tail arguments, as well as special handling for the last argument if it is of the function type. As we proposed in the second section, special handling could be also introduced for strings and string templates (values of the type `AdditionalContext.()-> String`). Further, one could introduce positional-only and keyword-only arguments and \*\*kwargs as in Python. In method invocations, parentheses can be omitted in some cases (while invocation is implied!), so methods as entities cannot be referred to by their name, the notation `::method` (`class::method` in fully qualified case) is used instead. Application of functions (values of the type `(args)-> R`) mimics method invocation, yet with several intransparent limitations: parentheses are mandatory, sometimes manual `.invoke()` has to be used.

Mimicking method invocation does not comply with the usual mathematical practice, where it is customary to write `sin x` instead of `sin(x)` and `f a b` for `( f(a) )(b)`. We propose to use `import FunctionalNotation` to introduce a new type `X -> Y` (without parens around `X`) to introduce functions like `sin` that can be used as customary in mathematics and functional programming languages. Additionally, we propose `import CoefficientNotation` to interpret `2x` for `2·x`, and `import SegmentsNotation` to interpret sequences of uppercase letters with optional indices (`AB`, `ABC`, `ABCD`, `X1X2`) as `Segments(A, B)`, `Segments(A, B, C)`, `Segments(A, B, C, D)`, `Segments(X1, X1)`. In the latter case uppercase identifiers are available with backticks (`` `ABC` ``).

In Kotlin, operators are always referred to by their verbatim name, like `minus`, `unaryMinus`, and `dec`. We propose to allow an alternative notation: operator symbols enclosed into parentheses with no whitespaces around for infix operators, whitespace before for postfix ones and whitespace after for prefix ones. Thus, one can use `::(-)` for `::minus`, `::(- )` for `::unaryMinus`, and `::( --)` for `::dec`rement.

The other way around, any binary (or vararg) function should be allowed to be used as an infix operator by surrounding it by chevron quotation marks, e. g. `a ‹and› b` , `2 ‹Nat.plus› 3`.

## Avoiding redundant type annotations
As in Coq, Agda, and Lean, variables with certain names can be declared to have default types

```Kotlin
variables n : Int, z : Point
... now variables n, n1,… will have default type Int; z, z1,… default type Point
```

and two or more consecutive variables of the same type can be separated by whitespaces:

```Kotlin
fun plus(x y : Int) : Int
```

## Let blocks
We suggest introducing let-blocks. Let-block header contains a list of vals being defined, the following block contains a list of conditions those have to satisfy.  

```
let x y : Float
  x + 2y = 5
  x - y = 4
```

A let-block compiles if there is a compiler solver-plugin that supports given condition forms and succeeds iff there is a unique or a preferred solution.

We envision at least two solvers: Linear solver precisely as in Knuth's METAPOST (in particular, solves the example above) and, in the distant future, a deep unification solver as defined in [The Verse Calculus paper](https://simon.peytonjones.org/assets/pdfs/verse-icfp23.pdf) by Simon Peyton Jones, Guy Steele et al., that possesses enormous expressive power, elegantly subsuming both Prolog and Datalog.

## Dual naming: verbose names and concise names
Naming things is hard both in programming and in mathematics. Objects and operations should have readable and self-explanatory names. However, verbose names may severely impair readability in formulas. Compare the following three variants of the same formula:
- `n·(n + 1) / 2`,
- `elementCount * (elementCount + 1) / 2`, and
- `div(times(elementCount, plus(elementCount, 1)), 2)`

Dual naming `` `verbose name`conciseName `` is a possible way to reconcile contradictory requirements.

```Kotlin
val `element count`n = ...
val (`height`x, `width`y) = o.getDimensions()
...

class List<`element type`T>
...
```

## Unicode names and custom operators 
It should be allowed to use non-ASCII characters and custom operators as `conciseName`s. Readable `verbose name` is strictly necessary (so one knows how to read those symbols aloud) and ASCII-only if `conciseName` contains characters not available on a standard keyboard.

```Kotlin
enum class `Boolean`𝔹 {`true`, `false`}

data class `Pair`(×)<out X, out Y>(val first : X, val second : Y)

val `factorial`( !) = fun(n : ℕ) {
  when(n) {0 -> 1; p⁺ -> n · p!}
}

val `conjugate`(+ ) = {c : ℂ -> Complex(c.re, -c.im)}
```

With those definitions, one can use `𝔹` for `Boolean`, `X × Y` for `Pair<X, Y>`, `n!` for `factorial(n)`, `+c` for `conjugate(c)` and  `⌊0.6⌋` for `floor(0.6)`.

The other way round, the verbose name can be a “closed operator”:

```Kotlin
fun<T> `if $c then $a else $b`ifelse(a b : T, c : 𝔹) : T

fun `⌊$x⌋`floor(x : Float)
```

#### Tightness
Expressions like `+n!` can be parsed both as `( +n )!` and `+( n! )`. With definitions as above, it is not a valid expression, it's a `syntax error: ambiguous expression`. However, one can specify tightness for operators. If `( !)` binds tighter than `(+ )`, `+n!` resolves into `+(n!)` and the other way around.

Infix operators may have different right and left tightness. For example, `(-)` binds tighter on the right than on the left: `a - b - c` resolves into `(a - b) - c`.

Tightness strengths must form a poset (actually, even less: a DAG), but to not form a set: they do not have to be pairwise comparable. We introduce abstract labels called Operator Categories and declare them to be tighter or weaker than some other labels.

Actually, an `OperatorCategory` is a bit more than a label: it specifies how to deal with respective homogeneous operator chains. For example, there is a large operator category `EqRel` that contains all comparison operators and resolves chains of the form `a < b < c`  into`a < b ‹and› b < c` .

#### Inner parameters
Operators may have inner parameters, e.g. the indexed access operator `arr[i]` is a postfix operator with an inner parameter `( [$idx])` . In mathematics, many binary operators, including tensor product and semidirect product, have optional parameters rendered as subscripts or superscripts.

Using parser techniques developed for the Agda programming language, we can embrace this complexity without any considerable problems.

By combining custom `OperatorCategory` and operators with inner parameters, one can even embrace the notorious example of insane operator complexity: the METAPOST path notation:
```Kotlin
draw a -- b -- c --cycle                  // A triangle, (--)-lines are straight
draw a ~~ b ~~ c ~~cycle                  // A circle through a, b, and c, (~~)-lines are curved
draw a ~~ b ~~ c ~- d -- e --cycle        // (~-) lines connect smoothly on the left side only
draw a ~~ b ~~[tension: 1.5, 1]~~ c ~~ d
draw a [curl: k]~~ c ~~[curl: k] d
draw a ~~ b [up]~~ c [left]~~ d ~~ e.
draw (0,0) ~~[controls: (26.8,-1.8), (51.4,14.6)]~~
 (60,40) ~~[controls: (67.1,61.0), (59.8,84.6)]~~ (30,50)
```

# Suggestion
# Disambiguating methods and properties
In Kotlin `obj.name(...)` can mean invocation of the method `name` of `obj` or extraction
of its property `name` of a callable type and its application. One cannot find out which one is used without looking up the definition of the class of `obj`. To resolve this ambiguity and to improve typographical properties of the code, we suggest displaying dots as `▸` in case of method calls following the long tradition of using arrows for method calls started by PL/I in the late 60s:
```Kotlin
files ▸dropLast(n) ▸withIndex ▸last  
```

```Kotlin
fun example(files : List<File>,
            target : File)
  files ▸filter
    it.size > 0 &&
    it.type = "image/png"
  ▸map { it.name }
  ▸withIndex ▸map fun(idx, item) 
    ...
  ...
```

# Part II: Semantic considerations
While being very radical, all of the above suggestions are merely syntactic surgar, they are almost exclusively limited to parser and IDE. Yet they are not quite enough to make Kotlin appealing as a substution for “pseudocode”.

## Pythonic integers
Academic pseudocode assumes the default integer type to be overflow-free as in Python. The operator `/` is always used as the true division operator even when both operands are integer. For integer division, an additional operator `//` should be introduced.

## Operator attribution and type classes
In accordance with their mathematical semantics, expressions like `1 + 1` should interpreted as `Int.plus(1, 1)` rather than `1.plus(1)`, i.e. arithmetical operators are considered to be properties belonging to companion objects of the given numeric type rather than methods of number objects themselves.

Since we mentioned companion objects containing operators like “plus”, we should also mention that the notion of type-classes is indispensable in many academic contexts. In Kotlin, one can define
both nested classes and extension functions. The type-classes are, in a sense, extension nested data classes with quite a bit of additional syntactic sugar.

Consider the following definition of a monoid-structure on a type `T`:
```Kotlin
data class <T>.Monoid(val compose : (vararg xs : T)-> T)
  val unit ≔ compose()    — Unit is the nullary composition

  contracts {
    unit ‹compose› x = x
    x ‹compose› unit = x
    x ‹compose› y ‹compose› z = x ‹compose› (y ‹compose› z)
    compose(x, *xs) = x ‹compose› compose(*xs)
  }
```

With such a definition, we now can write functions like this:
```Kotlin
fun<T : Monoid> square(x : T)
  x ‹T.compose› x
    
or, equivalently
    
fun<T : Monoid<::(∘)>> square(x : T)
  x ∘ x

Here,  generics resolution implicitly put both `T` and `(∘)` into the scope.
```

Support for higher-kinded type classes and proper inheritance for them can be directly imported from Arend. Eventually, one should be carefully introducing full-blown dependent types, following the defensive approach to dependent types pioneered in Haskell.

Amusingly, adding dependent types to Kotlin immediately allows embedding SQL-type queries almost verbatim:
```Kotlin
fun Table.select(cols : this.colsCtx.()-> List<t.Col>) : LazyTable
fun LazyTable.where(clause : this.ctx.()-> 𝔹) : LazyTable

users ▸select {name, age, address as "userAddress"} 
      ▸where {age > 18}
```

## Runtime-introspectable coroutines
We suggest using labeled blocks (e.g. `name@ { code }` in coroutines as runtime-introspectable execution states. If the coroutine job `j` is currently running inside of the labled block `EstablishingConnection@`, we want `(j.state is EstablishingConnection)` to be true. The hierarchy of nested blocks in the coroutine should autogenerate a corresponding interface hierarchy.

Furthermore, propose coroutines to have exposable read-only data-only properties that can be used to track the coroutine progress. We suggest allowing visibility modifiers `public` and `internal` for top-level `var`s and `val`s as well as the ones in labeled blocks and labeled loops:
```kotlin
val j = launch
  ...prepare data
  Moving@ for (i in files.indices)
    public val progress = i / files.size
    fs.move(...)
  ...finalize
 
val u = launch
  ...
  when (val s = j.state)
    Moving => println~ Moving files, \{s.progress · 100}% complete
  ...
```

To avoid misalignment, `j.state` must read out all properties immediately when invoked, all properties must be data-only, i.e. either of primitive data type, or hereditarily immutable data classes.

## Strong object typing
Eventually, structured concurrency should be generalized to structured ownership, with a general notion of managed object and managing scopes. Kotlinesque coroutine scopes and Rustacean lifetimes are managing scopes, jobs and shared mutable variables are respective managed objects, governed by separation logic. Redistributable references to managed objects can be faithfully treated as values, types of which are path-dependent (in Scala sense) on their respective managing scopes (cs.Job, lt.Var). Thus, to handle them, it would suffice to support full-blown PDTs and allow passing objects (coroutine scopes, lifetimes, etc.) not only as arguments, but alternatively as parameters, e.g. `fun <cs : CoroutineScope> example(v : cs.MutRef<Int>)`.

Besides managed objects, there are exclusively-owned objects (cf. uniqueness typing). References to those cannot be copied or passed arbitrarily, so they must be marked syntactically as being non-values. When a method gets them as arguments, the respective arguments must be annotated either `my obj` or `borrow obj` in case the object is returned back to the call site after completion. A local “variable” containing an exclusively-owned object should be declared `my obj` instead of `val obj`, e.g. `my job = lunch someCoroutine(…)` or `my o = object : SomeInterface {…}`. Exclusively-owned objects most frequently appear as receivers (`this`). Owing to smart casts, strong typing for exclusively-owned objects can be piggybacked on the existing Kotlin type system by extending the syntax and semantics for interfaces. The resulting type system fragment would closely reassemble the system by F. Pfennig and A. Das from “[Verified Linear Session-Typed Concurrent Programming](https://www.cs.cmu.edu/~fp/papers/ppdp20.pdf)”, see also <https://www.cs.cmu.edu/~fp/papers/lmcs22a.pdf> for a primer on possible concise syntax.

The third possibility is the external/standalone objects (resources), such as filesystem and database: those are properly handled by a capability system like that in Scala 3.
