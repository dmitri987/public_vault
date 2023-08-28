

### Using `useFetch` and `prefetch` to fetch and, optionally, cache data from the server

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
const { data } = useFetch(page, { fetcher })        

```



# Dir/file structure   
#structure #file #dir 

```
/ 
  src/

    api.ts   // everything related to communication with the server
             // mainly hooks ??

```

# Route-Component Model
# Component Model
// component signature

# Types

# API
Entry Point: `/start`
Request:
Response
Component
Mapping:

// Questionnaire

