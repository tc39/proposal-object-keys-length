# ECMAScript Proposal: `Object.keysLength`

Note: This proposal was split out of the broader `Object Property Counting` effort.
For the wider discussion and a more general API that counts different kinds of own properties, see the parent proposal: https://github.com/tc39/proposal-object-property-count

## Status

- Stage: 2
- Champions: [Ruben Bridgewater](@BridgeAR), [Jordan Harband](@ljharb)
- Authors: [Ruben Bridgewater](@BridgeAR), [Jordan Harband](@ljharb)

You can browse the spec text at https://tc39.es/proposal-object-keys-length/ or view the [source](./spec.emu).

## Overview

`Object.keysLength(target)` returns the number of an object’s own enumerable string-keyed properties — precisely the same set of properties returned by `Object.keys(target)`, but without allocating the intermediate array. In other words:

```js
Object.keysLength(obj) === Object.keys(obj).length;
```

This narrowly scoped API addresses the single most common “count properties” pattern in the wild: `Object.keys(obj).length`.

## Motivation

Developers frequently write `Object.keys(obj).length` to determine how many own enumerable string-keyed properties an object has. While clear, this forces the engine to create an array of keys only to immediately take its `.length`, incurring avoidable allocation and garbage-collection overhead. This cost can be significant on hot paths and in frameworks and libraries that perform this check repeatedly.

Providing a built-in that returns the same information without producing an intermediate array:

- avoids needless allocation and GC pressure,
- makes intent explicit and readable, and
- enables engines to optimize the common case directly.

This proposal intentionally focuses on the exact semantics of `Object.keys` (own, enumerable, string-keyed properties). Broader counting needs (e.g., including symbols or non-enumerables, or choosing among key types) are covered by the parent proposal, [`Object.propertyCount`].

## Proposed API

```js
Object.keysLength(target)
```

- Parameters: `target` — coerced with `ToObject` (like `Object.keys`). Passing `null` or `undefined` throws a `TypeError`.
- Return value: a non-negative integer equal to `Object.keys(target).length`.

See [the spec](./spec.emu) for the precise algorithm.

## Examples

Basic objects:

```js
Object.keysLength({}); // 0
Object.keysLength({ a: 1, b: 2 }); // 2
Object.keysLength(/./.exec('a')); // 4 ('0', 'index', 'input', 'groups')
```

Objects without a prototype:

```js
const o = { __proto__: null };
o.x = 1;
Object.keysLength(o); // 1
```

Arrays and sparse arrays (counts present indices only):

```js
Object.keysLength([, , 3]); // 1 (only index "2" exists)
const a = [];
a[10] = 'x';
Object.keysLength(a); // 1
```

Symbols are not counted (matching `Object.keys`):

```js
const s = Symbol('s');
const obj = { a: 1, [s]: 2 };
Object.keysLength(obj); // 1
```

Non-enumerable properties are not counted:

```js
const obj = { a: 1 };
Object.defineProperty(obj, 'hidden', { value: true, enumerable: false });
Object.keysLength(obj); // 1
Object.keysLength([]); // 0
```

Primitives are coerced like `Object.keys`:

```js
Object.keysLength('abc'); // 3 (indices "0","1","2")
Object.keysLength(42); // 0
```

Error cases:

```js
Object.keysLength(null); // throws TypeError
Object.keysLength(undefined); // throws TypeError
```

## Relationship to `Object.propertyCount`

`Object.keysLength` is a focused subset of the larger `Object.propertyCount` proposal. While `Object.keysLength` is fixed to match `Object.keys` semantics (own, enumerable, string-keyed properties), `Object.propertyCount` explores a flexible API that can also count symbols and/or non-enumerables via options. If your use case needs those capabilities, see the parent proposal: https://github.com/tc39/proposal-object-property-count.

## Prior art and usage

The pattern `Object.keys(obj).length` is ubiquitous in production codebases across frameworks and libraries (e.g., React, Angular, Vue, Lodash, Node.js, Storybook, VS Code, and many more). This API makes that common intent fast without changing semantics.

## Links

- Spec text: https://tc39.es/proposal-object-keys-length/
- Spec source: [spec.emu](./spec.emu)
- Parent proposal: https://github.com/tc39/proposal-object-property-count

[`Object.propertyCount`]: https://github.com/tc39/proposal-object-property-count