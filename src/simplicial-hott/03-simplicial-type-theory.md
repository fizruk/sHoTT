# 3. Simplicial Type Theory

These formalisations correspond to Section 3 of RS17 paper.

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Simplicies and their subshapes

Simplicies:

```rzk
-- 1-simplex
#def Δ¹ : 2 -> TOPE
  := \t -> TOP

-- 2-simplex
#def Δ² : (2 * 2) -> TOPE
  := \(t, s) -> s <= t

-- 3-simplex
#def Δ³ : (2 * 2 * 2) -> TOPE
  := \((t1, t2), t3) -> t3 <= t2 /\ t2 <= t1
```

Boundaries of simplices:

```rzk
-- boundary of a 1-simplex
#def ∂Δ¹ : Δ¹ -> TOPE
  := \t -> (t === 0_2 \/ t === 1_2)

-- boundary of a 2-simplex
#def ∂Δ² : Δ² -> TOPE
  := \ts -> 
        ((second ts) === 0_2 \/ (first ts) === 1_2 \/ (second ts) === (first ts))
```

Horns:

```rzk
-- the (2,1)-horn
#def Λ : (2 * 2) -> TOPE
  := \(t, s) -> (s === 0_2 \/ t === 1_2)
```

Products:

```rzk
-- the product of topes defines the product of shapes
#def shapeProd
  (I J : CUBE)
  (psi : I -> TOPE)
  (chi : J -> TOPE)
  : (I * J) -> TOPE
  := \(t, s) -> psi t /\ chi s

-- the square as a product
#def Δ¹×Δ¹ : (2 * 2) -> TOPE
  := shapeProd 2 2 Δ¹ Δ¹

-- the vertical boundary of the square 
#def ∂Δ¹×Δ¹ : (2 * 2) -> TOPE
  := shapeProd 2 2 ∂Δ¹ Δ¹

-- the prism from a 2-simplex in an arrow type
#def Δ²×Δ¹ : (2 * 2 * 2) -> TOPE
  := shapeProd (2 * 2) 2 Δ² Δ¹  
```  
