# Common Error Description Library for UNIX

This is an ancient piece of work, but strangely enough we need
it more than ever.  A great diversity of libraries, often used
underneath a few layers of abstraction, want to report errors.

Instead of merging ranges in the same `errno`, we have started
to summarise and overwrite error codes.  The approach of `com_err`
is much better.

The compiler, `compile_et`, takes an error table that defines
C-style symbols with strings into "unique" integer codes and
strings to explain them.

The codes are spread by hashing the table name (3 to 4 chars
long) into a 24-bit code in the top of an int32 range.  The
result can be positive or negative.  The low 8 bits are used
for the individual error codes in the range.

With 24 bits, there need to be 4096 error tables before there
is a 50% chance of a clash.  That is doable, even with today's
complex systems.

**Changes.**
The classical code generates `long` values, which back in 1989
was a way of specifying 32-bit numbers.  We now have an accurate
notation for 32-bit numbers, namely `int32` from `<stdint.h>`
and so shall generate those.
