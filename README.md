

### Design overview

This section is a non-normative short summary for easy digestion. See [**Detailed Design**](#detailed-design) below for normative text.


#### For package consumers

Things a package may do in a non-breaking way:

- widen what it accepts from you
- narrow what it provides to you
- add new exports

Things which constitute breaking changes in a package:

- narrowing what it accepts from you
- widening what it provides to you
- removing exports

Note that this summary elides *many* important details, and those details may surprise you! In particular "what it accepts" and "what it provides" have considerable depth and nuance: they include interfaces or types you can construct, function arguments, class field mutability, and more.


#### Reasons for Breaking Changes

Each of the kinds of breaking changes defined below will trigger a compiler error for consumers, surfacing the error. As such, they should be easily detectable by testing infrastructure (see below under [Tooling: Detect breaking changes in types](#detect-breaking-changes-in-types)).

There are several reasons why breaking changes may occur:

1.  The author of the package may choose to change the API for whatever reason. This is no different than the situation today for packages which do not support TypeScript. This would be a major version independent of types.

2.  The author of the package may need to make changes to adapt to changes in the JavaScript ecosystem, for example to support Octane idioms in Ember.js. This is likewise identical with the situation for packages which do not support TypeScript: it would require a major version regardless.

3.  Adopting a new version of TypeScript may change the meaning of existing types. For example, in TypeScript 3.5, generic types without a specified default type changed their default value from `{}` to `unknown`. This improved type safety, but broke many existing types, as [described in detail by Google][3.5-breakage].

4.  Adopting a new version of TypeScript may change the type definitions emitted in `.d.ts` files in backwards-incompatible ways. For example, changing to use the finalized ECMAScript spec for class fields meant that [types emitted by TypeScript 3.7 were incompatible with TypeScript 3.5 and earlier][3.7-emit-change].

[3.5-breakage]: https://github.com/microsoft/TypeScript/issues/33272
[3.7-emit-change]: https://github.com/microsoft/TypeScript/pull/33470

The kinds of breaking changes represented by reasons (1) and (2) are described below under [**Changes to Types**](#changes-to-types); reasons (3) and (4) are discussed below in [**Compiler Considerations**](#compiler-considerations).

Additionally, there are some changes which we define *not* to be breaking changes because, while they will cause the compiler to produce a type error, they will do so in a way which simply allows the removal of now-defunct code.

### Changes to types

#### Variance

Virtually all of the rules around what constitutes a breaking change to types come down to *variance*.[^variance] In a real sense, everything in the discussion below is a way of showing the variance rules by example.[^thanks-to-ryan]



In many cases, these are the standard variance rules applicable in any and all languages with types:

- read-only types (sources) may be covariant
- write-only types (sinks) may be contravariant
- read-write (mutable) types must be invariant

These basic intuitions underlie the guidelines below. However, several factors complicate them.

First of all, notice that the vast majority of objects in JavaScript are mutable, which means they must be *invariant*. When combined with type inference, this effectively means that *any* change to an object type *can* cause breakage for consumers. (The only real counter-examples are `readonly` types.)

Additionally, TypeScript has two other features many other languages do not which complicate reasoning about variance: *structural typing*, *higher-order type operations*. The result of these additional features is a further impossibility of safely writing types which can be *guaranteed* never to stop compiling for runtime-safe changes.

Accordingly, we propose the rules below, with the caveat that (as noted in several places throughout) they will *not* prevent all possible breakage—only the majority of it, and substantially the worst of it. Most of all, they give us a workable approach which can be well-tested and well-understood, and the edge cases identified here do not prevent the rules from being generally useful or applicable.[^satisficery]


For a more detailed explanation and analysis of the impact of variance on these rules, see [**Appendix C**](#appendix-c-on-variance-in-typescript).

#### Breaking Changes

##### Symbols

Changing a symbol is a breaking change when:

-   changing the name of an exported symbol (type or value), since users' existing imports will need to be updated. This is breaking for value exports (`let`, `const`, `class`, `function`) independent of types, but renaming exported `interface`, `type` alias, or `namespace` declarations is breaking as well.

-   removing an exported symbol, since users' existing imports will stop working. This is a breaking change for value exports (`let`, `const`, `class`, `function`) independent of types, but removing exported `interface`, `type` alias, or `namespace` declarations is breaking as well.

    This includes changing a previously type-and-value export such as `export class` to either—
    
    -   a type-only export, since the exported value symbol has been removed:

        ```diff
        -export class Foo {
        +class Foo {
          neato: string;
        }
        +
        +export type { Foo };
        ```
    
    -   a value-only export, since the exported type symbol has been removed:

        ```diff
        -export class Foo {
        +class _Foo {
          neato: string;
        }
        +
        +export let Foo: typeof _Foo;
        ```

-   changing the kind (*value* vs. *type*) of an exported symbol in any way, since users' imports and own definitions may both be broken, since imports resolve all symbols imported together if they share a name:

    -   Given a *value*-only exported symbol, including `namespace` declarations, adding a *type* export with the same name as the *value* may break users' code: they may have imported the value and safely created a type of the same name. Their existing import will now cause a re-declaration conflict. Note that this is distinct from adding an entirely new type export where there was no type or value export previously, since the user could never accidentally introduce the conflict, and could work around the conflict using the `as` import specifier when introducing the import.

    -   Given a *type*-only exported symbol, including `type`, `interface`, or `export type` for a type or value, adding a *value* export with the same name may break users' code: they may have imported the type and safely created a value of the same name. Their existing import will now cause a re-declaration conflict. Note that this is distinct from adding an entirely new value export where there was no type or value export previously, since the user could never accidentally introduce the conflict, and could work around the conflict using the `as` import specifier when introducing the import. (Type-only imports via `import type` do not change this because they still import the symbol into value space to use with `typeof`, e.g. to get a class' constructor.)

    -   Given a `namespace` export, changing it to a value-only export (that is, to an exported object) will break all nested type access, since types cannot be exported as nested members of non-`namespace` values. (`namespace` exports cannot be directly converted to type-only exports.)

-   changing an `interface` to a `type` alias will break any user code which used interface merging

-   changing a `namespace` export to any other type will break any code which used namespace merging

-   changing a `class` export to a type-only export will break any code which extended the class or constructed an instance of the class ([playground][class-to-type-only]), and changing a `class` export to a value-only export will break any code which referred to the class as a type ([playground][class-to-value-only])

[class-to-type-only]: https://www.typescriptlang.org/play?#code/PTAEEEDtQIgewE4EsDmTIEMA2NQBMBTAYywwQwBck5IAaUAdwAsCEDQKXGm4t2SMAZ0GMhoAgA8ADogoE8AOgBQhAW1ADhoAOpCAotNnzQAbwC+SpSFAAVJkhEOOXAgEcArkgBu2ApAqgcABmoBgcAJ5SBAC0NFjh4oYIAcGhGqTC9MxIREyMcO5YeKCQiAC22PFWYABG7AzIFHLQWEgA1uycDgBc1aB9AAZDfZoiuoIGMsnG5n2SUwEUkewmOvpJcsUW1kMDSqOgAPoAcnAUk0bFs0tRoKfnG8YAvEf3F9N4ANyWBwDK7jVRgAxRDvTaJZp4MbrBYzCx-AHAxBvR7FSSQkQo2FXCwqYikdRBdyQIhUGgcDAdQTjMHyAAUDBhl26awmqIAlCyvHAkF88Wp2ESSWToBRKQRBFjLgymR8WVKPpzQNzed99jRBAFGWzsaAXpACAxWbS8HT2d8iBqAqUHrr9Ya7mcTWbPkA

[class-to-value-only]: https://www.typescriptlang.org/play?#code/PTAEEEDtQIgewE4EsDmTIEMA2NQBMBTAYywwQwBck5IAaUAdwAsCEDQKXGm4t2SMAZ0GMhoAgA8ADogoE8AOgBQhAW1ADhoAOpCAotNnzQAbwC+SpSFAAVJkhEOOXAgEcArkgBu2ApAqgcABmoBigPljuBAC0NFgAnuKGCAHBoRqkwgqgAHKIALbYCaDxcO6MZVh4VmAMyHLOTgxInKAABpIyKaB8FG2g5JyszhjQbaqk6r1typoiAPp5FAZdcnimFhNk7L25cMvJawBcHPFSBGmL+ytGeADclnOgAMruAEZzAGKINynGknJIHgRLpBL81hslE9Xh9MoJvggluD-hJAcC9gdVsZzJYtuogu5IEQqDQOBgANYEQSg5F4AAUDH0h3kJxpzLwAEoTl44Eh7ipiJN2ASiSToBQKVSkeyGUysXgTtL5Vzwrz+VCaIIAoywezQABeUCQAgMHRy250jkPIiagKQa56w3G01Ki1WoA


##### Interfaces, Type Aliases, and Classes

Object types may be defined with interfaces, type aliases, or classes. Interfaces and type aliases define type symbols only. Classes define both type symbols and value symbols. The `namespace` construct defines a value symbol (as well as introducing a context in which you can name other nested type or value symbols). The additional constraints for the value symbols introduced by classes are covered above under [Breaking Changes: Symbols](#symbols).

A change to any object type (user constructible or not) is breaking when:

-   a non-`readonly` object property's type changes in any way:

    -   if it was previously `string` but now is `string | number`, some of the user's existing *reads* of the property will now be wrong ([playground][reads-of-property]). Note that this includes making a property optional.

    -   if it was previously `string | number` but now is `string`, some of the user's existing *writes* to the property will now be wrong ([playground][writes-to-property]). Note that this includes making a previously-optional property required. 

        Note that at present, TypeScript cannot actually catch all variants of this error. [This playground][writes-to-property] demonstrates that there is a runtime error but no *type* error in one scenario. TypeScript's type system understands these types in terms of *assignability*, rather than local *mutability*. However, package authors should test for the catchable variant of this condition.

-   a property is removed from the type entirely, since some of the user's existing uses of the type will break, even if the property was optional ([optional][optional-removed], [required][required-removed])

A change to a user-constructible type is breaking when:

-   a required property is added to the type, since all of the user's existing constructions of the type will be incorrect ([new-required-property][required-added])

A change to a non-user-constructible object type is breaking when:

-   a `readonly` object property type becomes a *less specific ("wider") type*, for example if it was previously `string` but now is `string | string[]`—since the user's existing handling of the property will be wrong in some cases ([playground][wider-property]—the playground uses a class but an interface or type alias would have the same behavior).

    Note that this includes making a property optional, since these are equivalent for the purposes of type-checking:

    ```diff
    interface A {
    - a: string;
    + a?: string;
    }
    ```

    ```diff
    interface A {
    - a: string;
    + a: string | undefined;
    }
    ```

[reads-of-property]: https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgOrADYfQEwiZAbwFgAoZC5OALmQGcwpQBzAbjIF8yzRJZEUAEWA5c+ImUpVaDJiGbIAPshABXALYAjaO1JdSZGKpAIwwAPYEAFnBA4MEOuixiQACgBucDKoi1n2CL4AJS0alrQEuSUUBBgqlAEXj4QAHRwqQ7yYFa6+mR4CBhwscgOYMgA7piBeCD+Na66NnYOTo1B7tUuncG6BRBFJSjlyDgirrTCop3NtvaOAa5u4zN1fWRAA

[writes-to-property]: https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgOrADYYHJylAewHdkBvAWAChkbk4AuZAZzClAHNkAfZEAVwC2AI2gBuKgF8qVUJFiIUAEWAATXPmJkqtOo37Cxk6ZRh8QCMMAIhkRNpCYAVAuizrCRABQA3OBj4QjK44eB4AlFrUtL7+EAB0cMgAvMgA5AAWEFgEqeKUUpRUKhAIGHgoGBBgtpghGkRBte7EeXbADs7BzV5ETaHEYXlFJWVQFVXIKqrdjMpq-UR5U-P1CcnIAEQADgRgcGAEG6LIAPQnyPCYTFRtHS599Z7L3YOn50x8CEgQKkzI0IQQBACHwmBgAJ5UIA

[optional-removed]: https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgEIHU4GcDyAHMYAexDgBtkBvAWAChkHk4AuZAIyKLIjhAG46jdgH5WHLj350AvnTqhIsRClQAlCAFsiANwgATKoMYt2nbrwG1ZtOtzDISZAJ4BBVBBhEoEVhmz5CEnJkAF4qJlYwKABXFGlLO2RQBDJovX1Ud09vX0xcAmJSCjDKCOQo2IAadlZ4Miw4y1sIe0dXFxhFX3UtXQMSsorG5vtk1PS9VA6utB6dfVDwkyHqtlryBuR4oA

[required-removed]: https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgEIHU4GcBKECOArsFBACbIDeAsAFDIPJwBcyARgPYcA2EcIAbjqN2rTjz6C6AXzp1QkWIhSo8AWw4A3clWGMW7Lr35Das2nV5hkoBN0JlyqVBBgdSrDNjxESOgLxUTKxgUIQQADSiyPDcWCjSppYQ1rb2jmSoAIIwip7qWgFBBqHhUWyssfHIiUA

[required-added]: https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgEIRgeyig3gWAChkTk4AuZAI00wBsI4QBuIgXyKNElkRQEEYPZAWKkK1WgyasxJKpRABXALZVosjoSIg4KiAGcADnzQZsEAKpGAJnEgiipZAzDIEmEAbBQlCMMCelOhYOMgAvCJklD5KKGyanIS6+samgjzWdg6izq7unt6+-oEglBnQEVESsfGJhEA

[wider-property]: https://www.typescriptlang.org/play/#code/CYUwxgNghgTiAEkoGdnwPIwJYHMsDsoJ4BvAKHnjimAHt8IBPeABxlpZBgBdGAueMm7Z8OANxkAvmTKgkcRNFTwA6llD4QwUhSoga9Jq3ace-QcII54AHwsicAbQC6UmQDMArvjDcs9eAALKHxgCBBkTFwCIgA5WHYAdwAKADciTxABKLxCCABKAXxPAFsAIy4dSjhuTxh8eHSITIA6Ng4uXhbw0W5AiWlZcGgFcO54KGzsXKIJORGEMfgygTUNLQlg0PDI6ZiIeJgk5Kh8zZCwiJz9w+Oys7IgA


### Conformance

To conform to this standard, a package must:

- link to the final published version of this specification
- specify the compiler support policy
- specify the currently-supported versions of TypeScript
- specify the definition of “public API” used by the library (e.g. “only documented types” vs. “all published types” etc.)
- author and publish its types with `strict: true` and `noUncheckedIndexedAccess: true` in its compiler configuration



## Notes

[^other-structural-types]: Many languages include structural typing in certain contexts, including Swift's protocols, Elm's record types, and row-polymorphic types in OCaml, PureScript, etc. However, of these only Elm provides language-level guidance or tooling, and at the time of authoring there is no public specification of its behavior. Its current algorithm is [implementations-specified][elm-compat] and roughly checks for addition or removal of fields.

[elm-compat]: https://github.com/elm/compiler/blob/770071accf791e8171440709effe71e78a9ab37c/builder/src/Deps/Diff.hs#L128-L136

[^re-export-antipattern]: In general, it is an antipattern for one package to re-export another package directly like this, and the cases where it makes sense (e.g. Ember re-exporting Glimmer APIs) are cases of collaborators which can manage this.

[^variance]: For the purposes of this discussion, I will *assume* knowledge of variance, rather than explaining it.

[^thanks-to-ryan]: Thanks to [Ryan Cavanaugh](https://github.com/RyanCavanaugh) of the TypeScript team for pointing out the various examples which motivated this discussion.

[^satisficery]: Precisely because SemVer is a *sociological* and not only a *technical* contract, the problem is tractable: We define a breaking change as above, and accept the reality that some changes are not preventable (but may in many cases be mitigated or fixed automatically). This is admittedly unsatisfying, but we believe it [satisfices](https://www.merriam-webster.com/dictionary/satisfice) our constraints.

[^nit-on-comparability]: Strictly speaking, one value may stop being comparable to another value in this scenario. However, this is both a rare edge case and fits under the standard rule where changes which simply let users delete now-defunct code are allowed.

[^variance-in-discriminating]: Mostly, anyway—see [Appendix C: On Variance in TypeScript – Higher-order type operations](#higher-order-type-operations) below for a discussion of how the `in` operator (or similar operations discriminating between unions) makes this cause breakage in some cases. These are rare enough, and easily-enough solved, that the rule remains workable.

[^const-controversy]: Rightly so, in my opinion!
