# Segal Types

These formalisations correspond to Section 5 of RS17 paper.

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Prerequisites

- `hott/total-space.rzk` — we rely on `contractible-fibers-projection-equiv` and `total-space-projection` from that file in the proof of Theorem 5.5
- `3-simplicial-type-theory.rzk` — we rely on definitions of simplicies and their subshapes

## The Segal condition

```rzk
-- [RS17, Definition 5.1]
-- The type of arrow in A from x to y
#def hom : (A : U) -> (x : A) -> (y : A) -> U
  := \A -> \x -> \y -> <{t : 2 | Δ¹ t } -> A [ ∂Δ¹ t |-> recOR(t === 0_2, t === 1_2, x, y) ]>

-- [RS17, Definition 5.2]
#def hom2 : (A : U) -> (x : A) -> (y : A) -> (z : A) -> (f : hom A x y) -> (g : hom A y z) -> (h : hom A x z) -> U
  := \A -> \x -> \y -> \z -> \f -> \g -> \h ->
    <{(t, s) : 2 * 2 | Δ² (t, s)} -> A	[ ∂Δ² (t, s) |->
        	recOR(s === 0_2, t === 1_2 \/ s === t, f t, recOR(t === 1_2, s === t, g s, h s)) ]>

-- [RS17, Definition 5.3]
#def isSegal : (A : U) -> U
  := \A -> (x : A) -> (y : A) -> (z : A) -> (f : hom A x y) -> (g : hom A y z) 
  -> isContr( ∑ (h : hom A x z), hom2 A x y z f g h)
```

```rzk
-- Segal types have a composition functor
#def Segal-comp : (A : U) -> (_ : isSegal A) -> (x : A) -> (y : A) -> (z : A) 
  -> (_ : hom A y z) -> (_ : hom A x y) -> hom A x z
  := \A -> \AisSegal -> \x -> \y -> \z -> \g -> \f -> first (first (AisSegal x y z f g))
```

### Characterizing Segal types

