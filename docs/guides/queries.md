---
id: queries
title: Queries
---

## Query Basics

A query is a declarative dependency on an asynchronous source of data that is tied to a **unique key**. A query can be used with any Promise based method (including GET and POST methods) to fetch data from a server. If your method modifies data on the server, we recommend using [Mutations](https://react-query.tanstack.com/docs/guides/mutations) instead.

To subscribe to a query in your components or custom hooks, call the `useQuery` hook with at least:

- A **unique key for the query**
- A function that returns a promise that:
  - Resolves the data, or
  - Throws an error

```js
import { useQuery } from '@tanstack/react-query'

function App() {
  const info = useQuery(['todos'], fetchTodoList)
}
```

The **unique key** you provide is used internally for refetching, caching, and sharing your queries throughout your application.

The query results returned by `useQuery` contains all of the information about the query that you'll need for templating and any other usage of the data:

```js
const result = useQuery(['todos'], fetchTodoList)
```

The `result` object contains a few very important states you'll need to be aware of to be productive. A query can only be in one of the following states at any given moment:

- `isLoading` or `status === 'loading'` - The query has no data yet
- `isError` or `status === 'error'` - The query encountered an error
- `isSuccess` or `status === 'success'` - The query was successful and data is available

Beyond those primary states, more information is available depending on the state of the query:

- `error` - If the query is in an `isError` state, the error is available via the `error` property.
- `data` - If the query is in a `success` state, the data is available via the `data` property.

For **most** queries, it's usually sufficient to check for the `isLoading` state, then the `isError` state, then finally, assume that the data is available and render the successful state:

```js
function Todos() {
  const { isLoading, isError, data, error } = useQuery(['todos'], fetchTodoList)

  if (isLoading) {
    return <span>Loading...</span>
  }

  if (isError) {
    return <span>Error: {error.message}</span>
  }

  // We can assume by this point that `isSuccess === true`
  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

If booleans aren't your thing, you can always use the `status` state as well:

```js
function Todos() {
  const { status, data, error } = useQuery(['todos'], fetchTodoList)

  if (status === 'loading') {
    return <span>Loading...</span>
  }

  if (status === 'error') {
    return <span>Error: {error.message}</span>
  }

  // also status === 'success', but "else" logic works, too
  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

TypeScript will also narrow the type of `data` correctly if you've checked for `loading` and `error` before accessing it.

### FetchStatus

In addition to the `status` field, the `result` object, you will also get an additional `fetchStatus`property with the following options:

- `fetchStatus === 'fetching'` - The query is currently fetching.
- `fetchStatus === 'paused'` - The query wanted to fetch, but it is paused. Read more about this in the [Network Mode](./network-mode) guide.
- `fetchStatus === 'idle'` - The query is not doing anything at the moment.

### Why two different states?

Background refetches and stale-while-revalidate logic make all combinations for `status` and `fetchStatus` possible. For example:
- a query in `success` status will usually be in `idle` fetchStatus, but it could also be in `fetching` if a background refetch is happening.
- a query that mounts and has no data will usually be in `loading` status and `fetching` fetchStatus, but it could also be `paused` if there is no network connection.

So keep in mind that a query can be in `loading` state without actually fetching data. As a rule of thumb:

- The `status` gives information about the `data`: Do we have any or not?
- The `fetchStatus` gives information about the `queryFn`: Is it running or not?

## Further Reading

For an alternative way of performing status checks, have a look at the [Community Resources](../community/tkdodos-blog#4-status-checks-in-react-query).
