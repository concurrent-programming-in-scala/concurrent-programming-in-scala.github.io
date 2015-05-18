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

