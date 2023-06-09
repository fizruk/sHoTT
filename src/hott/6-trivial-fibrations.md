# 6. Trivial Fibrations

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Contractible fibers
```rzk
-- In what follows we apply show that the projection from the total space of a sigma type is an equivalence if and only if its fibers are contractible
#def total-space-projection
    (A : U)
    (B : A -> U)
    : (∑ (x : A), B x) -> A
    := \z -> first z

-- The type that asserts that the fibers of a type family are contractible
#def contractible-fibers
    (A : U)
    (B : A -> U)
    : U
    := ((x : A) -> isContr (B x))

-- The center of contraction in a contractible fibers
#def contractible-fibers-section 
    (A : U)
    (B : A -> U)
    (ABcontrfib : contractible-fibers A B)
    : (x : A) -> B x
    := \x -> contraction-center (B x) (ABcontrfib x)

-- The section of the total space projection built from the contraction centers
#def contractible-fibers-actual-section 
    (A : U)
    (B : A -> U)
    (ABcontrfib : contractible-fibers A B)
    : (x : A) -> ∑ (x : A), B x
    := \x -> (x , contractible-fibers-section A B ABcontrfib x)

#def contractible-fibers-section-htpy 
    (A : U)
    (B : A -> U)
    (ABcontrfib : contractible-fibers A B)
    : homotopy A A 
            (composition A (∑ (x : A), B x) A (total-space-projection A B) (contractible-fibers-actual-section A B ABcontrfib))
            (identity A)
    := \x -> refl

#def contractible-fibers-section-is-section 
    (A : U)
    (B : A -> U)
    (ABcontrfib : contractible-fibers A B)
    : hasSection (∑ (x : A), B x) A (total-space-projection A B)
    := (contractible-fibers-actual-section A B ABcontrfib , contractible-fibers-section-htpy A B ABcontrfib)

-- This can be used to define the retraction homotopy for the total space projection, called "first" here
#def contractible-fibers-retraction-htpy 
    (A : U)
    (B : A -> U)
    (ABcontrfib : contractible-fibers A B)
    : (z : ∑ (x : A), B x) 
        -> ((contractible-fibers-actual-section A B ABcontrfib) (first z)) = z
     := \z -> fibered-path-to-sigma-path A B (first z) ((contractible-fibers-section A B ABcontrfib) (first z)) (second z)
            (contracting-htpy (B (first z)) (ABcontrfib (first z)) (second z))

#def contractible-fibers-retraction
    (A : U)
    (B : A -> U)
    (ABcontrfib : contractible-fibers A B)
    : hasRetraction (∑ (x : A), B x) A (total-space-projection A B)
    := (contractible-fibers-actual-section A B ABcontrfib , contractible-fibers-retraction-htpy A B ABcontrfib)

-- The first half of our main result:
#def contractible-fibers-projection-equiv 
    (A : U)
    (B : A -> U)
    (ABcontrfib : contractible-fibers A B)
    : isEquiv (∑ (x : A), B x) A (total-space-projection A B)
    := (contractible-fibers-retraction A B ABcontrfib , contractible-fibers-section-is-section A B ABcontrfib)
```

## Projection equivalences

