## What is it?

Standard Schema is a standard interface designed to be implemented by all JavaScript and TypeScript schema libraries. 

The goal is to make it easier for other frameworks and libraries to accept user-defined schemas, without needing to implement a custom adapter for each schema library. 

Because Standard Schema is a *specification*, they can do so with no additional runtime dependencies.

## Implementers

The following libraries have implemented the Standard Schema interface.



<svg viewBox="0 0 48 48" role="img" aria-label="Valibot icon" class="mr-2 h-8 shrink-0 md:h-9 lg:mr-3 lg:h-10" q:key="ER_0"><defs><linearGradient id="nCrZ" x1=".41" x2="0" y1=".26" y2=".93" gradientUnits="objectBoundingBox"><stop offset="0" stop-color="#eab308"></stop><stop offset="1" stop-color="#ca8a04"></stop></linearGradient><linearGradient id="jgAy" x1=".34" x2=".66" y1=".02" y2=".97" gradientUnits="objectBoundingBox"><stop offset="0" stop-color="#fde68a"></stop><stop offset="1" stop-color="#fbbf24"></stop></linearGradient><linearGradient id="YpWK" x2="1" y1=".5" y2=".5" gradientUnits="objectBoundingBox"><stop offset="0" stop-color="#7dd3fc"></stop><stop offset="1" stop-color="#0ea5e9"></stop></linearGradient></defs><path fill="url(#nCrZ)" d="M629.38 987.02c-6.26 0-11.17 5.13-11.43 11.86l-.24 8.95c-.37 7.37 6.75 9.89 11.9 9.89Z" transform="translate(-615.34 -978.37)"></path><path fill="url(#jgAy)" d="M8.68 0h21.3a9 9 0 0 1 9.23 8.75l.58 12.73c.07 6.31-4.51 8.9-9.8 8.94l-21.31.27c-5.49.04-8.78-4.1-8.68-9.21L.35 8.75C.7 3.15 3.13.1 8.69 0Z" transform="translate(5.85 8.65)"></path><path fill="#111827" d="M15.73 9.63h19.98a8.4 8.4 0 0 1 8.65 8.14l.54 11.84c.06 5.88-4.23 8.28-9.19 8.32l-19.98.25c-5.14.04-8.24-3.81-8.14-8.57l.34-11.84c.31-5.21 2.59-8.04 7.8-8.14Z"></path><path fill="url(#YpWK)" d="M2.59 0A2.59 2.59 0 1 1 0 2.59 2.59 2.59 0 0 1 2.59 0Z" transform="translate(34.23 19.25)"></path><path fill="url(#YpWK)" d="M2.59 0A2.59 2.59 0 1 1 0 2.59 2.59 2.59 0 0 1 2.59 0Z" transform="translate(14.25 19.25)"></path></svg>



The 

