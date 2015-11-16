
[![Build Status](https://travis-ci.org/waylonflinn/bvec.svg?branch=master)](https://travis-ci.org/waylonflinn/bvec)

# bvec

![bvec Logo](bvec.png)


## Fast Vector Operations on Pretty Big Data
bvec is numpy<sup>1</sup> for pretty big data<sup>2</sup>. It's based on [Bcolz](https://github.com/Blosc/bcolz/) and includes transparent disk-based storage for large results.

Supports

* dot
* divide

Coming Soon
* _multiply_
* _add_
* _subtract_


<sup>1.</sup> 1-dimensional and 2-dimensional operations on numerical data types
(`int64`, `int32`, `float64`, `float32`).

<sup>2.</sup> bigger than a breadbox, smaller than a blimp

## Install
```bash
pip install bvec
```

or build from source (requires bcolz >= 0.9.0)

```bash
python setup.py build_ext --inplace
python setup.py install
```

## Usage

### Dot

Multiply a matrix (`carray`) with a vector (`numpy.ndarray`), returns a vector (`numpy.ndarray`)

```python
import bvec
import numpy as np

matrix = np.random.random_integers(0, 12000, size=(300000, 100))
bcarray = bvec.carray(matrix, chunklen=2**13, cparams=bvec.cparams(clevel=2))

v = bcarray[0]

result = bcarray.dot(v)
expected = matrix.dot(v)

# should return True
(expected == result).all()

```


Multiply a matrix (`carray`) with the transpose of a matrix (`carray`), returns a matrix (`carray`)

```python
import bvec
import numpy as np

matrix = np.random.random_integers(0, 120, size=(1000, 100))
bcarray1 = bvec.carray(matrix, chunklen=2**9, cparams=bvec.cparams(clevel=2))
bcarray2 = bvec.carray(matrix, chunklen=2**9, cparams=bvec.cparams(clevel=2))

# calculates bcarray1 . bcarray2.T (transpose)
result = bcarray1.dot(bcarray2)
expected = matrix.dot(matrix.T)

# should return True
(expected == result).all()

```
### Save Result to Disk
Save really big results directly to disk

```python
# create correctly sized container (helper method, not required)
output = bcarray1.empty_like_dot(bcarray2, rootdir='/path/to/bcolz/output')

# generate results directly on disk
bcarray1.dot(bcarray2, out=output)

# make sure the last bits get written
output.flush()
```

The `out` parameter can also be used to get `ndarray` output, or specify an existing `bcolz` store.

## Test

```python
nosetests bvec
```

## Simple Benchmarks

Benchmarks were done on data structures generated by the above code, are very informal, and vary a bit across data sets.

### Space

* `numpy` ~229MB
* `bvec` ~64MB

compression ratio: 3.5

### Time

* `numpy` ~33 ms
* `bvec` ~48 ms

percent performance: 68%

## Goals

This project has three goals, each slightly more fantastic than the last:

1. Allow computation on (compressed) data which is (~5-10x) larger than RAM at approximately the same speed as `numpy.dot`


2. Allow computation on (slightly compressed) data at speeds that improve on `numpy.dot`


3. Allow computation on (compressed) data which resides on disk at some sizable percentage (~50-30%) of the speed of `numpy.dot`


So far, the first goal has been met.


## Acknowledgements

This library wouldn't be possible without all the talented people who worked hard to create [bcolz](https://github.com/Blosc/bcolz/) (and the libraries on which it's based).

Initial code was also heavily influenced by [bquery](https://github.com/visualfabriq/bquery).

Dot product code based on
[bdot](https://github.com/tailwind/bdot) (&copy; 2015 Tailwind)

Awesome TARDIS can be found [here](https://youtu.be/dUBxHd3bMhg?t=1m5s)
