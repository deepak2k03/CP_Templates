# Lazy Segment Tree — Usage Guide

This guide explains three Lazy Segment Tree variants:

1. **Range Add + Range Sum**
2. **Range Add + Range Minimum**
3. **Range Flip + Range Sum** for a binary array

> **Important:** These are separate implementations. You cannot define multiple classes with the same name in one C++ file. Rename them (for example, `LazySumTree`, `LazyMinTree`, and `LazyFlipTree`) or keep them in separate files.

## 1. What is Lazy Propagation?

Lazy propagation postpones updates to child nodes until they are needed. It makes range updates efficient without updating every element individually.

For the examples below:
- Arrays are **0-indexed**.
- Ranges `[l, r]` are **inclusive**.
- Build the tree before processing queries.
- Build takes `O(n)` time; range updates and queries take `O(log n)` time; storage is `O(n)`.

## 2. Variant A: Range Add + Range Sum

Use this when you need to add a value to every element in a range and query the sum of a range.

Assuming the class is named `LazySumTree` and uses the method signatures from your code:

```cpp
vector<long long> arr = {1, 2, 3, 4, 5};
int n = (int)arr.size();

LazySumTree st(n);
st.build(0, 0, n - 1, arr);

// Sum of arr[1..3] = 2 + 3 + 4 = 9
cout << st.query(0, 0, n - 1, 1, 3) << '\\n';

// Add 10 to each element in indices 1..3.
// Logical array becomes {1, 12, 13, 14, 5}.
st.update(0, 0, n - 1, 1, 3, 10);

// Sum of arr[1..3] = 39
cout << st.query(0, 0, n - 1, 1, 3) << '\\n';
```

When a node represents `[low, high]`, adding `val` to every element increases its sum by:

```cpp
seg[ind] += (high - low + 1) * val;
```

A typical query-processing loop might look like this:

```cpp
if (type == 1) {
    // Add val to all elements in [l, r].
    st.update(0, 0, n - 1, l, r, val);
} else if (type == 2) {
    // Print sum of [l, r].
    cout << st.query(0, 0, n - 1, l, r) << '\\n';
}
```

Adapt the input parsing and query types to the exact problem statement.

## 3. Variant B: Range Add + Range Minimum

Use this when range additions are followed by minimum queries.

```cpp
vector<long long> arr = {5, 2, 7, 4, 9};
int n = (int)arr.size();

LazyMinTree st(n);
st.build(0, 0, n - 1, arr);

// Minimum of arr[1..3] is 2.
cout << st.query(0, 0, n - 1, 1, 3) << '\\n';

st.update(0, 0, n - 1, 1, 3, 3);
// Logical array becomes {5, 5, 10, 7, 9}.

// Minimum of arr[1..3] is now 5.
cout << st.query(0, 0, n - 1, 1, 3) << '\\n';
```

For a fully covered node, adding `val` increases its minimum by `val`:

```cpp
seg[ind] += val;
```

For a minimum query, the no-overlap result should be a neutral value such as `LLONG_MAX` when storing `long long` values. If your code uses `INT_MAX`, consider changing it to `LLONG_MAX`.

## 4. Variant C: Range Flip + Range Sum

Use this when the array is binary (`0` and `1`), a range update flips every bit, and a query asks for the number of ones.

```cpp
vector<long long> arr = {1, 0, 1, 1, 0};
int n = (int)arr.size();

LazyFlipTree st(n);
st.build(0, 0, n - 1, arr);

// Number of ones in arr[1..4] is 2.
cout << st.query(0, 0, n - 1, 1, 4) << '\\n';

// Flip indices 1..3: array becomes {1, 1, 0, 0, 0}.
st.update(0, 0, n - 1, 1, 3);

// Number of ones in arr[1..4] is now 1.
cout << st.query(0, 0, n - 1, 1, 4) << '\\n';
```

For a fully covered segment, the number of ones becomes:

```cpp
seg[ind] = (high - low + 1) - seg[ind];
```

**Important:** do not use `(high - low + 1) - val` here. A flip uses the current number of ones, not an update value. Since flipping twice cancels out, toggle the pending flip flag:

```cpp
lazy[child] = !lazy[child];
```

This implementation assumes every array element is either `0` or `1`. For a sum query with no overlap, return `0`.

## 5. Understanding the method arguments

A common recursive API looks like this:

```cpp
st.build(0, 0, n - 1, arr);
st.update(0, 0, n - 1, l, r, val);
st.query(0, 0, n - 1, l, r);
```

| Argument | Meaning |
|---|---|
| First `0` | Segment-tree array index of the root node |
| Second `0` | Left boundary of the original array segment |
| `n - 1` | Right boundary of the original array segment |
| `l`, `r` | Inclusive range to update or query |
| `val` | Value to add (for range-add variants only) |
| `arr` | Initial array used to build the tree |

Keep the calls aligned with the exact method signatures in your class.

## 6. Common mistakes

1. **Using the wrong variant.** A sum tree does not answer minimum queries correctly unless its stored summary and merge logic are changed.
2. **Mixing up inclusive ranges.** Segment length is `high - low + 1`.
3. **Using an ordinary addition for a flip.** A flip changes the count to `segment_length - current_count`.
4. **Duplicate class names.** Rename the three classes if they are in one file.
5. **Incorrect bounds.** For an array of length `n`, build over `[0, n - 1]`.
6. **Variable-length arrays.** In standard C++, prefer `vector<long long> arr(n)` over `long long arr[n]` when `n` is read at runtime.
7. **Wrong neutral value.** Use `0` for sum queries with no overlap and `LLONG_MAX` for minimum queries with no overlap when using `long long`.
8. **Non-binary input for the flip tree.** The flip logic only works for `0` and `1`.

## 7. Quick decision guide

| Required operations | Variant |
|---|---|
| Add to a range, query range sum | Range Add + Range Sum |
| Add to a range, query range minimum | Range Add + Range Minimum |
| Flip a range of bits, count ones | Range Flip + Range Sum |
| Only point updates and range queries | A normal Segment Tree may be simpler |

### Complexity summary

| Operation | Time |
|---|---:|
| Build | `O(n)` |
| Lazy range update | `O(log n)` |
| Range query | `O(log n)` |
| Space | `O(n)` |

**Remember:** lazy propagation is especially useful when updates affect a whole range. The stored summary (`sum`, `minimum`, or count of ones) determines how each update should modify a node.
