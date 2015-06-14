# Fast Dot Products on Pretty Big Data

Right now only `matrix . vector` (nearest neighbor search style) dot products are suppored.

## Goals

This project has three goals, each slightly more fantastic than the last:

1. Allow computation on (compressed) data which is (~5-10x) larger than RAM at approximately the same speed as `numpy.dot`


2. Allow computation on (slightly compressed) data at speeds that improve on `numpy.dot`


3. Allow computation on (compressed) data which resides on disk at some small multiple (~2-3x) of the speed of `numpy.dot`


So far, the first goal has been met, bdot makes your RAM bigger on the inside.

![Bigger on the Inside](https://31.media.tumblr.com/dcd82ee9cc541ef6774572e9110de082/tumblr_inline_n3eq30Vjhh1rnbe7i.gif)

[Bcolz](https://github.com/Blosc/bcolz/) is the reason these things are possible.

## Usage

```
import bdot
import bcolz
import numpy as np

matrix = np.random.random_integers(0, 12000, size=(300000, 100))
bcarray = bdot.carray(matrix, chunklen=2**13, cparams=bcolz.cparams(clevel=2))

v = bcarray[0]

result = bcarray.dot(v)
```

## Simple Benchmarks

Benchmarks were done on data structures generated by the above code, are very informal, and vary a bit across data sets.

### Space

* bcarray ~64MB
* matrix ~229MB

compression ratio: 3.55 
(this tells you what multiple of the RAM in your machine you can operate with)


compression ratio on real world (non-random) data tends to be better (~5x)

### Time
```
%timeit -n30 -r2 bcarray.dot(v)
```

`30 loops, best of 2: 48.2 ms per loop`

```
%timeit -n30 -r2 matrix.dot(v)
```

`30 loops, best of 2: 33 ms per loop`


## Tests

```
(bcarray.dot(v) == matrix.dot(v)).all()
```

`True`

## Acknowledgements

This library wouldn't be possible without all the talented people who worked hard to create Bcolz (and the libraries on which it's based). Initial code was also heavily influenced by [Bquery](https://github.com/visualfabriq/bquery).