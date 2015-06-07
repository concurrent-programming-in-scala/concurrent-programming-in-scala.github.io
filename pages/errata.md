---
title: Learning Concurrent Programming in Scala
---

# Book Errata

## Chapter 3

### CAS pseudocode

On page 70, the following code snippet is shown as the pseudocode of the CAS operation:

    // Incorrect
    def compareAndSet(ov: Long, nv: Long): Boolean =
      this.synchronized {
        if (this.get == ov) false else {
          this.set(nv)
          true
        } 
      }

This is not consistent with the text -- CAS returns false if the current value is
*different* than the expected value `ov`.
The actual pseudocode should be the following:

    // Correct
    def compareAndSet(ov: Long, nv: Long): Boolean =
      this.synchronized {
        if (this.get != ov) false else {
          this.set(nv)
          true
        } 
      }

Similarly, the reference-based CAS should be (note `ne` instead of `eq`):

    // Correct
    def compareAndSet(ov: T, nv: T): Boolean =
      this.synchronized {
        if (this.get ne ov) false else {
          this.set(nv)
          true
        } 
      }

*Thanks [Normen Müller](https://github.com/normenmueller)!*


### Atomic buffers

On page 84, the `AtomicBuffer` example is as follows:

    class AtomicBuffer[T] {
      private val buffer = new AtomicReference[List[T]](Nil)
      @tailrec def +=(x: T): Unit = {
        val xs = buffer.get
        val nxs = x :: xs
        if (!buffer.compareAndSet(xs, nxs)) this += x
      }
    }

The method `+=` is tail recursive, so it should be either final or private.
Since we want to expose `+=` to the clients, we need to mark it with `final`, as follows:

    @tailrec final def +=(x: T): Unit = {
      val xs = buffer.get
      val nxs = x :: xs
      if (!buffer.compareAndSet(xs, nxs)) this += x
    }

*Thanks [Normen Müller](https://github.com/normenmueller)!*


## Chapter 9

### Scalable Concurrent Accumulator

On page 331, the scalable concurrent accumulator specialized for `Long`s is not
properly initializing the `total` value in its `apply` method.

    class ParLongAccumulator(z: Long)(op: (Long, Long) => Long) {
      private val par = Runtime.getRuntime.availableProcessors * 128
      private val values = new AtomicLongArray(par)
      @tailrec final def add(v: Long): Unit = {
        val id = Thread.currentThread.getId.toInt
        val i = math.abs(scala.util.hashing.byteswap32(id)) % par
        val ov = values.get(i)
        val nv = op(ov, v)
        if (!values.compareAndSet(i, ov, nv)) add(v)
      }
      def apply(): Long = {
        var total = 0L
        for (i <- 0 until values.length) total = op(total, values.get(i))
        total
      }
    }

The `apply` method should instead initialize the local variable `total` with `z` instead of `0L`.

      def apply(): Long = {
        var total = z
        for (i <- 0 until values.length) total = op(total, values.get(i))
        total
      }

*Thanks [Yuki N.](https://github.com/fairjm)!*
