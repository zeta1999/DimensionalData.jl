# DimensionalData

[![](https://img.shields.io/badge/docs-stable-blue.svg)](https://rafaqz.github.io/DimensionalData.jl/stable)
[![](https://img.shields.io/badge/docs-dev-blue.svg)](https://rafaqz.github.io/DimensionalData.jl/dev)
[![Build Status](https://travis-ci.org/rafaqz/DimensionalData.jl.svg?branch=master)](https://travis-ci.org/rafaqz/DimensionalData.jl)
[![Codecov](https://codecov.io/gh/rafaqz/DimensionalData.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/rafaqz/DimensionalData.jl)
[![Aqua.jl Quality Assurance](https://img.shields.io/badge/Aqua.jl-%F0%9F%8C%A2-aqua.svg)](https://github.com/JuliaTesting/Aqua.jl)


DimensionalData.jl provides tools and abstractions for working with datasets
that have named dimensions. It's a pluggable, generalised version of
[AxisArrays.jl](https://github.com/JuliaArrays/AxisArrays.jl) with a cleaner
syntax, and additional functionality found in NamedDimensions.jl. It has similar
goals to pythons [xarray](http://xarray.pydata.org/en/stable/), and is primarily
written for use with spatial data in [GeoData.jl](https://github.com/rafaqz/GeoData.jl).

:exclamation: | Status
:-----------: | :-------

    This is a work in progress under active development, it may be a while before
    the interface stabilises and things are fully documented.


## Dimensions

Dimensions are just wrapper types. They store the dimension index
and define details about the grid and other metadata, and are also used
to index into the array, wrapping a value or a `Selector`.
`X`, `Y`, `Z` and `Ti` are the exported defaults.

A generalised `Dim` type is available to use arbitrary symbols to name dimensions.
Custom dimensions can be defined using the `@dim` macro.

We can use dim wrappers for indexing, so that the dimension order in the underlying array
does not need to be known:

```julia
a[X(1:10), Y(1:4)]
```

The core component is the `AbstractDimension`, and types that inherit from it,
such as `Time`, `X`, `Y`, `Z`, the generic `Dim{:x}` or others you
define manually using the `@dim` macro.

Dims can be used for indexing and views without knowing dimension order:

```julia
a[X(20)]
view(a, X(1:20), Y(30:40))
```

And for indicating dimensions to reduce or permute in julia
`Base` and `Statistics` functions that have dims arguments:

```julia
`mean(a, dims=Time)` 
`permutedims(a, [X, Y, Z, Time])` 
```


## Selectors

Selectors find indices in the dimension based on values `At`, `Near`, or
`Between` the index value(s). They can be used in `getindex`, `setindex!` and
`view` to select indices matching the passed in value(s)

- `At(x)`: get indices exactly matching the passed in value(s)
- `Near(x)`: get the closest indices to the passed in value(s)
- `Where(f::Function)`: filter the array axis by a function of dimension 
  index values.
- `Between(a, b)`: get all indices between two values (inclusive)
- `Contains(x)`: get indices where the value x falls in the interval. 
  Only used for `Sampled` `Intervals`, for `Points` us `At`.

We can use selectors with dim wrappers:

```julia
a[X(Between(1, 10)), Y(At(25.7))]
```

Without dim wrappers selectors must be in the right order:

```julia
using Unitful
a[Near(23u"s"), Between(10.5u"m", 50.5u"m")]
```

For values other than `Int`/`AbstractArray`/`Colon` (which are set aside for regular indexing) the `At`
selector is assumed, and can be dropped completely:

```julia
julia> A = DimArray(rand(3, 3), (X(Val((:a, :b, :c))), Y([25.6, 25.7, 25.8])))
DimArray with dimensions:
 X: Val{(:a, :b, :c)}()
 Y: Float64[25.6, 25.7, 25.8]
and data: 3×3 Array{Float64,2}
 0.280308  0.92255    0.023938
 0.129487  0.0993857  0.618391
 0.246378  0.276186   0.0425624

julia> A[:b, 25.8]
0.61839141062599
```

It's also easy to write your own custom `Selector` if your need a different behaviour.

_Example usage:_

```julia
using Dates, DimensionalData
timespan = DateTime(2001,1):Month(1):DateTime(2001,12)
A = DimArray(rand(12,10), (Ti(timespan), X(10:10:100)))

julia> A[X(Near(35)), Ti(At(DateTime(2001,5)))]
0.658404535807791

julia> A[Near(DateTime(2001, 5, 4)), Between(20, 50)]
DimArray with dimensions:
 X: 20:10:50
and referenced dimensions:
 Time (type Ti): 2001-05-01T00:00:00
and data: 4-element Array{Float64,1}
[0.456175, 0.737336, 0.658405, 0.520152]
```

Dim types or objects can be used instead of a dimension number in many
Base and Statistics methods:

## Methods where dims can be used containing indices or Selectors

`getindex`, `setindex!` `view`

## Methods where dims can be used

- `size`, `axes`, `firstindex`, `lastindex`
- `cat`
- `reverse`
- `dropdims`
- `reduce`, `mapreduce`
- `sum`, `prod`, `maximum`, `minimum`, 
- `mean`, `median`, `extrema`, `std`, `var`, `cor`, `cov`
- `permutedims`, `adjoint`, `transpose`, `Transpose`
- `mapslices`, `eachslice`
- `fill`

_Example usage:_

```julia
A = DimArray(rand(20,10), (X, Y))
size(A, Y)
using Statistics
mean(A, dims=X)
std(A; dims=Y())
```

## Alternate Packages

There are a lot of similar julia packages in this space. AxisArrays.jl, NamedDims.jl, NamedArrays.jl are registered alternative that each cover some of the functionality provided by DimensionalData.jl. DimensionalData.jl should be able to replicate any of their functionality, although with slightly more verbose syntax and less polish in some cases. If there is anything it doesn't do that these packages can do, put in an issue with the feature requrest.

[AxisRanges.jl](https://github.com/mcabbott/AxisRanges.jl) and [AbstractIndices.jl](https://github.com/Tokazama/AbstractIndices.jl) are some other interesting developments. For more detail on why there are so many similar options and where things are headed, read this [thread](https://github.com/JuliaCollections/AxisArraysFuture/issues/1)