```rzk
-- more generally, there is a restriction along any subspace inclusion
#def horn-restriction : (A : U) -> (f : <{t : 2 * 2 | Δ² t} -> A >) -> <{t : 2 * 2 | Λ t} -> A >
  := \A -> \f -> \t -> f t

-- checking partial evaluations of functions
#def identity-identity-composition : (A : U) -> (composition A A A (identity A) (identity A)) =_{(z : A) -> A} (identity A)
  := \A -> refl_{identity A : (a : A) -> A}

#def horn
  : (A : U) ->
    (x : A) -> (y : A) -> (z : A) ->
    (f : hom A x y) ->
    (g : hom A y z) ->
    <{t : 2 * 2 | Λ t } -> A >
  := \A -> \x -> \y -> \z -> \f -> \g ->
    \{(t, s) : 2 * 2 | Λ (t, s) } -> 
      recOR(s === 0_2, t === 1_2, f t, g s)

-- Here, we prove the equivalence used in [RS17, Theorem 5.5].
-- However, we do this by constructing the equivalence directly,
-- instead of using a composition of equivalences, as it is easier to write down
-- and it computes better (we can use refl for the witnesses of the equivalence).
#def compositions-are-horn-fillings
  : (A : U) ->
    (x : A) -> (y : A) -> (z : A) ->
    (f : hom A x y) ->
    (g : hom A y z) ->
    Eq (∑ (h : hom A x z), hom2 A x y z f g h)
       <{t : 2 * 2 | Δ² t } -> A [ Λ t |-> horn A x y z f g t ]>
  := \A -> \x -> \y -> \z -> \f -> \g ->
    (\hh -> \{t : 2 * 2 | Δ² t} -> (second hh) t,
      ((\k -> (\(t : 2) -> k (t, t), \{(t, s) : 2 * 2 | Δ² (t, s)} -> k (t, s)), \hh -> refl_{hh}),
       (\k -> (\(t : 2) -> k (t, t), \{(t, s) : 2 * 2 | Δ² (t, s)} -> k (t, s)), \hh -> refl_{hh})))


#def restriction-equiv
  : (A : U) ->
    Eq (<{t : 2 * 2 | Δ² t} -> A >)
      (∑ (k : <{t : 2 * 2 | Λ t} -> A >),
        ∑ (h : hom A (k (0_2, 0_2)) (k (1_2, 1_2))),
          hom2 A (k (0_2, 0_2)) (k (1_2, 0_2)) (k (1_2, 1_2))
                 (\t -> k (t, 0_2)) (\t -> k (1_2, t)) h)
  := \A ->
    (\k ->
      (\{t : 2 * 2 | Λ t} -> k t,
        (\(t : 2) -> k (t, t),
         \{t : 2 * 2 | Δ² t} -> k t)),
     ((\khh -> \{t : 2 * 2 | Δ² t} -> (second (second khh)) t, \k -> refl_{k}),
      (\khh -> \{t : 2 * 2 | Δ² t} -> (second (second khh)) t, \k -> refl_{k})))

-- [RS17, Theorem 5.5]
#def Segal-restriction-equiv : (A : U) -> (AisSegal : isSegal A) 
  -> Eq (<{t : 2 * 2 | Δ² t} -> A >) (<{t : 2 * 2 | Λ t} -> A >) -- (horn-restriction A)
  := \A -> \AisSegal ->
    compose_Eq
        (<{t : 2 * 2 | Δ² t} -> A >)
        (∑ (k : <{t : 2 * 2 | Λ t} -> A >),
            ∑ (h : hom A (k (0_2, 0_2)) (k (1_2, 1_2))),
            hom2 A (k (0_2, 0_2)) (k (1_2, 0_2)) (k (1_2, 1_2))
                    (\t -> k (t, 0_2)) (\t -> k (1_2, t)) h)
        (<{t : 2 * 2 | Λ t} -> A >)
        (restriction-equiv A)
        (total-space-projection
            (<{t : 2 * 2 | Λ t} -> A >)
            (\k -> ∑ (h : hom A (k (0_2, 0_2)) (k (1_2, 1_2))),
                        hom2 A (k (0_2, 0_2)) (k (1_2, 0_2)) (k (1_2, 1_2))
                            (\t -> k (t, 0_2)) (\t -> k (1_2, t)) h),
        (contractible-fibers-projection-equiv
            (<{t : 2 * 2 | Λ t} -> A >)
            (\k -> ∑ (h : hom A (k (0_2, 0_2)) (k (1_2, 1_2))),
                        hom2 A (k (0_2, 0_2)) (k (1_2, 0_2)) (k (1_2, 1_2))
                            (\t -> k (t, 0_2)) (\t -> k (1_2, t)) h)
            (\k -> AisSegal (k (0_2, 0_2)) (k (1_2, 0_2)) (k (1_2, 1_2))
                            (\t -> k (t, 0_2)) (\t -> k (1_2, t)))))
```

## Identity

```rzk
-- [RS17, Definition 5.7]
-- Segal types have identity arrows
#def Segal-id : (A : U) -> (_ : isSegal A) -> (x : A) -> hom A x x
  := \A -> \AisSegal -> \x -> \{t : 2 | Δ¹ t} -> x 

-- [RS17, Proposition 5.8a]
-- the right unit law for identity
#def Segal-comp-id : (A : U) -> (AisSegal : isSegal A) -> (x : A) -> (y : A) -> (f : hom A x y) 
  -> hom2 A x y y f (Segal-id A AisSegal y) f
  := \A -> \AisSegal -> \x -> \y -> \f -> \{(t, s) : 2 * 2 | Δ² (t, s)} -> f t

-- [RS17, Proposition 5.8b]
-- the left unit law for identity
#def Segal-id-comp : (A : U) -> (AisSegal : isSegal A) -> (x : A) -> (y : A) -> (f : hom A x y) 
   -> hom2 A x x y (Segal-id A AisSegal x) f f
  := \A -> \AisSegal -> \x -> \y -> \f -> \{(t, s) : 2 * 2 | Δ² (t, s)} -> f s
```

## Associativity

To be done.

## Homotopies

To be done.

## Anodyne maps

To be done.