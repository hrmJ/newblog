---
title: "Conditional return types for ts functions"
datePublished: Fri Apr 29 2022 11:55:22 GMT+0000 (Coordinated Universal Time)
cuid: cl2kdoptl04ua24nv2c004z5c
slug: conditional-return-types-for-ts-functions
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/1sIz2NtVyI8/upload/v1651233113476/xQpf1qrWxf.jpeg
tags: typescript

---

Here's a simple TS problem: I want to define multiple possible return types
for a function based on the arguments of the function.

Initially, we might have a function like the following

```typescript
export function formatItem(input: number): string {
  /// ..... ////
  return "final result";
}
```

Later on, the need arises to sometimes return an array of strings instead of a
simple string. The return type is based on another function parameter:

```typescript
export function formatItem(input: number, asArray = false): string {
  /// ..... ////
  if (asArray) return ["final result"];
  return "final result";
}
```

So how do I annotate the return type since `:string` is no longer valid?
First place for me where I have used [function overloads](https://www.typescriptlang.org/docs/handbook/2/functions.html#function-overloads)

```typescript
export function formatItem(input: number, asArray?: false): string;
export function formatItem(input: number, asArray?: true): string[];
export function formatItem(input: number, asArray = false): string {
  /// ..... ////
  if (asArray) return ["final result"];
  return "final result";
}
```

A detail worth noting is that the order of the overload statements matters. Since `asArray = false` is the default here,
the return value caused by this should be marked first so that in `const result = formatItem(99)` the resulting variable
is correctly interpreted as a string.
