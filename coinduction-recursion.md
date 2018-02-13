# Coinduction-Corecursion
## Basic inline syntax
```
f(x)              # application

(a, b)            # pair formation

\x ↦ t(x)         # raw lambda expression

(\x : T) ↦ t(x)   # annotated lambda expression

(\x : A) ⟶ B(x)  # dependent function type

(\x : A) × B(x)   # dependent pair type

# Examples of inline pattern matching:
(\n, \m) ↦ n + m                # Match a pair

(tt ↦ ff; ff ↦ tt)              # Negate a boolean

(Zero ↦ ff; Succ(\n) ↦ tt)      # False on zero, true on all others

(Zero ↦ ff; Succ(\n) ↦ neg(.n)) # False on evens, true on odds

# Same type variables shorthand:
(\x \y : T)   ≡   (\x : T, \y : T)
```
Backslash is the mandatory freshness sigil used each time a new variable is introduced.

Use of freshness sigils in particular allows to shorten signatures like
```
interlace[\T : ∗](\a \b : Stream[T]) : Stream[T]
# For an (automatically inferred if possible) type `T` and two `Stream[T]`s produces a `Stream[T]`
```
to
```
interlace(\a \b : Stream[\T]) : Stream[T]
# Takes two `Stream`s of some `T` and produces a `Stream` of the same `T`
```

## Block syntax for dependent records (and more general coinductive types)
We propose the keyword `@Structure` for block definition of dependent records for two reasons: 
– simple records are known as structures in the C programming language and its numerous derivatives;
– dependent records with type members encode spaces equipped with extra structure, examples being a monoid, a group, a topological space etc.

Examples:
```
@Structure CartesianPoint:
  x : Real
  y : Real

@Structure PointedType:
  T : ∗
  p : T
```

The definition of such a type is a list of declarations of its methods having the syntax `name : type` (whitespaces around colon are mandatory). As seen in the second example, member signatures may depend on previously declared members. We'll also allow recursion generalizing dependent records to so called coinductive types. The signature
of eliminators will be allowed include the type that is being defined (yet only
strictly-positively), which enables definitions of infinitary
structures while using only finite number of eliminators:
```
@Structure NatStream:
  head : Nat
  tail : NatStream
```

We'll also allow to define structures with parameters:
```
@Structure Pair[\A \B : ∗]:
  fst : A
  snd : B
  
@Structure Stream[\T : ∗]:
  head : T
  tail : Stream[T]
```
Use of square brackets for parameters suggest they can omited if their values can be inferred from the usage context. Usual parentheses should be used when optionality is not desired or possible due to non-uniform use of the parameter, e.g. one performs case analysis on its value:
```
@Structure NumberGenerator(negativeNumbersAllowed : Bool):
  nextNumber : negativeNumbersAllowed ▶ (tt ↦ Int; ff ↦ Nat)
```

