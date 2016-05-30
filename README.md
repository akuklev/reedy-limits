# Reedy Limits

This is a repository for playing around with syntactical definitions
for Reedy Limits in univalent type theory, i.e. a type theory where
all types are fibrant.

Even for the weakest Reddy limits we'll need a very sophisticated
machinery, namely the coinductive counterpart of induction-recursion:
coinduction-recursion, a mutually dependent (however non-circular)
definition of a coinductive type and a function signature of which
depends on the type being defined. This is precisely the construction
anticipated by Nicolai Kraus [1] and strongly related to so called
very dependent types by Jason Hickey [2]. Categorical semantics of
such gadgets was not yet studied, but we hope that if we get syntax
and computation rules to do the right thing, semantics can bent into
shape under some unburdensome conditions.

Look into coinduction-recursion.md for brief introduction to used
syntax and short discussion of Coinduction-Recursion.

Now here's a code primer:
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
We coinductivly define a semisimplicial type `(x : SST)` to be an
object having a `Type`-valued family of eliminators `Filler{dim}`.

These are types of n-dimensional fillers (vertices, edges, faces,
3-faces, 4-faces, etc). For `n > 0`, they are indexed by their
boundaries, i.e. simplices with `(n + 1)` vertices where all faces
up to dimension `(n - 1)` are filled.

The type `Simplex(n, m)` of n-dimensional simplices filled up to
dimension m is defined recursively, its definition is mutually but
non-circularily dependent on `Filler{dim}` and a further helper
function `face`.

Unfilled simplices are given by a list of their vertices
`(v : Fin(n) -> x.Filler{0})`. Then we have to fill its edges (i.e.
1-dimensional subsimplices), faces (i.e. 2-dimensional subsimplices),
etc. How can we write that down? 

Well, let Combs(n, m : Fin(n)) denote a type of strictly monotone
functions from Fin(m) to Fin(n), thus its inhabitants choose
unordered combinations (couples, triples) of elements of a list
of the length n. If we a given an dim-dimensional simplex (it has
`dim + 1` vertices), each its dim'-dimensional subsimplex can be
uniquely described by choice of (dim' + 1) vertices among (dim + 1)
that we have, thus by an element comb : Combs(dim + 1, dim' + 1).

Now we'll define an extracting function function face(comb, simplex)
that extract given subsimplex from the simplex, then we can give a
definition for m-filled simplex, namely for going a dimension higher
one has to provide a filler for each m-subsimplices:
```
Simplex(n : Nat, succ(m)): (base : Simplex(n, m)) ×
 (filler(comb : Combs(n + 1, m + 2)) : Filler(face(comb)(base)))
```

Now for any (x : SST) and any natural number (n : Nat) we have the
type x.Simplex(n, n) of its n-simplices and for any face map encoded
by a strictly monotone function f : Fin(n) -> Fin(m) we have a map
face(f)(x.Simplex(n, n)) : x.Simplex(m, m), such that
composition is strictly preserved by definition of face. 
Thus we obtain functors from the category Δ+ into types.