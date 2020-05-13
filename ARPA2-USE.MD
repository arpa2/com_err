# Modern Use of COM_ERR

> *How com_err should have been defined, and how we can fix it.*

First off, `com_err` is a very old, but also a very good idea.
Here is a small list of error code ranges in TLS Pool:

```
# Use com_err to translate this table to more-or-less
# unique codes that fit a signed 32-bit integer without
# hitting on POSIX error codes (which are not standard
# but for 0).
#
# Everybody  uses           0 to           0
# UNIX       uses           1 to         255
# Unbound    uses         -10 to          -1
# GnuTLS     uses        -500 to          -1 (overlap!)
# BerkeleyDB uses      -30999 to      -30800
# TLS Pool   uses -1167372288 to -1167372033
# MIT krb5   uses -1765328384 to -1765328129 (multi-range)
#
# Our module name "tlsP" landed us in the given range.
# All error messages start with "TLS Pool" to direct
# attention of administrators seeking to shoot trouble.
```

This is terrible in terms of design, because there is no
structure in the ranges.  At the very least, `com_err` allows
a minimal risk that ranges could clash.  Basically, identities
of 3-4 characters determine unique ranges of error codes. But
that only works when the various packages actually use it.


## Why plain Error Handling is Silly

The `errno` range for `<errno.h>` errors is limited.  So
many packages define their own error codes, error printing
routines and so on.  As a package using several other
packages, you end up juggling various error values.  This
can be simplified if we use the normal `errno` variable
and have several tables for printing them as messages.
This is just what `com_err` is doing, except that the use
of `errno` is our personal extension.


## What went wrong

The `com_err` package is very old.  Its assumption is that
it takes a `long` to store a 32-bit signed integer.  Today,
we would have used `int32_t` from `<stdint.h>` for that kind
of use.

We tried (in past versions) to retro-fit the error table
compiler `compile_et`, is known to never produce integers
longer than 32 bits, to use `int32_t` instead of `long`.
This works, but only when we compile all error tables with
this software.

Enter Kerberos, and probably other tools that have their
own built-in `compile_et` version.  This produces a
structure with `long` error code variables.  Linking to
a library that uses `int32_t` would lead to misalignment
of data structures.


## What we are Hoping to Do

The reason to care about this is to be able to use `errno`
from `<errno.h>` as a storage variable for the codes that
the error table compiler produces.  This would even work
with threading, due to special support and macro magic.

This may not be possible in compilers that produce 16-bit
integers, but practically every compiler reserves enough
space for the `int` type of `errno` to allow it to store
the 32-bit numbers from `com_err` and allow processing
the codes with `error_message()` instead of `strerror()`
and so on.


## The best fix

Any error code produced by the `compile_et` library should
be type-cast to `int32_t` and can then be stored in `errno`
where a range error may follow if the latter is too small,
but the type cast suppresses reports about storing a
`long` value into an `int` variable.

The output of `compile_et` however, is more like this:

```
#include <et/com_err.h>

#define KIPERR_KEY_REQUIRED                      (11685888L)
#define KIPERR_KEYID_NOT_FOUND                   (11685889L)
#define KIPERR_NO_MATCHING_KEY                   (11685890L)
```

The `L` is a problem; it would be completely useful to
instead produce

```
#include <stdint.h>
#include <et/com_err.h>

#define KIPERR_KEY_REQUIRED                      ((int32_t) 11685888L)
#define KIPERR_KEYID_NOT_FOUND                   ((int32_t) 11685889L)
#define KIPERR_NO_MATCHING_KEY                   ((int32_t) 11685890L)
```

Another format is the use of an explicit cast,

```
if (!some_test) {
   errno = (int32_t) KIPERR_NO_MATCHING_KEY;
   goto fail;
}
```

This form would be required for external dependencies anyway.
There is never a problem to do thinks like

```
printf ("Some Message (%s) -- Try Again\n", error_message (errno));
```

because `error_message()` expects a `long` and the cast from `errno`
is implied by the C language.

