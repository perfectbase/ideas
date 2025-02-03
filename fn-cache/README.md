# Idea

A light-weight caching library that would provide a Next.js `unstable_cache`-like experience for any JavaScript framework. This library would make it easy to cache expensive function calls with minimal configuration, whether they're API requests, database queries, or heavy computations.

# Motivation

I've been interested in using TanStack Start for some future projects, but I love how easy it is to cache things on the server-side with Next.js, so it would be great to have an easy way to have the same experience with it.

## Key Features

- Easily cache synchronous or asynchronous functions.
- TypeScript support.
- Configure expiration for cached entries.
- Use the default in-memory adapter or plug in external databases such as Redis, DynamoDB, MongoDB, MySQL, PostgreSQL, etc.

## Installation

```bash
npm install fn-cache
```

## Setup

```ts
// /src/lib/cache.ts
import { buildCacheClient } from "fn-cache";
import { RedisAdapter } from "@fn-cache/redis";
import superjson from "superjson";

export const fnCache = buildCacheClient({
  adapter: new RedisAdapter({
    host: "localhost",
    port: 6379,
  }),
  transformer: superjson,
});
```

## Usage

### Basic Example

```ts
import { fnCache } from "@/lib/cache";

// Example of an expensive function (could be an API call, a heavy computation, etc.)
async function fetchData(param) {
  await new Promise((resolve) => setTimeout(resolve, 5000));
  return `Result for ${param}`;
}

// Create a cached version of the function
const cachedFetchData = fnCache(fetchData, {
  additionalKeys: ["additional-key"],
  tags: ["data"],
  revalidate: "1hour",
});

// Use the cached function
const result = await cachedFetchData("example");
console.log(result); // Will print "Result for example"
```

### On-Demand Cache Invalidation

```ts
import { invalidateCache } from "fn-cache";

await invalidateCache(["data"]);
```

## Parameters

### fnCache

```ts
const data = (await fnCache(fetchData, options))();
```

- `fetchData`: The function to cache.
- `options` (optional): Controls the cache behavior.
  - `additionalKeys` (optional): Additional keys to use for caching. The stringified version of the function and its arguments are automatically used.
  - `tags` (optional): Tags to use for caching. Can be used to invalidate the cache later.
  - `revalidate` (optional): The time it takes for the cache to expire. Can be set to a number of seconds, or a string like `1hour` or `1day`. Defaults to `Infinity` or to the value set in `buildCacheClient`.

### buildCacheClient

```ts
const fnCache = buildCacheClient(options);
```

- `options` (optional): The cache client configuration.
  - `adapter` (optional): The adapter to use for caching. Defaults to the in-memory adapter.
  - `transformer` (optional): The transformer to use for serializing and deserializing the cache data. Defaults to `JSON.stringify` and `JSON.parse`. You can use `superjson` to make it possible to cache data that is not serializable by `JSON`. e.g. `Date`, `Map`, `Set`, etc.
  - `validTags` (optional): If provided, the `tags` option in `fnCache` will be type-safe and you will get autocomplete for the tags.
  - `defaultRevalidate` (optional): The default revalidate time to use for `fnCache` if not provided. Can be set to a number of seconds, or a string like `1hour` or `1day`. Defaults to `Infinity`.
