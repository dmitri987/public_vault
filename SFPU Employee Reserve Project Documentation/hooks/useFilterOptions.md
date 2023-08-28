This is how I propose to manage enterprise filters.
Keep all options in single object, not in three `useState`.
Make them available to components with `Context Provider`.
Get and set them in the same way as with `useState`.

### Types
```ts
// simplified
export interface FilterOptions {
  ownershipKeys?: string[]; // 'DP' | 'AT' | 'TOV'
  industryKeys?: string[];  // 'ENERGY' | 'CHEMICAL' | ...
  regionKeys?: string[];    // 'VOLYN_OBLAST' | ...
}

function useFilterOptions(): [options, setOptions]
```

### Usage
#### Put `FilterOptionsProvider` over all components, where you plan to use `useFilterOptions`
```tsx
// src/components/App/App.tsx

import { FilterOptionsProvider } from '../FilterOptions';

		    ...
		const App = () => {
		  return (
		    <>
		      <Header />
		      <FilterOptionsProvider>
		        <Main />
		      </FilterOptionsProvider>
		      <Footer />
		    </>
		  );
		};
```

#### Use `useFilterOptions` inside your component
```tsx
// src/components/VacancyList.tsx

import { useFilterOptions } from '../FilterOptions';

const VacancyList = () => {
  const [options, setOptions] = useFilterOptions();

  useEffect(() => {
    // do something if options is changed
  }, [options])

  // refetch only if options is changed
  const { data } = useFetch(options, { fetcher: someFilterOptionBasedFetcher })

	return (
	  <div>
	      ...
      // set filter options
	    <button onClick={() => setOptions({ ownershipKeys: ['DP'] })})> ...
}
```

##### Use `setOptions`
###### Reset `options`
```tsx
  setOptions()  or  setOptions(null)
```
###### Set one key list, **reset the rest**
```ts
setOptions({ ownershipKeys: ['DP', 'AT'] })
```
###### Set one key list, the rest is unchanged
```ts
setOptions(prev => ({ ...prev, ownershipKeys: ['DP'] }))
```


### Source Code
```tsx
import {
  createContext,
  useCallback,
  useContext,
  useEffect,
  useState,
} from 'react';
import { FilterOptions } from 'src/types';
import { isEqual as isEqualDeep } from 'lodash';

type FilterOptionsSetter = (
  v?: FilterOptions | ((prev: FilterOptions) => FilterOptions)
) => void;

const emptyOptions = {
  ownershipKeys: [],
  industryKeys: [],
  regionKeys: [],
  // searchString: '', // TODO add later
};

let _options: FilterOptions = emptyOptions;

const FilterOptionsContext = createContext<
  [FilterOptions, FilterOptionsSetter] | null
>(null);

/* Use it to get current filter options in non-reactive environment.
 *
 * Example:
 *   ...
 *   <Route loader={() => prefetch(someUrl, getFilterOptions())}
 *
 */
export const getFilterOptions = () => ({ ..._options });

/* This is where we keep the state of the filter options.
 * Put it over components, where you want to have access
 * to useFilterOptions.
 *
 * Example:
 *   ...
 *   <FilterOptionsProvider>
 *     <Home />
 *      ...
 *   </FilterOptionsProvider>
 *
 */
export const FilterOptionsProvider = ({ children }) => {
  const [options, setOptions] = useState<FilterOptions>(emptyOptions);
  console.log('filterOptions:', options);

  useEffect(() => {
    _options = options;
  }, [options]);

  const addOptions: FilterOptionsSetter = useCallback(
    v => {
      if (v === null || v === undefined) setOptions({ ...emptyOptions });
      // else if (typeof v === 'string') { // TODO: add when searchString is ready
      //   setOptions(prev => v !== prev.searchString ? ({ ...emptyOptions, ...prev, searchString: v } : prev));
      else {
        setOptions(prev => {
          const newOptions = {
            ...emptyOptions,
            ...(v instanceof Function ? v(prev) : v),
          };

          const newSortedOptions = Object.entries(newOptions).reduce(
            (sorted, [key, value]) => {
              if (value instanceof Array) sorted[key] = value.sort();
              else sorted[key] = value;

              return sorted;
            },
            {}
          );

          return isEqualDeep(prev, newSortedOptions) ? prev : newSortedOptions;
        });
      }
    },
    [setOptions]
  );

  return (
    <FilterOptionsContext.Provider value={[options, addOptions]}>
      {children}
    </FilterOptionsContext.Provider>
  );
};

/* Usage:
 *
 *   const [options, setOptions] = useFilterOptions();
 *
 *   useEffect(() => {
 *     // do something if options is changed
 *   }, [options])
 *      ...
 *
 *     <button onClick={() => setOptions()}>Reset Filters</button>
 *
 *
 * // reset options
 * setOptions() or setOptions(null)
 *
 * // reset searchString, keep the rest
 * setOptions('search string')
 *
 * // set 'ownershipKeys' to new value, reset the rest
 * setOptions({ ownershipKeys: ['DP'] })
 *
 * // add 'ownershipKeys', keep the rest
 * setOptions(prev => ({ ...prev, ownershipKeys: ['DP'] }))
 *
 * Note: setOptions checks deep equality of new and existing options and keep
 * the existing one if they are equal, so 'options' will be the same object
 * and won't cause rerender when it's used as a dependency in hooks.
 */
export function useFilterOptions(): [
  options: FilterOptions,
  setOptions: FilterOptionsSetter
] {
  return useContext(FilterOptionsContext);
}

```