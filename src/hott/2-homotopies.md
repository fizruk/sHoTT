# 3. Homotopies

This is a literate `rzk` file:

```rzk
#lang rzk-1
```
## Homotopies and their algebra

```rzk
-- The type of homotopies between parallel functions.
#def homotopy 
    (A B : U)               -- Two types.
    (f g : A -> B)          -- Two parallel functions.
    : U
    := (a : A) -> (f a = g a)
    
#def homotopy-rev 
    (A B : U)               -- Two types.
    (f g : A -> B)          -- Two parallel functions.
    (H : homotopy A B f g)  -- A homotopy from f to g.
    : homotopy A B g f
    := \a -> rev B (f a) (g a) (H a)

-- Homotopy composition is defined in diagrammatic order like concat but unlike composition.
#def homotopy-composition 
    (A B : U)                 -- Two types.
    (f g h : A -> B)          -- Three parallel functions.
    (H : homotopy A B f g)
    (K : homotopy A B g h)
    : homotopy A B f h
    := \a -> concat B (f a) (g a) (h a) (H a) (K a)

#def homotopy-postwhisker 
    (A B C : U)                 -- Three types.
    (f g : A -> B)              -- Two parallel functions.
    (H : homotopy A B f g)      -- A homotopy from f to g.
    (h : B -> C)
    : homotopy A C (composition A B C h f) (composition A B C h g)
    := \a -> ap B C (f a) (g a) h (H a)

#def homotopy-prewhisker 
    (A B C : U)                 -- Three types.
    (f g : B -> C)              -- Two parallel functions
    (H : homotopy B C f g)
    (h : A -> B)
    : homotopy A C (composition A B C f h) (composition A B C g h)
    := \a -> H (h a)
```

## Naturality

```rzk
-- The naturality square associated to a homotopy and a path.
#def nat-htpy 
    (A B : U)               -- Two types.
    (f g : A -> B)          -- Two parallel functions.
    (H : homotopy A B f g)  -- A homotopy from f to g.
    (x y : A)
    (p : x = y)
    : (concat B (f x) (f y) (g y) (ap A B x y f p) (H y)) = (concat B (f x) (g x) (g y) (H x) (ap A B x y g p))
    := idJ(A, x, 
            \y' p' 
                -> (concat B (f x) (f y') (g y') 
                        (ap A B x y' f p') (H y')) =
                   (concat B (f x) (g x) (g y') 
                        (H x) (ap A B x y' g p')), 
            refl-concat B (f x) (g x) (H x), y, p)

-- In the case of a homotopy H from f to the identity the previous square applies to the path H a to produce the following naturality square.
#def a-cylinder-homotopy-coherence 
    (A : U)
    (f : A -> A) 
    (H : homotopy A A f (identity A))
    (a : A) 
    : (concat A (f (f a)) (f a) a (ap A A (f a) a f (H a)) (H a)) =
            (concat A (f (f a)) (f a) (a) (H (f a)) (ap A A (f a) a (identity A) (H a)))
    := nat-htpy A A f (identity A) H (f a) a (H a)

-- After composing with ap-id, this naturality square transforms to the following:
#def another-cylinder-homotopy-coherence 
    (A : U)
    (f : A -> A) 
    (H : homotopy A A f (identity A))
    (a : A) 
    : (concat A (f (f a)) (f a) a (ap A A (f a) a f (H a)) (H a)) =
            (concat A (f (f a)) (f a) (a) (H (f a)) (H a))
    := concat ((f (f a)) = a) (concat A (f (f a)) (f a) a (ap A A (f a) a f (H a)) (H a)) 
            (concat A (f (f a)) (f a) (a) (H (f a)) (ap A A (f a) a (identity A) (H a)))
            (concat A (f (f a)) (f a) (a) (H (f a)) (H a))
                (a-cylinder-homotopy-coherence A f H a)
                (concat-homotopy A (f (f a)) (f a) (H (f a)) a (ap A A (f a) a (identity A) (H a)) (H a) (ap-id A (f a) a (H a)))

-- Cancelling the path (H a) on the right and reversing yields a path we need:
#def cylinder-homotopy-coherence 
    (A : U)
    (f : A -> A) 
    (H : homotopy A A f (identity A))
    (a : A) 
    : (H (f a)) =(ap A A (f a) a  f (H a))  
    := rev (f (f a) = f a) (ap A A (f a) a  f (H a)) (H (f a)) 
        (concat-right-cancel A (f (f a)) (f a) a 
            (ap A A (f a) a  f (H a)) 
            (H (f a)) 
            (H a) 
                (another-cylinder-homotopy-coherence A f H a))
```
