.. See file COPYING distributed with niunf for copyright and license.

=====
niunf
=====

Introduction
------------

niunf extends Universal Numeric Fingerprints (UNFs) to neuroimaging volumes.

The purpose of a UNF is to give a data set a unique fingerprint that distinguishes it from other data sets.  If the data set is changed or corrupted, the UNF will change.  And important aspect of a UNF that distinguishes it from a checksum is that the UNF of a dataset applies to the semantic content of the dataset, so a change in, say, numeric data type or data container format will not change the UNF.

http://guides.dataverse.org/en/latest/developers/unf/index.html

Image data (in the form we are interested in here) contains a sampling of values at regularly spaced points in space.  A 2- or 3-D sampling of values is described by its dimensionality, orientation in space, and serialized for storage in a file.  For instance, a 2-D sampling: ::

 y
 3 | 5 6
 2 | 3 4
 1 | 1 2
   +----
     1 2 x

or in tabular form: ::

  x | y | value
 ---+---+-------
  1 | 1 |   1
  2 | 1 |   2
  1 | 2 |   3
  2 | 2 |   4
  1 | 3 |   5
  2 | 3 |   6

This can be more efficiently stored on disk as just the values and a description of the x and y coordinates of each value, or by storing: ::

 1 2 3 4 5 6

and specifying:

- there are two values in the fastest varying direction (as you move along the serialized values, changes in x are seen faster than changes in y) and three values in the next fastest varying direction

- the first value is at (x, y) = (1, 1)

- and single index changes in the fastest varying direction correspond to a change of (x, y) = (1, 0), and single index changes in the next fastest varying direction correspond to a change of (x, y) = (0, 1).

The same data could be stored on disk as: ::

 6 4 2 5 3 1

with the first value at (2, 3), three fastest varying values corresponding to changes of (0, -1) and two next fastest varying values corresponding to changes of (-1, 0).

These two representations have the same semantic meaning, so must have the same UNF.

This version of niunf applies to 3D volumes.  See below for other limitations.

Conventions
-----------

The fastest, second fastest, and slowest varying voxel indices are i, j, and k, respectively.

The real-world coordinates are expressed as r, a, and s.

The mapping between the voxel indices and the real-world coordinate system is expressed by a matrix A: ::

     [ m11 m12 m13 a ]
 A = [ m21 m22 m23 b ]
     [ m31 m32 m23 c ]
     [  0   0   0  1 ]

such that: ::

 [ r ]   [ m_ir m_jr m_kr o_r ] [ i ]
 [ a ] = [ m_ia m_ja m_ka o_a ] [ j ]
 [ s ]   [ m_is m_js m_ks o_s ] [ k ]
 [ 1 ]   [  0    0    0    1  ] [ 1 ]

o_r, o_a, and o_s are the offset or origin terms; these describe the location of the first voxel (where (i, j, k) = (0, 0, 0)).  The m values describe the motion in space as each voxel index increases, so one step along i corresponds to motion of (m_ir, m_ia, m_is) in space.

See http://nipy.org/nibabel/coordinate_systems.html for a more complete treatment of coordinate systems.

UNF scheme
----------

The UNF must take into account the values in the volume and the position of these values in space, so it must operate on A and the (3-D) matrix of values D.  Since this will ultimately be a mathematical operation, we need to manipulate A and D before this calculation so that different representations of the same data are expressed the same way before calculating the UNF.  In other words, we need to agree on how we transform the two representations of our initial (2-D) example: ::

     [ 1 0 1 ]
 A = [ 0 1 1 ]
     [ 0 0 1 ]

 M = [ [ 1 2 ] [ 3 4 ] [ 5 6 ] ]

and ::

     [  0 -1 2 ]
 A = [ -1  0 3 ]
     [  0  0 1 ]

 M = [ [ 6 4 2 ] [ 5 3 1 ] ]

to feed the UNF algorithm the same inputs.

There are two variables to consider: the direction of each dimension and the order the dimensions are stored.

For each dimension (i, j, k), examine in turn m_r, m_a, and m_s.  Transform to ensure that the first non-zero value encountered is positive.

In the first (2-D) case above, (m_ir, m_ia) = (1, 0).  m_ir is non-zero and positive, so we leave its direction.  (m_jr, m_ja) = (0, 1), so we skip m_jr since it is zero and find that m_ja is positive, so we leave its direction.

In the second case, we find that (m_ir, m_ia) = (0, -1).  We skip m_ir and find that m_ia is negative, so we must swap the direction of this dimension.  (m_jr, m_ja) = (-1, 0), so we must also swap its direction.  We then have: ::

     [ 0 1 1 ]
 A = [ 1 0 1 ]
     [ 0 0 1 ]

 M = [ [ 1 3 5 ] [ 2 4 6 ] ]

Note that o_r and o_a are also modified in A to preserve its mapping from (i, j) to (r, a).

Note also that this transformation will fail if all of the m_* are zero for a given dimension, but if this is the case, the volume is ill-defined and a UNF is undefined.

Now we handle the order of the dimensions.  Note that the columns of A in the two cases are the same, but in different orders.  We calculate the UNF for each column as a vector, including the zero in the final place: ::

 UNF((1, 0, 0)) = UNF:6:URJNfhUTYGE1Eg/J6r1jDg==

 UNF((0, 1, 0)) = UNF:6:wy3A2VEtZmSX5p6MDG701A==

These are then (POSIX-locale) sorted to determine the order of dimensions.  Since "UNF:6:URJNfhUTYGE1Eg/J6r1jDg==" comes before "UNF:6:wy3A2VEtZmSX5p6MDG701A==", (1, 0, 0) must be the direction of the fastest-varying index.  Now each case becomes: ::

     [ 1 0 1 ]
 A = [ 0 1 1 ]
     [ 0 0 1 ]

 M = [ [ 1 2 ] [ 3 4 ] [ 5 6 ] ]

and we have the same inputs to the UNF calculation.

The final UNF is calculated by creating a vector from the UNF for A and the UNF for M, in that order, and calculating the UNF for the vector.  In this case: ::

 UNF(A) = UNF:6:59HfZp8Y4JL2iV1VYIUToQ==
 UNF(M) = UNF:6:0bcm3Fem0lGYlWI5ctnahg==

 UNF(image) = UNF(("UNF:6:59HfZp8Y4JL2iV1VYIUToQ==", "UNF:6:0bcm3Fem0lGYlWI5ctnahg==") = UNF:6:GtdcjAw+tnOeyQlafNHnjA==

Note that the UNF of A and M follow the python-unf scheme for matrices, not the R scheme, which treats matrices as data frames and reorders components of the matrix before calculating the final UNF.

We also do not distinguish between positive and negative zeros in this calculation, treating them as semantically equal.  All negative zeros are converted to positive zeros before being used in a UNF calculation.
