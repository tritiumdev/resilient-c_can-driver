Resilient c_can driver patch
============================

The standard Linux c_can driver quite reasonably assumes that the underlying
hardware is well-behaved and adheres to the following rules:

- if it raises a receive interrupt for a message buffer, that buffer will 
  have new data in it
- if it accepts a transfer of a populated transmit buffer, it will actually
  send that message and eventually raise a transmit interrupt

It turns out that, for at least some hardware, these two assumptions may be
false.

The patch in this repo modifies the c_can driver implementation to avoid
making either of those assumptions.

It also makes various other changes that simplified the process of identifying
the underlying hardware faults.
