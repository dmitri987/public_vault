
Use `useFetch` to fetch and, optionally, cache data. 
Use `prefetch` in the same way, but outside of React components.

#### Types
```ts
// simplified
export type Fetcher = (
  key: Key,
  options?: { method?: string; body?: any } // TODO: remove it ??
) => Promise;

export type FetchOptions = {
  fetcher?: Fetcher;
  cacheMap?: Map;
  staleTime?: number; // in milliseconds
  refetch?: boolean;
  onError?(error): void;
  onSuccess?(data): void;
  method?: any;
  body?: any;
};

export type FetchResult<Data = any, Error = any> = {
  data?: Data;
  isLoading: boolean;
  error?: Error;
};

function useFetch(key, options?: FetchOptions): FetchResult

async function prefetch(key, options?: FetchOptions): Promise 

```

### `useFetch`
#### Usage
##### Simple fetch, no caching
```tsx
const MyComponent = () => {
	const { data, isLoading, error } = useFetch('https://...');

  return (
    <div>
      {isLoading && 'Loading...'}
      {data && data.map(item => ...)}
      {error && error.message}
    </div>
    )
}
```

##### Using custom `fetcher`
```tsx
const fetcher = async (page) => {
  return fetch(url + '?page=' + page).then(res => res.json());
}
        ...
// useFetch will pass 'key' to the 'fetcher' as is				
// and save its resolved result to 'data'
const { data } = useFetch(page, { fetcher })        
```

##### Using custom `fetcher` to fake data
```tsx
import companies from 'src/fakedata/companies.json';

const fetcher = async () => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(companies), 200)
  })
}
     ...
```

##### Cache fetched data
```tsx
...   useFetch(url, { staleTime: 60_000 }) // cache results for 1 min
```

##### Force fetch even if cached data is still fresh, and cache it for 10 seconds
```tsx
       useFetch(url, { refetch: true, staleTime: 10_000 })
```

##### Return cached data even if it's stale, otherwise fetch it and cache for 10 sec
```tsx
       useFetch(url, { refetch: false, staleTime: 10_000 })
```

When no `refetch`, stale data will be refetched automatically and saved to cache if `staleTime` > 0

### `prefetch`
It has the same arguments as `useFetch`, but return `Promise` instead of `{ data, isLoading, error }`
`prefetch` and `useFetch` save and load data from the same `cacheMap` by default.

##### Using to prefetch data in `React Router` loader
```tsx
	<Route
		path="/"
		element={<MainLayout />}
		loader={() => prefetch(url, { staleTime: Infinity })}
	>
				...
```