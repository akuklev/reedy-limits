# Coinduction-Corecursion

## Coinductive types and basic syntax
Coinductive types are defined by their eliminators, the most
trivial coinductive types are (dependent) records, that's why
we'll use the `@Structure` keyword for them:
```
@Structure Pair[A B : Type]:
  fst : A
  snd : B
```
(We use brakets for proof-irrelevant, and curly braces for implicit
yet possibly proof-relevant arguments. Fresh variable introduction
has the syntax `name : type` whitespaces around colon are strictly
mandatory, all fresh variables MUST be typed on the spot.)

Note that it's allowed for eliminator signatures to depend on other
eliminator values, as long as the dependency is non-circular, e.g.
```
@Structure PointedType:
  T : Type
  p : T
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
Function types (X -> Y) and П-types in general are very simple
examples of coinductive types, they just have a family of
eliminators indexed by X and returning Y:
```
@Structure Func[X Y : Type]:
  apply(x : X) : Y

@Structure DepFunc[X : Type, Y : Func[X, Type]]:
  apply(x : X) : Y.apply(x)
```

We'll allow omiting eliminator name altogether for this case to obtain
natural syntax for function types, lambda expressions and definitions
by pattern matching:
```
@Structure [X : Type]->[Y : Type]:
  (x : X) : Y
  
@def square: Nat -> Nat
  (n : Nat): n · n
  
@def factorial: Nat -> Nat
  zero: succ(zero)
  succ(n : Nat): succ(n) · factorial(n)
```

### Equiping objects with optional structure
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
  
@MonotoneMap[X Y : OrderedType]:
  @on f : X -> Y
  monotonicity(a b : X) : (a < b) -> (f(a) < f(b))
```

## Very Dependent Types
Now if we were to allow a function type to be defined simultaneously
(allowing non-circular mutual dependence) with an inductive family of
eliminators, we obtain the Very Dependent Types as defined by Jason
Hickey, that's the coinductive counterpart of induction-recursion.

```
@Structure VeryDepFuncOnNat:
  apply(n : Nat) : TypeAt(n)
  
  @def TypeAt: Nat -> Type
    zero: T0
	succ(n): T(n, apply(n))
	# an expression depending on n and the value
	# of apply(n) on the current object.
```

It is conjectured [NK15] that the types defined with help of this
technique are precisely the Reedy limits. In particular, we think
that the following code defines semi-simplicial types:
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
Primary usage of coinduction is the definition of infinitary data
structures, such as
```
@Structure Stream[T : Type]:
  head: T
  tail: Stream[T]
```

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