A consortium of schema library authors have collaborated to craft a standard interface for schema libraries to benefit the entire JavaScript ecosystem. Standard Schema provides third-party libraries a uniform integration to automatically support multiple schema libraries at once, without adding a single runtime dependency. This simplifies implementation, prevents vendor lock-in, and enables innovation, especially for smaller schema libraries with new ideas. For more information on the origins and use cases of Standard Schema, see [background](#background).

## The Interface

The Standard Schema interface is a set of validation-related properties that must be defined under a key called `~standard`.

```ts
export interface StandardSchemaV1<Input = unknown, Output = Input> {
  readonly "~standard": {
    /**
     * The version number of the standard.
     */
    readonly version: 1;
    /**
     * The vendor name of the schema library.
     */
    readonly vendor: string;
    /**
     * Validates unknown input values.
     */
    readonly validate: (
      value: unknown,
    ) => Result<Output> | Promise<Result<Output>>;
    /**
     * Inferred types associated with the schema.
     */
    readonly types?: Types<Input, Output> | undefined;
  };
}
```

- `~standard` contains the Standard Schema properties and can be used to test whether an object is a Standard Schema. 
  - `version` defines the version number of the standard. This can be used in the future to distinguish between different versions of the standard.
  - `vendor` stores the name of the schema library. This can be useful for performing vendor-specific operations in special cases. 
  - `validate` is a function that validates unknown input and returns the output of the schema if the input is valid or an array of issues otherwise. This can be discriminated by checking whether the `issues` property is `undefined`.
  - `types` is used to associate type metadata with the schema. This property should be declared on the schema's type, but is not required to exist at runtime. Authors implementing a schema using a class are encouraged to use TypeScript's `declare` keyword or other means to avoid runtime overhead. `InferInput` and `InferOutput` can be used to extract their corresponding types.

## Implementation

Two parties are required for Standard Schema to work. First, the schema libraries that implement the standard interface, and second, the third-party libraries that accept schemas as part of their API that follow the standard interface.

### Schema Library

Schemas libraries that want to support Standard Schema must implement its interface. This includes adding the `~standard` property. To make this process easier, schema libraries can optionally extend their interface from the `StandardSchemaV1` interface.

> It doesn't matter whether your schema library returns plain objects, functions, or class instances. The only thing that matters is that the `~standard` property is defined somehow.

```ts
import type { StandardSchemaV1 } from "@standard-schema/spec";

// Step 1: Define the schema interface
interface StringSchema extends StandardSchemaV1<string> {
  type: "string";
  message: string;
}

// Step 2: Implement the schema interface
function string(message: string = "Invalid type"): StringSchema {
  return {
    type: "string",
    message,
    "~standard": {
      version: 1,
      vendor: "valizod",
      validate(value) {
        return typeof value === "string"
          ? { value }
          : { issues: [{ message }] };
      },
    },
  };
}
```

Instead of implementing the `StandardSchemaV1` interface natively into your library code, you can also just add it on top and reuse your existing functions and methods within the `validate` function.

### Third Party

Other than for schema library authors, we recommend third party authors to install the `@standard-schema/spec` package when implementing Standard Schema into their libraries. This package provides the `StandardSchemaV1` interface and the `InferInput` and `InferOutput` utility types.

```sh
npm install @standard-schema/spec --save-dev  # npm
yarn add @standard-schema/spec --dev          # yarn
pnpm add @standard-schema/spec --dev          # pnpm
bun add @standard-schema/spec --dev           # bun
deno add jsr:@standard-schema/spec --dev      # deno
```

> Alternatively, you can also copy and paste [the types](https://github.com/standard-schema/standard-schema/blob/main/packages/spec/src/index.ts) into your project.

After that you can accept any schemas that implement the Standard Schema interface as part of your API. We recommend using a generic that extends the `StandardSchemaV1` interface in most cases to be able to infer the type information of the schema.

```ts
import type { StandardSchemaV1 } from "@standard-schema/spec";

// Step 1: Define the schema generic
function createEndpoint<TSchema extends StandardSchemaV1, TOutput>(
  // Step 2: Use the generic to accept a schema
  schema: TSchema,
  // Step 3: Infer the output type from the generic
  handler: (data: StandardSchemaV1.InferOutput<TSchema>) => Promise<TOutput>,
) {
  return async (data: unknown) => {
    // Step 4: Use the schema to validate data
    const result = await schema["~standard"].validate(data);

    // Step 5: Process the validation result
    if (result.issues) {
      throw new Error(result.issues[0].message ?? "Validation failed");
    }
    return handler(result.value);
  };
}
```

#### Common Tasks

There are two common tasks that third-party libraries perform after validation fails. The first is to flatten the issues by creating a dot path to more easily associate the issues with the input data. This is commonly used in form libraries. The second is to throw an error that contains all the issue information.

##### Get Dot Path

To generate a dot path, simply map and join the keys of an issue path, if available.

```ts
import type { StandardSchemaV1 } from "@standard-schema/spec";

async function getFormErrors(schema: StandardSchemaV1, data: unknown) {
  const result = await schema["~standard"].validate(data);
  const formErrors: string[] = [];
  const fieldErrors: Record<string, string[]> = {};
  if (result.issues) {
    for (const issue of result.issues) {
      const dotPath = issue.path
        ?.map((item) => (typeof item === "object" ? item.key : item))
        .join(".");
      if (dotPath) {
        if (fieldErrors[dotPath]) {
          fieldErrors[dotPath].push(issue.message);
        } else {
          fieldErrors[dotPath] = [issue.message];
        }
      } else {
        formErrors.push(issue.message);
      }
    }
  }
  return { formErrors, fieldErrors };
}
```

##### Schema Error

To throw an error that contains all issue information, simply pass the issues of the failed schema validation to a `SchemaError` class. The `SchemaError` class extends the `Error` class with an `issues` property that contains all the issues.

```ts
import type { StandardSchemaV1 } from "@standard-schema/spec";

class SchemaError extends Error {
  public readonly issues: ReadonlyArray<StandardSchemaV1.Issue>;
  constructor(issues: ReadonlyArray<StandardSchemaV1.Issue>) {
    super(issues[0].message);
    this.name = "SchemaError";
    this.issues = issues;
  }
}

async function validateInput<TSchema extends StandardSchemaV1>(
  schema: TSchema,
  data: unknown,
): Promise<StandardSchemaV1.InferOutput<TSchema>> {
  const result = await schema["~standard"].validate(data);
  if (result.issues) {
    throw new SchemaError(result.issues);
  }
  return result.value;
}
```

## Ecosystem

These are the libraries that have already implemented the Standard Schema interface. Feel free to add your library to the list **in ascending order** by creating a pull request.

### Schema Libraries

- [ArkType](https://github.com/arktypeio/arktype): TypeScript's 1:1 validator, optimized from editor to runtime ⛵
- [Valibot](https://github.com/fabian-hiller/valibot): The modular and type safe schema library for validating structural data 🤖
- [Zod](https://github.com/colinhacks/zod) (v3.24+): TypeScript-first schema validation with static type inference

### Third Parties

- [Formwerk](https://github.com/formwerkjs/formwerk): A Vue.js Framework for building high-quality, accessible, delightful forms.
- [GQLoom](https://github.com/modevol-com/gqloom): Weave GraphQL schema and resolvers using Standard Schema.
- [Nuxt UI](https://github.com/nuxt/ui): A UI Library for Modern Web Apps, powered by Vue & Tailwind CSS.
- [oRPC](https://github.com/unnoq/orpc): Typesafe API's Made Simple 🪄
- [TanStack Form](https://github.com/TanStack/form): 🤖 Headless, performant, and type-safe form state management for TS/JS, React, Vue, Angular, Solid, and Lit.
- [TanStack Router](https://github.com/tanstack/router): A fully type-safe React router with built-in data fetching, stale-while revalidate caching and first-class search-param APIs.
- [tRPC](https://github.com/trpc/trpc): 🧙‍♀️ Move Fast and Break Nothing. End-to-end typesafe APIs made easy.
- [UploadThing](https://github.com/pingdotgg/uploadthing): File uploads for modern web devs

## Background

### The Problem

Validation is an essential building block for almost any application. Therefore, it was no surprise to see more and more JavaScript frameworks and libraries start to natively support specific schema libraries. Frameworks like Astro and libraries like the OpenAI SDK have adopted Zod in recent months to streamline the experience for their users. But to be honest, the current situation is far from perfect. Either only a single schema library gets first-party support, because the implementation and maintenance of multiple schema libraries is too complicated and time-consuming, or the choice falls on an adapter or resolver pattern, which is more cumbersome to implement for both sides.

For this reason, Colin McDonnell, the creator of Zod, came up with [the idea](https://x.com/colinhacks/status/1634284724796661761) of a standard interface for schema libraries. This interface should be minimal, easy to implement, but powerful enough to support the most important features of popular schema libraries. The goal was to make it easier for other libraries to accept user-defined schemas as part of their API, in a library-agnostic way. After much thought and consideration, Standard Schema was born.

### Use Cases

The first version of Standard Schemas aims to address the most common use cases of schema libraries today. This includes API libraries like tRPC and JavaScript frameworks like Astro and Qwik who secure the client/server communication in a type safe way using schemas. Or projects like the T3 Stack, which uses schemas to validate environment variables. It also includes UI libraries like Nuxt UI and form libraries like Reach Hook Form, which use schemas to validate user inputs. Especially with the rise of TypeScript, schemas became the de facto standard as they drastically improved the developer experience by providing the type information and validation in a single source of truth.

At the moment, Standard Schema deliberately tries to cover only the most common use cases. However, we believe that other use cases, such as integrating schema libraries into AI SDKs like Vercel AI or the OpenAI SDK to generate structured output, can also benefit from a standard interface.

## FAQ

These are the most frequently asked questions about Standard Schema. If your question is not listed, feel free to create an issue.

### Do I need to include `@standard-schema/spec` as a dependency?

No. The `@standard-schema/spec` package is completely optional. You can just copy and paste the types into your project, or manually add the `~standard` properties to your existing types. But you can include `@standard-schema/spec` as a dev dependency and consume it exclusively with `import type`. The `@standard-schema/spec` package contains no runtime code and only exports types.

### Why did you choose to prefix the `~standard` property with `~`?

The goal of prefixing the key with `~` is to both avoid conflicts with existing API surfaces and to de-prioritize these keys in auto-complete. The `~` character is one of the few ASCII characters that occurs after `A-Za-z0-9` lexicographically, so VS Code puts these suggestions at the bottom of the list.

![Screenshot showing the de-prioritization of the `~` prefix keys in VS Code.](https://github.com/standard-schema/standard-schema/assets/3084745/5dfc0219-7531-481e-9691-cff5bc471378)

### Why don't you use symbols for the keys instead of the `~` prefix?

In TypeScript, using a plain `Symbol` inline as a key always collapses to a simple `symbol` type. This would cause conflicts with other schema properties that use symbols.

```ts
const object = {
  [Symbol.for('~output')]: 'some data',
};
// { [k: symbol]: string }
```

By contrast, declaring the symbol externally makes it "nominally typed". This means that the key is sorted in autocomplete under the variable name (e.g. `testSymbol` below). Thus, these symbol keys don't get sorted to the bottom of the autocomplete list, unlike `~`-prefixed string keys.

![Screenshot showing the prioritization of external symbols in VS Code](https://github.com/standard-schema/standard-schema/assets/3084745/82c47820-90c3-4163-a838-858b987a6bea)

### What should I do if I only accept synchronous validation?

The `~validate` function does not necessarily have to return a `Promise`. If you only accept synchronous validation, you can simply throw an error if the returned value is an instance of the `Promise` class.

```ts
import type { StandardSchemaV1 } from "@standard-schema/spec";

function validateInput(schema: StandardSchemaV1, data: unknown) {
  const result = schema["~standard"].validate(data);
  if (result instanceof Promise) {
    throw new TypeError('Schema validation must be synchronous');
  }
  // ...
}
```
