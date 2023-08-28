
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
```

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

const fetcher = async (page) => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(companies))
  })
}
```

