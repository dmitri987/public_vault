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

```