```rzk
-- From a projection equivalence, it's not hard to inhabit fibers
#def projection-equiv-implies-inhabited-fibers 
    (A : U)
    (B : A -> U)
    (ABprojequiv : isEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (x : A)
    : B x
    := transport A B (first ((first (second ABprojequiv)) x)) x 
            ((second (second ABprojequiv)) x) (second ((first (second ABprojequiv)) x))

-- This is great but I'll need more coherence to show that the inhabited fibers are contractible; the following proof fails
-- #def projection-equiv-implies-contractible-fibers 
--    (A : U)
--    (B : A -> U)
--    (ABprojequiv : isEquiv (∑ (x : A), B x) A (total-space-projection A B)) 
--    : contractible-fibers A B
--    := \x -> (second ((first (first ABprojequiv)) x) , 
--        \u -> total-path-to-fibered-path A B ((first (first ABprojequiv)) x) (x, u) ((second (first ABprojequiv)) (x, u)) )

-- We start over from a stronger hypothesis of a half adjoint equivalence
#def projection-coherent-equiv-inverse 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (a : A) 
    : ∑ (x : A), B x
    := (first (first ABprojHAE)) a

#def projection-coherent-equiv-base-htpy 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (a : A) 
    : (first (projection-coherent-equiv-inverse A B ABprojHAE a)) = a
    := (second (second (first ABprojHAE))) a

#def projection-coherent-equiv-section 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (a : A) 
    : B a
    := transport A B (first (projection-coherent-equiv-inverse A B ABprojHAE a)) a 
            (projection-coherent-equiv-base-htpy A B ABprojHAE a) 
            (second (projection-coherent-equiv-inverse A B ABprojHAE a))

#def projection-coherent-equiv-total-htpy 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (w : (∑ (x : A), B x)) 
    : (projection-coherent-equiv-inverse A B ABprojHAE (first w)) = w
    := (first (second (first ABprojHAE))) w

#def projection-coherent-equiv-fibered-htpy 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (w : (∑ (x : A), B x)) 
    : (transport A B (first ((projection-coherent-equiv-inverse A B ABprojHAE (first w)))) (first w) 
            (total-path-to-base-path A B (projection-coherent-equiv-inverse A B ABprojHAE (first w)) w
                (projection-coherent-equiv-total-htpy A B ABprojHAE w)) 
            (second (projection-coherent-equiv-inverse A B ABprojHAE (first w)))) 
                = (second w)
    := total-path-to-fibered-path A B (projection-coherent-equiv-inverse A B ABprojHAE (first w)) w
            (projection-coherent-equiv-total-htpy A B ABprojHAE w)

#def projection-coherent-equiv-base-coherence 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (w : (∑ (x : A), B x)) 
    : (projection-coherent-equiv-base-htpy A B ABprojHAE (first w)) 
                = (total-path-to-base-path A B (projection-coherent-equiv-inverse A B ABprojHAE (first w)) w
                    (projection-coherent-equiv-total-htpy A B ABprojHAE w)) 
    := (second ABprojHAE) w

#def projection-coherent-equiv-transport-coherence 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (w : (∑ (x : A), B x)) 
    : (projection-coherent-equiv-section A B ABprojHAE (first w))
             =  (transport A B (first ((projection-coherent-equiv-inverse A B ABprojHAE (first w)))) (first w) 
                    (total-path-to-base-path A B (projection-coherent-equiv-inverse A B ABprojHAE (first w)) w
                        (projection-coherent-equiv-total-htpy A B ABprojHAE w)) 
                    (second (projection-coherent-equiv-inverse A B ABprojHAE (first w))))
    := transport2 A B (first (projection-coherent-equiv-inverse A B ABprojHAE (first w))) (first w) 
            (projection-coherent-equiv-base-htpy A B ABprojHAE (first w)) 
            (total-path-to-base-path A B (projection-coherent-equiv-inverse A B ABprojHAE (first w)) w
                (projection-coherent-equiv-total-htpy A B ABprojHAE w))
            (projection-coherent-equiv-base-coherence A B ABprojHAE w)
            (second (projection-coherent-equiv-inverse A B ABprojHAE (first w)))

#def projection-coherent-equiv-fibered-contracting-htpy 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    (w : (∑ (x : A), B x)) 
    : (projection-coherent-equiv-section A B ABprojHAE (first w)) =_{B (first w)} (second w)
    := concat (B (first w)) 
            (projection-coherent-equiv-section A B ABprojHAE (first w))
            (transport A B (first ((projection-coherent-equiv-inverse A B ABprojHAE (first w)))) (first w)
                (total-path-to-base-path A B (projection-coherent-equiv-inverse A B ABprojHAE (first w)) w
                    (projection-coherent-equiv-total-htpy A B ABprojHAE w)) 
                (second (projection-coherent-equiv-inverse A B ABprojHAE (first w))))
            (second w)
            (projection-coherent-equiv-transport-coherence A B ABprojHAE w)
            (projection-coherent-equiv-fibered-htpy A B ABprojHAE w)

-- Finally we have
#def projection-coherent-equiv-contractible-fibers 
    (A : U)
    (B : A -> U) 
    (ABprojHAE : isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B))
    : contractible-fibers A B
    := \x -> ((projection-coherent-equiv-section A B ABprojHAE x), 
                \u -> (projection-coherent-equiv-fibered-contracting-htpy A B ABprojHAE (x, u)))
    
-- The converse to our first result    
#def projection-equiv-contractible-fibers 
    (A : U)
    (B : A -> U)
    (ABprojequiv : isEquiv (∑ (x : A), B x) A (total-space-projection A B))
    : contractible-fibers A B
    := projection-coherent-equiv-contractible-fibers A B 
            (isEquiv-isHalfAdjointEquiv (∑ (x : A), B x) A (total-space-projection A B) ABprojequiv)
    
-- The main theorem    
#def projection-theorem 
    (A : U)
    (B : (a : A) -> U) 
    : iff (isEquiv (∑ (x : A), B x) A (total-space-projection A B)) (contractible-fibers A B)
    := (\ABprojequiv -> projection-equiv-contractible-fibers A B ABprojequiv, 
            \ABcontrfib -> contractible-fibers-projection-equiv A B ABcontrfib)
```            