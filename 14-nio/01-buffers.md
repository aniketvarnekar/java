# Buffers

## Learning Objectives

- Understand `ByteBuffer`'s capacity/position/limit model precisely
- Use `flip()`, `clear()`, `rewind()`, and `compact()` correctly, and know exactly what each does to the buffer's internal state
- Understand why NIO is built around buffers rather than the direct read/write calls of `java.io`

## Prerequisites

Module 13 (IO), Module 09 (Arrays — a buffer is conceptually a wrapped array with extra bookkeeping)

## Motivation

Every single NIO operation — reading a file, writing to a socket, non-blocking I/O — flows through a `Buffer`. Getting this one concept precisely right unlocks everything else in this module; getting it wrong (a genuinely common beginner experience, since the API is less intuitive than streams) makes every subsequent topic confusing.

## The Core Difference: `java.io` vs. `java.nio`'s Data Model

Recall Module 13: a `java.io` stream is **unidirectional and unbuffered-by-default** — you `read()` data and it's immediately gone from the stream's perspective, or you `write()` and it's immediately sent. **NIO instead reads/writes data into and out of a `Buffer`** — a fixed-capacity container you can inspect, rewind, and re-read, which is what makes non-blocking operations (Topic 3) and efficient bulk transfers (Topic 2) possible at all.

## `ByteBuffer` — The Core Buffer Type

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);   // a NEW buffer, 1024 bytes capacity
```

Every `Buffer` (there are typed variants — `ByteBuffer`, `CharBuffer`, `IntBuffer`, etc. — `ByteBuffer` is by far the most commonly used, since raw I/O is fundamentally byte-based, Module 13, Topic 1) tracks **four** key properties:

```
 Buffer State

capacity : the TOTAL, FIXED size of the buffer (set at creation, NEVER changes)
limit    : the boundary past which you cannot read or write
position : the NEXT index that will be read from or written to
mark     : a saved position you can return to later (less commonly used)


  0                     position                 limit               capacity
  │                         │                      │                     │
  ├─────────────────────────┼──────────────────────┼─────────────────────┤
  │ already processed       │ available to         │ unused buffer space │
  │ (already read/          │ read/write           │                     │
  │  written)               │                      │                     │
  └─────────────────────────┴──────────────────────┴─────────────────────┘
```

**These four properties always satisfy: `0 <= mark <= position <= limit <= capacity`.**

## The Write-Then-Read Cycle — `flip()`

**A buffer is used in two distinct modes**: **writing** into it, then **reading** from it — and switching between these modes is where beginners most commonly get confused:

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);

// --- WRITE MODE ---
buffer.put((byte) 'H');
buffer.put((byte) 'i');
// after two put() calls: position = 2, limit = 1024 (still the full capacity)

// --- SWITCH TO READ MODE ---
buffer.flip();   // THE MOST IMPORTANT, MOST EASILY FORGOTTEN NIO METHOD
// flip() does exactly two things:
//   1. limit = position   (limit now marks "how much real data is here" -- exactly 2 bytes)
//   2. position = 0         (reading starts from the beginning)

// --- READ MODE ---
while (buffer.hasRemaining()) {   // hasRemaining() == (position < limit)
    byte b = buffer.get();          // reads ONE byte, ADVANCES position by 1
    System.out.print((char) b);
}
// prints: Hi
```

```
BEFORE flip() (write mode, just finished writing "Hi"):

  0            position = 2                                      capacity = 1024
  │                 │                                                   │
  ├─────────────────┼───────────────────────────────────────────────────┤
  │ H │ i │         unused buffer space (limit = capacity)              │
  └─────────────────┴───────────────────────────────────────────────────┘


AFTER flip() (read mode):

  0       position = 0      limit = 2                    capacity = 1024
  │            │                │                               │
  ├────────────┼────────────────┤───────────────────────────────┤
  │ H │ i │                    unused buffer space              │
  └────────────┴────────────────┴───────────────────────────────┘

position is RESET to 0.
limit marks the REAL data boundary.
As get() is called, position advances: 0 → 1 → 2.
```

**Why is `flip()` needed at all, rather than the buffer just "knowing" it's time to read?** The buffer has **no inherent concept** of "I'm done writing now" — from its perspective, `position` just tracks "how far writing has progressed so far." `flip()` is the **explicit, deliberate signal** that converts "how much I've written" (`position`) into "how much valid data exists to read" (the new `limit`), and resets the read cursor to the start. **Forgetting to call `flip()` before reading is, by a wide margin, the single most common NIO bug** — without it, you'd start "reading" from wherever writing happened to leave off, generally reading garbage/unused buffer space, or immediately finding `hasRemaining()` false since `position` already equals the old `limit`.

