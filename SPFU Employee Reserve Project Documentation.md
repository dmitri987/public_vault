

### Using `useFetch` and `prefetch` to fetch and, optionally, cache data from the server

#### Types
```ts
export type Fetcher<Key = any, Data = any> = (
  key: Key,
  options?: { method?: string; body?: any } // TODO: remove it ??
) => Promise<Data>;

export type FetchOptions<Key = any, Data = any, Error = any> = {
  fetcher?: Fetcher<Key, Data>;
  cacheMap?: CacheMap<Data>;
  staleTime?: StaleTimeInMilliseconds;
  refetch?: boolean;
  onError?(error: Error): void;
  onSuccess?(data: Data): void;
  method?: any;
  body?: any;
};

export type FetchResult<Data = any, Error = any> = {
  data?: Data;
  isLoading: boolean;
  error?: Error;
};
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

