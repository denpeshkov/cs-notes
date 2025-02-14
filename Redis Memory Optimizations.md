---
tags:
  - Distributed-Systems
---

# Memory Optimization Using Listpack

Hashes, Lists, Sets, and Sorted Sets, when smaller than a given number of elements, and up to a maximum element size, are encoded using a memory-efficient `listpack`. When they exceed the configured max size, Redis automatically converts them into normal encoding

## Representation

```
<tot-bytes> <num-elements> <element-1> ... <element-N> <listpack-end-byte>
```

- `tot-bytes`: The total amount of bytes representing the `listpack` including the header itself and the terminator. This basically is the total size of the allocation needed to hold the `listpack`
- `num-elements`: The total number of elements in the `listpack`
- `listpack-end-byte`: Terminator that marks the end of the `lipstick`

Each element in a `listpack` has the following structure:

```
<encoding-type><element-data><element-tot-len>
```

- `encoding-type`: The type of element data, as strings can be encoded as little-endian integers
- `element-data`: The data itself, like an integer or an array of bytes representing a string
- `element-tot-len`: Element total length, is used in order to traverse the list backward from the end of the `listpack` to its head

The `encoding-type` and `element-tot-len` are always present. The `element-data` itself sometimes is missing, since certain small elements are directly represented inside the spare bits of the encoding type

# References

- [Introduction to Running Redis at Scale](https://redis.io/learn/operate/redis-at-scale)
- [Redis Deep Dive w/ a Ex-Meta Senior Manager - YouTube](https://youtu.be/fmT5nlEkl3U?si=HnngxAjzvNyi466b)
- [Memory optimization \| Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/)
- [Redis: OBJECT ENCODING \| Docs](https://redis.io/docs/latest/commands/object-encoding/)
- [listpack/listpack.md at master · antirez/listpack · GitHub](https://github.com/antirez/listpack/blob/master/listpack.md)
