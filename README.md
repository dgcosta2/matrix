# Rhombus Matrix

The Matrix class allows the creation of n-dimensional matrices (or tensors) in Rhombus along with a series of operations on them. The Matrix object contains three fields:

- data: an internal flat representation of the `Matrix` as a `List` of `Number`s
- shape: a `List` containing the sizes of each dimension of the `Matrix` object
- stride: a `List` of factors used for indexing


## Constructors
```
def m = Matrix()
```
The no-args constructor creates an "empty" matrix with empty `data`, `shape`, and `stride` lists

___

```
def m = Matrix(~rows: n :: PosInt, ~columns: k :: PosInt)
```
`m` is a 2-dimensional array (matrix) of zero's with n rows and k columns. The field `shape` is set to `[n, k]` and `stride` to `[k, 1]`.

**Example**

```
> Matrix(~rows: 3, ~columns: 2)
[[0, 0], [0, 0], [0, 0]]
```
____________________________
```
def m = Matrix(~l: l)
```
This constructor takes a possibly nested list `l` and creates a `Matrix` object. Its `shape` and `stride` are calculated from the nesting of `l`

**Example**

```
> def m = Matrix(~l: [[3, 2, 4],
                      [1, -1, 0]])
> m.data
[3, 2, 4, 1, -1, 0]
> m.shape
[2, 3]                      
```
_________________________
```
def m = Matrix(~l: l :: List, ~s: s :: List)
```

This constructor can be used for assigning a shape `s` to a flat list `l`. The constraint

$\Pi s_i = l.length()$

where $s_i$ are the members of `s` must be obeyed.

**Example**
```
> def m = Matrix(~l: [1, 2, 3, 4], ~s: [2, 2])
> m
[[1, 2], [3, 4]]
m.shape
[2, 2]
```
__________________
```
def m = Matrix(~s: s)
```

This constructor allows the user to create a `Matrix` with the desired shape filled with 0s.

**Example**
```
> Matrix(~s: [3, 2, 2])
[[[0, 0], [0, 0]], [[0, 0], [0, 0]], [[0, 0], [0, 0]]]
```

## Operations

No operations involving `Matrix` objects happen in-place
### add

Addition requires two objects of the same `shape`. The method 
```
add(other :: Matrix)
```
adds every element of `this` to its corresponding (same index) element in `other`.

**Example**
```
> def m = Matrix(~l: [1, 2, 5, 4, 2, 3], ~s: [2, 3])
> def n = Matrix(~l: [0, 1, 3, 2, -1, 1], ~s: [2, 3])
> m.add(n)
[[1, 3, 8], [6, 1, 4]]
```

The operator `<+>` is provided 

```
> def m = Matrix(~l: [1, 2, 5, 4, 2, 3], ~s: [2, 3])
> def n = Matrix(~l: [0, 1, 3, 2, -1, 1], ~s: [2, 3])
> m <+> n
[[1, 3, 8], [6, 1, 4]]
```
### s_mult

The method
```
s_mult(n :: Number)
```
multiplies every elements in `this` Matrix by `n`

**Example**
```
> def m = Matrix(~l: [1, 2, 5, 4, 2, 3], ~s: [2, 3])
> m.s_mult(2)
[[2, 4, 10], [8, 4, 6]]
```

The operator `<*>` is provided
```
> def m = Matrix(~l: [1, 2, 5, 4, 2, 3], ~s: [2, 3])
> 2 <*> m
[[2, 4, 10], [8, 4, 6]]
```
> The scalar must be to the left of the operator and the `Matrix` object to the right

### Tensor dot product

The method
```
contr(other :: Matrix)
```

performs a tensor dot product which, for 2-dimensional matrices, is the standard matrix product ($A_{k \times n} \times B_{n \times l} = C_{k \times l}$, where $c_{ij} = \sum_{m = 1}^{n} a_{im} b_{mj}$). The library provides the operator `<.>` for tensor contraction

