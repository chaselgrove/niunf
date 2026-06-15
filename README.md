# niunf

The goal of this project is to develop a semantic fingerprint for
neuroimaging data.  The concept of a semantic fingerprint and some
techniques are taken from the Dataverse [Universal Numerical
Fingerprint](https://guides.dataverse.org/en/latest/developers/unf/index.html)
(UNF).

The purpose of a UNF is to give a dataset a unique fingerprint
that distinguishes it from other data sets.  If the data set is
changed or corrupted, the UNF will change.  And important aspect
of a UNF that distinguishes it from a checksum is that the UNF of
a dataset applies to the semantic content of the dataset, so a
change in, say, numeric data type or data container format will not
change the UNF.

The UNF is not defined for raster or matrix data, so we cannot
create a UNF for neuroimaging data; rather, we develop a method to
transform neuroimaging volumes so that a UNF can be calculated.
The approach is therefore accurately described as the UNF of
neuroimaging data prepared according to this scheme, but this
documentation might colloquially refer to the UNF of a neuroimaging
volume (or raster or matrix data).

It is also important to acknowledge that declaring what information
is part of the semantic meaning of a dataset is a choice and a
tradeoff.  The UNF, for example, does not consider units in its
calculation; this is arguably a fatal oversight since numbers without
units carry no semantic meaning, but even so the UNF has proven
useful.

Exploratory schemes are:

- [Scheme 1](SCHEME_1.md): Reorders data based on properties of the
data index to real world mapping, then applies UNFs.
- [Scheme 2](SCHEME_2.md): Treats data as a collection of
(x, y, z, value) vectors.
