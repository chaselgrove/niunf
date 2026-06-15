# Scheme 2

This scheme treats the dataset as a collection of (position, value)
vectors.  If we take the 2-D image from [Scheme 1](SCHEME_1.md):

```
y
3 | 5 6
2 | 3 4
1 | 1 2
  +----
    1 2 x
```

and look at it again in tabular form:

| x | y | value |
| --- | --- | --- |
| 1 | 1 | 1 |
| 2 | 1 | 2 |
| 1 | 2 | 3 |
| 2 | 2 | 4 |
| 1 | 3 | 5 |
| 2 | 3 | 6 |

we see that the data can be expressed as a collection of vectors:

```
(1, 1, 1)
(2, 1, 2)
(1, 2, 3)
(2, 2, 4)
(1, 3, 5)
(2, 3, 6)
```

We then borrow from the UNF specification and its treatment of a
collection of parts: UNF, sort, UNF.  We calculate UNFs for the
elements:

```
UNF((1, 1, 1)) = UNF:6:iHmCtWyALeThX6KuvdFOAQ==
UNF((2, 1, 2)) = UNF:6:O8c+OReZf8RUpFrprIW+7w==
UNF((1, 2, 3)) = UNF:6:AvELPR5QTaBbnq6S22Msow==
UNF((2, 2, 4)) = UNF:6:T+JOnNWriQ5/s0SLUJVFXg==
UNF((1, 3, 5)) = UNF:6:uQ/0PTuPBFzw/0GjN8Th5w==
UNF((2, 3, 6)) = UNF:6:N5+ILNgsYxoMrOWPmuMwdQ==
```

order them:

```
UNF:6:AvELPR5QTaBbnq6S22Msow==
UNF:6:N5+ILNgsYxoMrOWPmuMwdQ==
UNF:6:O8c+OReZf8RUpFrprIW+7w==
UNF:6:T+JOnNWriQ5/s0SLUJVFXg==
UNF:6:iHmCtWyALeThX6KuvdFOAQ==
UNF:6:uQ/0PTuPBFzw/0GjN8Th5w==
```

and calculate the UNF of the vector of these:

```
UNF(image) = UNF((
    "UNF:6:AvELPR5QTaBbnq6S22Msow=="
    "UNF:6:N5+ILNgsYxoMrOWPmuMwdQ=="
    "UNF:6:O8c+OReZf8RUpFrprIW+7w=="
    "UNF:6:T+JOnNWriQ5/s0SLUJVFXg=="
    "UNF:6:iHmCtWyALeThX6KuvdFOAQ=="
    "UNF:6:uQ/0PTuPBFzw/0GjN8Th5w==")) = UNF:6:1SurrYDpPqg2gSQULxXZ1w==
```

This scheme adapts trivially to three dimensions ((x, y, z, value)),
time series ((x, y, z, t, value)), RGB values, and so on.

## Alternate

The above scheme adapts operations in the UNF specification, but
the calculation of a UNF for each point in the data set is relatively
expensive.  An alternative is to order take the (position, value)
vectors:

```
(1, 1, 1)
(2, 1, 2)
(1, 2, 3)
(2, 2, 4)
(1, 3, 5)
(2, 3, 6)
```

order them by positions:

```
(1, 1, 1)
(1, 2, 3)
(1, 3, 5)
(2, 1, 2)
(2, 2, 4)
(2, 3, 6)
```

join these into a single vector:

```
(1, 1, 1, 1, 2, 3, 1, 3, 5, 2, 1, 2, 2, 2, 4, 2, 3, 6)
```

and calculate the UNF of that vector:

```
UNF((1, 1, 1, 1, 2, 3, 1, 3, 5, 2, 1, 2, 2, 2, 4, 2, 3, 6) = UNF:6:4EwPeIK8x5DQGlqSFx2SVA==
```
