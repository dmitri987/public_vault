
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

##### Cache fetched data for 1 minute
```tsx
...   useFetch(url, { staleTime: 60_000 }) 
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
It has the same arguments as `useFetch`, but return `Promise` instead of `{ data, isLoading, error }`.

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


### Source Code
```tsx
import { useEffect, useRef, useState } from 'react';

type TimestampInMilliseconds = number;
type StaleTimeInMilliseconds = number;

export type CacheMap<Data = any> = Pick<
  Map<
    string,
    {
      data: Data | Promise<Data>;
      lastUpdate: TimestampInMilliseconds;
      staleTime: StaleTimeInMilliseconds;
    }
  >,
  'set' | 'get' | 'delete'
>;

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

const defaultCacheMap: CacheMap = new Map();

const defaultFetcher: Fetcher = async (url, options) => {
  return fetch(url, options).then(res => {
    if (!res.ok)
      throw new Error(
        `Fetching error. Response status: ${res.status}, message: '${res.statusText}'`
      );

    return res.json();
  });
};

function stringifyKey<Key = any>(key: Key, strKey?: string) {
  if (strKey) return strKey;

  try {
    return typeof key === 'string' ? key : JSON.stringify(key);
  } catch (err) {
    console.error("'Key' argument must be stringifiable with JSON.stringify");
  }
}

/*
 * Get data with 'then' and error with 'catch'
 * 'data' == null, when 'key' == null | undefined
 *
 * call onSuccess and onError
 */
async function _fetch<Key = any, Data = any, Error = any>(
  key: Key,
  options?: FetchOptions<Key, Data, Error>
): Promise<Data | Error | null> {
  const { fetcher, cacheMap, onError, onSuccess, staleTime, refetch, method, body } = options ?? {};

  if (key === null || key === undefined) return null;

  const strKey = stringifyKey(key);
  if (!refetch) {
    const record = cacheMap.get(strKey);
    if (record !== undefined) {
      if (
        refetch === false ||
        record.data instanceof Promise ||
        Date.now() - record.lastUpdate < record.staleTime
      ) {
        // console.log('_fetch return cached Promise', record.data)
        return record.data;
      } else if (staleTime === 0) {
        cacheMap.delete(strKey);
      }
    }
  }

  const promise = fetcher(key, { body, method })
    .then(result => {
      // console.log('_fetch .then', result)
      // when promise is resolved change cache with its result
      if (staleTime) cacheMap.get(strKey).data = result;
      onSuccess?.(result);
      return result;
    })
    .catch(error => {
      onError?.(error);
      throw error;
    });

  if (staleTime) {
    cacheMap.set(strKey, {
      data: promise, // keep Promise object in cachd until it's resolved
      lastUpdate: Date.now(),
      staleTime,
    });
  }

  return promise;
}

/*
 * Return { data, isLoading, error } object.
 * On first fetch 'data' is undefined and 'isLoading' == true
 * 'error' is always undefined on success.
 *
 * Arguments:
 *   key        string or any object serializable with JSON.serialize()
 *              It will be passed to 'fetcher' as is, and be optionally
 *              used as stringified key for received data in cache map
 *
 *              When null or undefined, don't fetch
 *
 *   options:   When omitted use default fetcher, no caching
 *
 *     fetcher  Function, which takes 'key' and returns Promise resolved to data object
 *              When omitted default fetcher is used
 *
 *              Note: use axios or manually throw error on 400 and 500 statuses
 *              in the 'fetcher' if you use native 'fetch', otherwise
 *              useFetch won't call options.onError or return 'error' property
 *              for statuses > 200
 *
 *     cacheMap   Optionally provide custom map for cache
 *
 *     staleTime  How long to keep fetched data in cache, in milliseconds
 *                If 0, fetched data won't be cached
 *                Default 0
 *
 *     refetch    true:    force refetch, write to cache if staleTime > 0
 *                false:   if cached data exists, return it even if it's stale
 *                         if cached data doesn't exist, fetch it,
 *                         write to cache if staleTime > 0
 *                omitted  if cached data is stale, refetch and update the cache,
 *                         if fresh, return from the cache, don't refetch
 *
 *     onError    Call it on error: onError(error)
 *     onSuccess  Call it on successful fetch: onSuccess(data)
 *
 * Note: fetcher, cacheMap, onSuccess, onError are set on first render
 *       and ignore later changes.
 */
export default function useFetch<Key = any, Data = any, Error = any>(
  key: Key,
  options?: FetchOptions<Key, Data, Error>
): FetchResult<Data, Error> {
  const [data, setData] = useState<Data>();
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState();

  const staticOptionsRef = useRef({
    fetcher: options?.fetcher ?? (defaultFetcher as Fetcher<Key, Data>),
    cacheMap: options?.cacheMap ?? (defaultCacheMap as CacheMap),
    onError: options?.onError,
    onSuccess: options?.onSuccess,
  });

  const staleTime = options?.staleTime ?? 0;
  const { refetch, method, body } = options ?? {};

  useEffect(() => {
    const _options = {
      ...staticOptionsRef.current,
      staleTime,
      refetch,
      method,
      body,
    };

    setIsLoading(true);
    _fetch(key, _options)
      .then(data => {
        // console.log('useFetch .then', data)
        setIsLoading(false);
        setError(undefined);
        if (data !== null) setData(data);
      })
      .catch(error => {
        setError(error);
        setData(undefined);
        setIsLoading(false);
      });
  }, [key, staleTime, refetch, method, body]);

  return { data, isLoading, error };
}


export async function prefetch<Key = any, Data = any, Error = any>(
  key: Key,
  options?: FetchOptions<Key, Data, Error>
): Promise<Data | Error | null> {
  const _options = {
    fetcher: options?.fetcher ?? defaultFetcher,
    cacheMap: options?.cacheMap ?? defaultCacheMap,
    onError: options?.onError,
    onSuccess: options?.onSuccess,
    staleTime: options?.staleTime ?? 0,
    refetch: options?.refetch,
    method: options?.method ?? 'GET',
    body: options?.body,
  };

  return _fetch(key, _options);
}


```