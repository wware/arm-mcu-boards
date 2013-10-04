====================
ARM microcontrollers
====================

There are a lot of interesting microcontroller boards these days with 32-bit
ARM processors (see Wikipedia's `list of ARM cores`_) and I'd like to do some
tinkering with them. And I'd like a codebase that makes it easy to start a
new project using one of them.

Originally I had done a lot of this work on Ubuntu 10.04, but nowadays I work on
a Macbook running Ubuntu 12.04 in a VirtualBox (be sure to use the amd64 ISO to
set it up). Prerequisites for building the toolchain::

 sudo apt-get install libgmp3-dev libmpfr-dev texinfo libncurses5-dev

.. _`list of ARM cores`: http://en.wikipedia.org/wiki/List_of_ARM_microprocessor_cores

In the past I've played with the ARM7TDMI cores, particularly the SAM7 family
from Atmel. More recently I'm interested in the more advanced Cortex-M3 core,
with one of the more popular implementations being the STM32_ family from ST
Microelectronics, used on the Leaflabs Maple board and the STM32VL Discovery_
board.

.. _STM32: http://en.wikipedia.org/wiki/STM32
.. _Discovery: http://en.wikipedia.org/wiki/STM32#Discovery_kits

As a general rule, each manufacturer designs their own peripherals that they
connect to the core. The SAM7 peripherals typically have one 32-bit register
that sets bits and another that clears bits.

There is an interesting-looking open-source RTOS called `ChibiOS/RT`_.

.. _`ChibiOS/RT`: http://www.chibios.org/dokuwiki/doku.php

It provides preemptive multithreading, 128 priority levels, a scheduler,
timers, semaphores, mutexes, queues, etc.

Specific ideas to tinker with
=============================

People have started using JavaScript as a hardware control language. This is
a surprisingly good idea. JS's approach to concurrency is enviable. You have
a single thread of execution, so every function runs to completion before
any other thread. Communication with other things is done with events and
queues which invoke callbacks. In this way, JS neatly avoids all the nasty
concurrency cruft that most other languages suffer with, while providing a
snappy user experience in the browser, because the UI thread never blocks.

Android and iOS also have a non-blocking UI thread that communicates with
the rest of the system using events and queues. So it's the same pattern
again.

More recently my thinking on easy concurrency has been influenced by an
18-minute talk given by Ned Batchelder (starting at about the six-minute mark
of `this video`_) discussing the use of mutable and immutable values in Python.
Use of immutable values to eliminate thread interaction is employed in the
Clojure_ language.

.. _`this video`: http://www.youtube.com/watch?v=hO7S600Reok
.. _Clojure: http://clojure.org/

I've considered a couple of ways to do this in an embedded platform. One
would be to port NodeJS_ and simply run JavaScript as the BeagleBone Black
does. But the BBB has a complete Linux implementation and I'm too lazy to
do a bare-metal port of NodeJS.

.. _NodeJS: http://nodejs.org/

My current thinking is that I can get the same gain by writing threads in
ANY language, provided

* there are no global variables, only global constants with immutable values
* the only interaction between threads is via queues, and only immutable values
  can be enqueued

I have been giving some thought to creating a thread-safe language that uses
these ideas in a virtual machine, and has some kind of bytecode compiler. But
in the shorter term, I can do these kinds of experiments simply in C, as long
as I get the threading to work right on an embedded platform.
