Resilient c_can driver patch
============================

The standard Linux c_can driver quite reasonably assumes that the underlying
hardware is well-behaved and adheres to the following rules:

- if it raises a receive interrupt for a message buffer, that buffer will
  have new data in it
- if it accepts a transfer of a populated transmit buffer, it will actually
  send that message and eventually raise a transmit interrupt

It turns out that, for at least some hardware in sufficiently adverse
environments, these two assumptions may be false.

The patches in this repo modify the c_can driver implementation to avoid
making either of those assumptions. They definitely introduce some additional
latency, and probably limit the maximum data throughput of the driver, but
are known to be able to cope with at least of 500 kbps CAN bus rate.

The patches also make various other changes that simplified the process of
identifying the underlying hardware faults (including turning off all the
hardware interrupts the driver doesn't actually care about).


Building the modified drivers
-----------------------------

1. Clone the linux-stable git repo: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
2. Check out the desired baseline kernel version (e.g. `v3.16.43`, `v4.19.67`)
3. Apply the relevant patch (conflicts may occur at this point if there's no
   patch file matching the target kernel series)
4. Run the following steps (ensuring that the CAN module is enabled in the
   networking section, and the Bosch C_CAN/D_CAN module is enabled in the
   CAN device driver section):

    make ARCH=i386 defconfig
    make oldconfig
    make prepare
    make modules
    make M=drivers/net/can

This will produce the following relevant driver files:

    drivers/net/can/c_can/c_can.ko
    drivers/net/can/c_can/c_can_pci.ko
    drivers/net/can/can-dev.ko

It will typically be desirable to reduce the size of these by stripping debug
symbols:

    strip --strip-debug drivers/net/can/c_can/c_can.ko \
        drivers/net/can/c_can/c_can_pci.ko drivers/net/can/can-dev.ko

If the final make command doesn't actually emit any CAN drivers, use
`make menuconfig` to check that CAN support is enabled (set to `<M>`) in the
networking section, and the Bosch C_CAN/D_CAN module is enabled under that.
The rerun the following steps:

    make modules
    make M=drivers/net/can


Installing the modified drivers
-------------------------------

Use whatever mechanism is appropriate for your target system to install an
updated kernel module:

* update base system image
* create a custom kernel package
* directly replace the kernel modules under `/lib/modules/` (this option is
  not ideal but it works as long as the base kernel version is ABI compatible)
