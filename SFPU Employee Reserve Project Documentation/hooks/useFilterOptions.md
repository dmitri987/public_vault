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

const VacancyList = () => {
  const [options, setOptions] = useFilterOptions();

  useEffect(() => {
    // do something if options are changed
  }, [options])

  // refetch only if options are changed
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
###### Set 
