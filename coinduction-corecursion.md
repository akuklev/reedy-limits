§ Coinduction-corecursion

Coinductive types are defined by their eliminators, the most
trivial coinductive types are records, that's why we'll use
the `@Structure` keyword for them:
```
@Structure Pair[A B : Type]:
  fst : A
  snd : B
```
Elements of coinductive types can be defined by means of
corecursion, i.e. by providing for the object we define
a scheme how to compute each of eliminators on it, e.g.
```
@def p: Pair[Nat, Nat]
  fst: 1
  snd: 2
```
Eliminating has the syntax `p.fst`.

In general corecursion can be very involved, @see 
[http://www.types2016.uns.ac.rs/images/abstracts/setzer2.pdf].

Function types (X -> Y) and П-types in general are very simple
examples of coinductive types, they just have a family of
eliminators indexed by X and returning Y:
```
@Structure Func[X Y : Type]:
  apply(x : X) : Y

@Structure DepFunc[X : Type, Y : Func[X, Type]]:
  apply(x : X) : Y.apply(x)
```

Now if we were to allow a function to be defined type simultaneously
(allowing non-circular mutual dependence) with an inductive family of
eliminators, we obtain the Very-Dependent Types as defined by Jason
Hickey, that's the coinductive counterpart of induction-recursion.

```
@Structure VeryDepFuncOnNat:
  apply(n : Nat) : TypeAt(n)
  
  @def TypeAt(n : Nat):
    zero: T0
	succ(n): T(n, apply(n))
	# an expression depending on n and the value
	# of apply(n) on the current object.
```