## `clear()`, `rewind()`, `compact()`

```java
buffer.clear();     // resets for a FRESH round of writing:
                       //   position = 0, limit = capacity  (does NOT actually erase old data --
                       //   just resets the bookkeeping so new writes will overwrite it)

buffer.rewind();      // resets position = 0 WITHOUT touching limit --
                        //   lets you RE-READ the same data from the beginning again

buffer.compact();       // for when you've PARTIALLY read a buffer and want to write MORE data
                          //   after what's left unread: shifts the UNREAD data to the beginning,
                          //   and sets position/limit correctly to continue writing after it
```

**Each of these solves a genuinely different real scenario**: `clear()` for "I'm completely done with this data, start fresh." `rewind()` for "let me process this same data again from the top." `compact()` for "I've consumed some of this data, but not all — preserve the leftover and let me add more."

## Real-World Analogy

Think of a `ByteBuffer` like a **whiteboard with a moving "current writing position" marker and a separate "boundary line" someone can draw**. While you're writing (filling the whiteboard left to right), the marker (`position`) tracks how far you've gotten, and there's no boundary line drawn yet (`limit` = the whiteboard's full edge, `capacity`). `flip()` is like **someone walking up and drawing a boundary line exactly where your marker currently is** ("everything up to here is what you actually wrote"), then moving the marker back to the very start, so a reader can now walk through exactly the content you wrote, and not one inch of blank whiteboard beyond it.

## Advantages

- The explicit capacity/position/limit model lets you write partial data, pause, resume, and re-read without the awkwardness of stream-based APIs — foundational to NIO's non-blocking capabilities (Topic 3).
- `compact()` specifically supports efficient handling of partially-consumed data — a genuinely important pattern for network protocols where messages can arrive in incomplete fragments.

## Disadvantages / Trade-offs

- The buffer API is genuinely less intuitive than `java.io` streams, with a real, well-known learning curve (forgetting `flip()` is an almost universal rite of passage for anyone learning NIO).
- More manual bookkeeping is required compared to the simpler, more automatic `java.io` model.

## Best Practices

- Always call `flip()` after writing to a buffer and before reading from it — make this an automatic habit.
- Use `hasRemaining()` (rather than manually comparing `position`/`limit`) to check whether more data is available to read.
- Use `compact()`, not `clear()`, when you need to preserve unread data while making room for more.

## Common Mistakes

- Forgetting to call `flip()` before reading — the single most common NIO bug, by a wide margin.
- Confusing `clear()` (reset for fresh writing, old data logically discarded) with `rewind()` (re-read the same data again) — they solve different problems.
- Assuming `clear()` actually erases the buffer's underlying data — it only resets the position/limit bookkeeping; the old bytes remain physically present until overwritten.

## Interview Questions

1. **Q: What does `Buffer.flip()` do, and why is it necessary?**
   A: It sets `limit = position` (marking exactly how much real data was written) and resets `position = 0` (so reading starts from the beginning). It's necessary because the buffer has no inherent way to know "writing is done, now read" — `flip()` is the explicit signal converting write-mode bookkeeping into read-mode bookkeeping.

2. **Q: What's the difference between `clear()` and `rewind()`?**
   A: `clear()` resets `position = 0` and `limit = capacity`, preparing the buffer for fresh writing (old data is logically discarded, though not physically erased). `rewind()` resets only `position = 0`, keeping `limit` unchanged, letting you re-read the same already-written data from the beginning again.

3. **Q: What problem does `compact()` solve?**
   A: When a buffer has been partially read (some data consumed, some remaining) and you need to add more data after what's left — `compact()` shifts the unread portion to the beginning of the buffer and adjusts position/limit so new data can be written after it, without losing the unconsumed remainder.

## Summary

- NIO's `Buffer` tracks **capacity** (fixed), **limit** (read/write boundary), and **position** (next read/write index), always satisfying `0 <= position <= limit <= capacity`.
- **`flip()`** switches a buffer from write mode to read mode (`limit = position`, `position = 0`) — the single most important, most commonly forgotten NIO operation.
- **`clear()`** resets for fresh writing; **`rewind()`** re-reads existing data; **`compact()`** preserves unread data while making room for more writing.