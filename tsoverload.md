---
title: Conditional return types put simply
slug: annotating-random-samples-in-r
tags: typescript, ts, overloading
cover: https://images.unsplash.com/photo-1526115540060-c0548a6a3096?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2070&q=80
date: 2018-05-30 08:25:08 UTC+03:00
---

Here's a simple TS problem: I want to define multiple possible return types
for a function based on the arguments of the function.

Initially, we might have a function like the following

```typescript
export function formatMinutes(durationInMinutes: number): string {
  /// ..... ////
  return "final result";
}
```

Later on, the need arises to sometimes return an array of strings instead of a
simple string. The return type is based on another function parameter:

```typescript
export function formatMinutes(
  durationInMinutes: number,
  asArray = false
): string {
  /// ..... ////
  if (asArray) return ["final result"];
  return "final result";
}
```

So how do I annotate the return type since `:string` is no longer valid?
First place for me where I have used [function overloads](https://www.typescriptlang.org/docs/handbook/2/functions.html#function-overloads)

```typescript
export function formatMinutes(
  durationInMinutes: number,
  asArray?: false
): string;
export function formatMinutes(
  durationInMinutes: number,
  asArray?: true
): string[];
export function formatMinutes(
  durationInMinutes: number,
  asArray = false
): string {
  /// ..... ////
  if (asArray) return ["final result"];
  return "final result";
}
```

A detail worth noting is that the order of the overload statements matters. Since `asArray = false` is the default here,
the return value caused by this should be marked first so that in `const result = formatMinutes(99)` the resulting variable
is correctly interpreted as a string.