```
> def m1 = Matrix(~l: [[3, 7], [4, 9]])
> def m2 = Matrix(~l: [[6, 2], [5, 8]])
> m1 <.> m2
[[53, 62], [69, 80]]
```

For example, for a tensor contraction between tensors `m1` and `m2` where `m1.shape` is `[3, 4, 2]` and `m2.shape` is `[2, 5, 7]`, the resulting tensor `m3 = m1 <.> m2` has shape `[3, 4, 5, 7]` (the last dimension in `m1` gets contracted with the first dimension of `m2`) and the element `m3[[i, j, k, m]]` is the sum of all `m1[[i, j, n]] * m2[[n, k, m]]`

As expected, multiplying a matrix $A_{n \times k}$ by $I_{n \times k}$ results in $A$ itself.

```
> def m_id = Matrix(~l: [1, 0, 0, 0, 1, 0, 0, 0, 1], ~s: [3, 3])
> def m2 = Matrix(~l: [1, 2, 3, 4, 5, 6, 7, 8, 9], ~s: [3, 3])
> m2 <.> m_id == m2
#true 
```

### Determinants

The library can calculate the determinant of any `Matrix` objects in which the `shape` is `[n, n]` where `n` is a `PosInt`.

```
> def m1 = Matrix(~l: [2, -8, 6, 8, 3, -9, 5, 10, -3, 0, 1, -2, 1, -4, 0, 6], ~s: [4, 4])
> m1.det()
-36
> def m2 = Matrix(~l: [3, -1, 2, -5, 0, 5, -3, -6, -6, 7, -7, 4, -5, -8, 0, 9], ~s: [4, 4])
> m2.det()
0
```

### Row Reduction

The method
```
row_reduce()
```

generates an upper-triangular `Matrix` object from `this` by performing Gaussian elimination. 

```
> def m1 = Matrix(~l: [[3, 7], [4, 9]])
> m1.row_reduce()
[[4, 9], [0, 1/4]]
```

### Copy

To create a new `Matrix` object with the same `data` and `shape` use the method `copy()`

```
> def m1 = Matrix(~s: [3, 4, 5])
> def m2 = m1.copy()
> m1 == m2
#true
> m1[[1, 2, 1]] := 1
> m1 == m2
#false
```

## Object Protocols

The `Matrix` class inherits from `Indexable`, `MutableIndexable`, `Equatable`, and `Printable`.

### Indexing

To access members of a `Matrix` object you can pass a list of indexes to `[...]`. For example, for the `Matrix` object
```
def m1 = Matrix(~l: for List(i: 0..30): i, ~s: [5, 3, 2])
```
you can access the first element in this object by calling
```
> m1[[1, 1, 1]]
0
```
> Indexing in `Matrix` uses mathematical indices that begin at 1, not 0.

> The outer brackets are a requirement of the `Indexable` Interface, while the inner ones represent the `List` of indices

If the list passed has less elements than `shape`, a `Matrix` object is returned.

```
> m1[[1, 1]]
[0, 1]
```
### Mutability

You can set a specific position in the `Matrix` object using indexing and the operator `:=`
```
> def m1 = Matrix(~l: [1, 2, 3, 4, 5, 6], ~s:[3, 2])
> m1[[2, 1]]
3
> m1[[2, 1]] := 0
> m1[[2, 1]]
0
```

This operation does not create a new `Matrix` object, instead it creates a new `data` list and assigns it to the field `data`.

### Equality

Two `Matrix` objects are equal if and only if they have the same `data` *and* `shape`.

```
> def m1 = Matrix(~l: [1, 2, 3, 4, 5, 6], ~s:[3, 2])
> def m2 = Matrix(~l: [[2, 4],
                       [6, 8],
                       [10, 12]])
> 2 <*> m1 == m2
#true 
> def m3 = Matrix(~l: [1, 2, 3, 4, 5, 6], ~s:[2, 3])
> m1 == m3
#false                    
```