Elements of coinductive types can be defined by means of
corecursion, i.e. by providing for the object we define
a scheme how to compute each of eliminators on it, e.g.
```
@def p: Pair[Nat, Nat]
  fst: 1
  snd: 2
```
Corecursion allows using already defined eliminators for defining
next ones, thus, well-founded corecursion enables very rich usage,
@see [http://www.types2016.uns.ac.rs/images/abstracts/setzer2.pdf].

Eliminating has the syntax `p.fst`, definitions make use of colon
_without_ preceding whitespace.

## Π-Types, Pattern Matching and λ-expressions
Above we considered only coinductive types with finite number of
eliminators, but there might be an infinite family of eliminators,
which is facilitated by allowing eliminators to be parametrized. 
Function types `(X ⟶ Y)` and П-types in general are very simple
examples of coinductive types, they just have a family of
eliminators indexed by `X` and returning `Y`:
```
@Structure Func(\X \Y : ∗):
  apply(\x : X) : Y

@Structure DepFunc(\X : ∗, \Y : Func(X, ∗)):
  apply(\x : X) : Y.apply(x)
```

We'll allow omiting eliminator name altogether for this case to obtain
natural syntax for function types, lambda expressions and definitions
by pattern matching:
```
@Structure @infix (\X : Type)⟶(\Y : Type):
  (\x : X) : Y
  
@def square: Nat ⟶ Nat
  (\n : Nat): n · n
  
@def factorial: Nat ⟶ Nat
  Zero: Succ(Zero)
  Succ(\n : Nat): Succ(n) · factorial(n)
```

## Indexed Coinductive Types Include VDTs (Jason Hickey’s "Very Dependent Types")

The canonical example of a very dependent type is the infinite generalization of the dependent pair — the stream of the form `(a : A, b : B(a), c : C(b), ...)`. Indexed (dependent) coinductive types facilitate definitions of both a type signature of such a stream and of a stream for a given signature. Recall that the constructor Sigma of the dependent pair type requires an closure of the form `(\a : FirstType) ↦ SecondType`. The constructor for the TypeTower will be a "closure" of the form `(\v : HeadType) ↦ NextType, "tail"`. What’s the type of the "tail"? It should be a box, containing the "closure" of the same form for the NextType in the role of the HeadType. Let’s write it down:
```
@Structure TypeTower[HeadType : *]:
  peek(\v : HeadType) : (NextType : *, Ts : TypeTower[NextType])
```

Now the counterpart of the sigma constructor, turning the signature into the type of this signature. In the case of Sigma, we take the closure `(\a : FirstType) ↦ SecondType` and make a dependent record `fst : FirstType, snd : SecondType(.fst)` from it. The same strategy leads to the following definition:
```
@Structure DepStream[HeadType : *, Ts : TypeTower[HeadType]]:
  head : HeadType
  tail : DepStream[Ts.peek(head)]
```

We defined a Very-Dependent Stream, a VD-counterpart of a dependent function `(\n : Nat) ⟶ T(n)`. We can do the same not only for `Nat`, but for every inductive type. In the definition of the `DepTower`, the HeadType corresponds to then `Zero` constructor of `Nat` and `peek` to the `Succ`. Let’s write it down this way:
```
@Structure Nat-Tower:
  zeroT : *
  succT(\v : zeroT) : Nat-Tower

@Structure Nat-VDT[T : Nat-Tower]:
  zeroV : T.zeroT
  succV : Nat-VDT[T.succT(zeroV)]
```

Let’s play the same game for a binary tree and then formalize the strategy for a generic indexed container.

(TODO)

It is worth mentioning, indexed dependent coinductive types can be defined impredicatively. The types we presented require existence of a universe for such encodings since we use `*` not only as a range for quantification, but also as a value type. It is also possible that these types can be described inductively using a form of induction-recursion: instead of describing them coinductively via their destructors, we might describe their possible corecursors, which is possible with induction-recursion for a number of nontrivial cases.

Now let me tell you the main motivation for this story: using partially-instantiates coinductive types (the dual to higher inductive types), we get enough power to define weak categories (quasicategories) and weak ω-categories, or more generally
presheaves over inductively presented categories, the simplest example being semisimplicial types.

### Partial instantiation
Consider the type
```
@Structure Point:
 x : Real
 y : Real
 z : Real
```
The syntax `Point[x: 5]` defines a type of points `p` with `p.x = 5`,
this feature is called “partial instantiation”. Please note that partially instantiated coinductive types are not present in most formalizations of MLTT. They can be expressed in type theories with strict equailty: `Point[x: 5] = (p : Point, e : x ≡ 5)`, but supposably not the other way round: we hope that a variant of Homotopy Type Theory with partially instantiated coinductive types will be strong enough to encompass higher categorical structures (and its own inner model) without having non-fibrant types.

Consider the following two types
```
@Structure PointedType:
  T : *
  p : T
  
@Structure Pointed[\T : *]:
  p : T
```
Evidently, for a given type `X` the type `Pointed[X]` and the type `PointedType[T: X]` are precisely the same: availability of partial instantiation makes the difference between parameters and methods fluid. To make this explicit, in a theory with partial instantiation we will treat parametrized coinductive types as a syntactic sugar over non-parametrized ones, and will in particular allow to write `take(\l : Stream) : Nat ⟶ List[l.T]` instead of proper `take[T : *](\l : Stream[T]) : Nat ⟶ List[T]`.


## Defining Semi-Simplicial Types
(TODO)
```
@Structure SST:
  Filler{0} : Type
  Filler{succ(n : Nat)}(boundary : Simplex(n + 1, n)) : Type

  @def Simplex(n : Nat, m: Fin(n + 1)): Type
    Simplex(n : Nat, 0): (Fin(n + 1) -> Filler{0})
    Simplex(n : Nat, succ(m)): (base : Simplex(n, m)) ×
     (filler(comb : Combs(n + 1, m + 2)) : Filler(face(comb)(base)))

  @def face{n : Nat, m : Fin(n), k : Fin(n + 1)}
   (c : Combs(n, m))(s : Simplex(n, k)): Simplex(m, k ∧ m)
    face(c)(vertices : Simplex(n, 0)): vertices ∘ c    
    face(c)((base : Simplex(n, m)) ×
     (filler(c' : Combs(n + 1, m + 2)) : Filler(face(c')(base)))):
      face(c)(base) × ((c' : Combs(n + 1, m + 2)): filler(c ∘ c')) 
```
where Combs(n, m : Fin(n)) denotes the type of strictly monotone
functions from Fin(m) to Fin(n).

### Open Recursion and Hereditary Substitution
The type Stream[Nat] cannot be exhausted by induction, but we still
can define dependent types T(s : Stream[Nat]) which enable, by nature
of the dependence, very recursive definitions for dependent functions
(s : Stream[Nat]) -> T(s).

Let's assume we have a dependent type Data(l : List[Nat]) : Type that
associates some data to finite initial segments of Nat-streams. Then
consider:
```
@Structure EData(l : List[Nat])(s : Stream[Nat]):
  hdata : Data(l)
  tdata : EData(s.head :: l)(s.tail)
  pdata : s.head.elim 
    zero: ()
    succ(n : Nat): (s' : Stream[Nat]) -> EData(l)(n :: s')
```

If we have (e : EData(nil)(s)) for a stream (s : Stream[Nat]), we do
have not only Data(l) for all its initial segments of finite length,
but also EData for all streams s' which are lexicographically below
`s`, i.e. such that their first element different from s differs
towards zero. Note that lexicographic order is not wellfounded and
there are uncountably many streams below each except (0 :: 0 :: ...). 
Thus, every gadget (e : EData(nil)(s)) contains/is able to generate
lazily uncountably infinite amount of data.

Now if consider the corecursion schema for EData, tdata and pdata can
be filled automatically:
```
@def t(l : List[Nat])(s : Stream[Nat]): EData(l)(s)
  tdata: t(s.head :: l)(s.tail)
  pdata: s.head.elim
    zero: ()
    succ(n): (s' : Stream[Nat]) -> t(l)(n :: s')
  hdata: ...
```

While hdata may inspect the both the sequence and values of t for all
sequences lexicographically below it. That is exactly how the Open 
Recursion a la Berger (a constructive realizer for dependent choice)
is supposed to work.

It seems to be the only reasonable way to define an infinite number
of mutually dependent functions simultaneously, which is necessary
for defining hereditary substitution for the homotopy type theory.

~~~~~~~~

### Syntax Sugar: Equiping objects with optional structure
One very often talks about a type or an object being equipped with
an additional structure or property. To facilitate such casual usage
of structures, one eliminator returning a manifestly non-record type
(an inductive type, Π-Type or a universe) is allowed to be marked by
the @on annotation. This eliminator is implicitly applied every time
an expression won't typecheck otherwise, e.g.
```
@OrderedType{@Infix _<_}:
  @on T : Type
  (a : T)<(b : T) : Prop
  
@MonotoneMap[\X \Y : OrderedType]:
  @on f : X ⟶ Y
  monotonicity(a b : X) : (a < b) -> (f(a) < f(b))
```
