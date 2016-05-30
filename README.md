# Reedy Limits

This is a repository for playing around with syntactical definitions
for Reedy Limits in univalent type theory, i.e. a type theory where
all types are fibrant.

Even for the weakest Reddy Limits we'll need a very sophisticated
machinery, namely mutually dependent definition of an inductive and
coinductive type family, where the inductive makes an essential use
of induction-recursion. Denotational semantics of that involved
gadgets is non-trivial, so we'll first concentrate of the syntactic
side hoping that semantics can be obtained afterwards under some
unburdensome conditions if we'll manage syntacs and computations
to do the right thing.

Now here's a code primer:
```
@Structure SST:
  (x : SST).Filler{0} : Type
  (x : SST).Filler{succ(dim : Nat)}(simplex : x.Simplex(dim + 1, dim)) : Type
  
  @Family (x : SST).Simplex(n : Nat, dim: Fin(n + 1)):
    vertices{n : Nat}(v : Fin(n) -> x.Filler{0}) : x.Simplex(n, 0)
    fillings{n : Nat, dim : Fin(n + 1)}
	 (base : x.Simplex(n, dim))
	 (comb : Combs(n, dim)) -> s.Filler{dim}(face(s, comb)))
	 : x.Simplex(n, dim)

    @def face{n : Nat, m : Fin(n)}(comb : Combs(n, m))
	 {x : SST, k : Fin(n + 1)}
	 (s : x.Simplex(n, k))
	 : x.Simplex(m, k \min (m + 1))
	# complicated combinatorics
```

We coinductivly define a semisimplicial type `(x : SST)` to be
an object having a `Type`-valued family of destructors `Filler{dim}`
of n-dimensional fillers (vertices, edges, faces, 3-faces, 4-faces,
etc) indexed by unfilled simplices in `x`, while the type of
simplices in `x` is inductively defined simultaneously with `Filler`
type family.

The type `x.Simplex(n, dim)` denotes a `n`-dimensional simplex (i.e.
with `n` vertices) filled up to dimension `dim` by fillers taken from
`x.Filler` types.

0-filled simplices are given by a set of their vertices, their
constructor `vertices{n}` just takes a list of vertices (i.e.
0-dimensional fillers from `x.Filler{0}`) of the length n, we
represent these lists as functions (v : Fin(n) -> x.Filler{0}).
To get one filling dimension higher we need to provide edges for
any unordered pair of vertices, then for any unordered triple of
edges and so on.
In general in the filling dimension dim we take all combinations
"dim of n" and provide dim-fillers for each combination. To provide
the type `x.Filler{dim}(...)` for dim-fillers for these combinations
we need a procedure `face` to extract the whole subsimplices for the
given combination code `comb`, that's why we need induction-recursion.

So simultaneously with definition of x.Filler{dim} and fillings{dim}
we define the extractor function face.

Now for any (x : SST) and any natural number (n : Nat) we have the
type x.Simplex(n, n) of its n-simplices and for any face map encoded
by a strictly monotone function f : Fin(n) -> Fin(m) we have a map
subsimplex(f)(x.Simplex(n, n)) : x.Simplex(m, m), such that
composition is strictly preserved (@see composition-lemma), so they
are functors from the category Δ+ into types.