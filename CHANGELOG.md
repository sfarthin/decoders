v1.18.1
-------

- Also fix Flow type bugs when Flow option `exact_by_default=true` in `debrief`
  dependency


v1.18.0
-------

**New decoders:**

- `nonEmptyString`: like `string`, but will fail on inputs with only whitespace
  (or the empty string)

- `nonEmptyArray`: like `array`, but will fail on inputs with 0 elements


**Fixes:**

- Fix Flow type bugs when Flow option `exact_by_default=true` is enabled


v1.17.0
-------

May cause breakage for Flow users:

- Fix subtle bug in `object()` and `exact()` Flow type definitions that could
  cause Flow to leak `any` under rare circumstances.


v1.16.1
-------

- Internal change to make the code Flow 0.105.x compatible.  Basically stops
  using array spreads (`[...things]`) in favor of `Array.from()`.


v1.16.0
-------

**New feature:**

- Allow `map()` calls to throw an exception in the mapper function to reject
  the decoder.  Previously, these mapper functions were not expected to ever
  throw.


v1.15.0
-------

**New features:**

- Support constructors that have required arguments in `instanceOf` decoders in
  TypeScript (see #308, thanks @Jessidhia!)
- Add support for type predicates in `predicate()` in TypeScript (see #310,
  thanks @Jessidhia!)

**Fixes:**

- Add support for Flow >= 0.101.0


v1.14.0
-------

**Potential breaking changes:**

- Stricten `pojo` criteria.  Now, custom classes like `new String()` or
  `new Error()` will not be accepted as a plain old Javascript object (= pojo)
  anymore.

**Fixes:**

- Add support for Flow 0.98+


v1.13.1
-------

**Fixes:**

- Don't reject URLs that contains commas (`,`)


v1.13.0
-------

**Breaking changes:**

* Changed the API interface of `dispatch()`.  The previous version was too
  complicated and was hardly used. The new version is easier to use in
  practice, and has better type inference support in TypeScript and Flow.
  
  **Previous usage:**
  ```
  const shape = dispatch(
      field('type', string),
      type => {
          switch (type) {
              case 'rect': return rectangle;
              case 'circle': return circle;
          }
          return fail('Must be a valid shape');
      }
  );
  ```
  
  **New usage:**
  ```
  const shape = dispatch('type', { rectangle, circle });
  ```
  
  Where `rectangle` and `circle` are decoders of which exactly one will be
  invoked.
  
- Removed the `field()` decoder.  It was not generic enough to stay part of the
  standard decoder library. (It was typically used in combination with
  `dispatch()`, which now isn't needed anymore, see above.)

- `pojo` decoder now returns `Decoder<{[string]: mixed}>` instead of the unsafe
  `Decoder<Object>`.

**Fixes and cleanup:**

- Internal reorganization of modules
- Improve TypeScript support
  - Reorganization of TypeScript declarations
  - More robust test suite for TypeScript
  - 100% TypeScript test coverage


v1.12.0
-------

**New decoders:**

- `oneOf(['foo', 'bar'])` will return only values matching the given values
- `instanceOf(...)` will return only values that are instances of the given
  class. For example: `instanceOf(TypeError)`.


v1.11.1
-------

- Reduce bundle size for web builds
- New build system
- Cleaner package output


v1.11.0
-------

**Potentially breaking changes:**

- Decoders now all take `mixed` (TypeScript: `unknown`) arguments, instead of
  `any` 🎉 !  This ensures that the proper type refinements in the
  implementation of your decoder are made.  (See migration notes below.)
- Invalid dates (e.g. `new Date('not a date')`) won’t be considered valid by
  the `date` decoder anymore.

**New features:**

- `guard()` now takes a config option to control how to format error
  messages.  This is done via the `guard(..., { style: 'inline' })` parameter.

  - `'inline'`: echoes back the input value and inlines errors (default);
  - `'simple'`: just returns the decoder errors.  Useful for use in sensitive contexts.

- Export `$DecoderType` utility type so it can be used outside of the decoders
  library.

**Fixes:**

- Fixes for some TypeScript type definitions.
- Add missing documentation.

**Migration notes:**

If your decoder code breaks after upgrading to 1.11.0, please take the
following measures to upgrade:

1. If you wrote any custom decoders of this form yourself:
   
   ```javascript
   const mydecoder = (blob: any) => ...
   //                       ^^^ Decoder function taking `any`
   ```
   
   You should now convert those to:
   
   ```javascript
   const mydecoder = (blob: mixed) => ...
   //                       ^^^^^ Decoders should take `mixed` from now on
   ```
   
   Or, for TypeScript:
   
   ```javascript
   const mydecoder = (blob: unknown) => ...
   //                       ^^^^^^^ `unknown` for TypeScript
   ```
   
   Then follow and fix type errors that pop up because you were making
   assumptions that are now caught by the type checker.

2. If you wrote any decoders based on `predicate()`, you may have code like
   this:
   
   ```javascript
   const mydecoder: Decoder<string> = predicate(
     s => s.startsWith('x'),
     'Must start with "x"'
   );
   ```
   
   You'll have to change the explicit Decoder type of those to take two type
   arguments:
   
   ```javascript
   const mydecoder: Decoder<string, string> = predicate(
   //                               ^^^^^^ Provide the input type to predicate() decoders
     s => s.startsWith('x'),
     'Must start with "x"'
   );
   ```
   
   This now explicitly records that `predicate()` makes assumptions about its
   input type—previously this wasn’t get caught correctly.


v1.10.6
-------

- Make Flow 0.85 compatible


v1.10.5
-------

- Update to latest debrief (which fixes a TypeScript bug)


v1.10.4
-------

- Drop dependency on babel-runtime to reduce bundle size


v1.10.3
-------

- Fix minor declaration issue in TypeScript definitions


v1.10.2
-------

- Tuple decoder error messages now show decoder errors in all positions, not
  just the first occurrence.

**New decoders:**

- New tuple decoders: `tuple3`, `tuple4`, `tuple5`, and `tuple6`
- `unknown` decoder is an alias of `mixed`, which may be more recognizable for
  TypeScript users.


v1.10.1
-------

- TypeScript support


v1.10.0
-------

**Breaking:**

- Private helper function `undefined_or_null` was accidentally exported in the
  package.  This is a private API.


v1.9.0
------
**New decoder:**

- `dict()` is like `mapping()`, but will return an object rather than a Map
  instance, which may be more convenient to handle in most cases.

**Breaking:**

- `optional(..., /* allowNull */ true)` has been removed (was deprecated since
  1.8.3)


v1.8.3
------
**New decoder:**

- `maybe()` is like `optional(nullable(...))`, i.e. returns a "maybe type".

**Deprecation warning:**

- `optional(..., /* allowNull */ true)` is now deprecated in favor of `maybe(...)`


v1.8.2
------
**Improved error reporting:**

- Fix bug where empty error branches could be shown in complex either
  expressions (fixes #83)


v1.8.1
------
- Fix: revert accidentally emitting $ReadOnlyArray types in array decoders


v1.8.0
------
- Drop support for Node 7
- Declare inputted arrays will not be modified (treated as read-only)


v1.7.0
------
- Make decoders fully [Flow Strict](https://flow.org/en/docs/strict/)
  compatible


v1.6.1
------
- Upgraded debrief dependency
- Behave better in projects that have Flow's `experimental.const_params`
  setting turned on


v1.6.0
------
- **New decoders!**
  - `exact()` is like `object()`, but will fail if the inputted object contains
    superfluous keys (keys that aren't in the object definition).


v1.5.0
------
- Collect and report all nested errors in an object() at once (rather than
  error on the first error).

**Breaking:**

- Remove deprecated `message` argument to `object()`


v1.4.6
------
- Add missing documentation


v1.4.5
------
- Upgrade second-level dependencies


v1.4.4
------
- Declare library to be side effect free (to help optimize webpack v4 builds)
- Upgrade dependencies


v1.4.1
------
- Improve internals of the error message serializer (debrief)


v1.4.0
------
- **New decoders!**
  - `email` validator based on the [almost perfect email regex](http://emailregex.com/)
  - `url` validator for validating HTTPS URLs (most common use case)
  - `anyUrl` validator for validating any URL scheme


v1.3.1
------
- Fix bug where dates, or arrays (or any other Object subclass) could pass for
  a record with merely optional fields.


v1.3.0
------
- Much improved error messages!  They were redesigned to look great in
  terminals and to summarize only the relevant bits of the error message,
  striking a balance between all the details and the high level summary.


v1.2.4
------
- **New features**:
  - `truthy` takes any input and returns whether the value is truthy
  - `numericBoolean` takes only numbers as input, and returns their boolean
    interpretation (0 = false, non-0 = true)


v1.2.2, v1.2.3
--------------
- **New feature** `mixed` decoder, for unverified pass-thru of any values


v1.2.1
------
- **Fix** Expose the following decoders publicly:
  - `integer`
  - `positiveInteger`
  - `positiveNumber`


v1.2.0
------
- **New feature** `regex()`, for building custom string decoders
- Tiny tweaks to improve error messages (more structural improvements are on
  the roadmap)


v1.1.0
------
- Expose pojo() decoder, for plain old objects (with mixed contents)
- Expose poja() decoder, for plain old arrays (with mixed contents)
- Perf: make `tuple2()` decoder lazier


v1.0.1
------
- Expose new "either" decoders at the too level


v1.0.0
------
- **BREAKING** Removes the old public ("compat") API
- Finalize/settle on public API


v0.1.3
------
- Add whole series for either, either3, either4, ..., either9
- Updated dev dependencies


v0.1.2
------
- Add `date` decoder, which decodes `Date` instances
- Improve error output detail when throwing errors


v0.1.1
------
- Export `g2d()` helper function that can help adoption to new-style APIs by
  converting old-style decoders (now called guards) to new-style decoders.


v0.1.0
------
- **Breaking change** New API: simplified names, split up decoders from guards.
  What used to be called "decoders" in 0.0.x ("things that either return
  a value or throw a runtime error") are now called "guards" in 0.1.0.
  The meaning of the term "decoders" is now changed to a thing that either is
  an "Ok" value or an "Err" value.

  To convert to the new API, do this:

  ```javascript
  // Old way
  import { decodeNumber, decodeObject, decodeString } from 'decoders';

  const decoder = decodeObject({
      name: decodeString(),
      age: decodeNumber(),
  });

  // -------------------------------------------------------------------

  // New way
  import { guard, number, object, string } from 'decoders';

  const guard = guard(object({
      name: string,
      age: number,
  }));
  ```

