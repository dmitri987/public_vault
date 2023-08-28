*Here is some documentation on SFPU Employee Reserve Project*

### Hooks
- [useFetch](hooks/useFetch)
- [useFilterOptions](hooks/useFilterOptions)

### API
[link to Google Sheet with API](https://docs.google.com/spreadsheets/d/1ihWrgdj_U8cEzfwruCdzkGEAfEKQ7FbzWwb7i-LOoP0/edit#gid=0)

#### `/start`
##### Response: 
```js
{
  ownershipTypes: [
	  {
		  key: string; // 'DP' | 'AT' | 'TOV'
		  abbreviation: string; // 'ДП' | 'АТ' | 'ТОВ'
		  name: string; // 'Державне підприємство' | ...
		  numberEnterprise?: number;
		  numberVacancy?: number;
	  },
	     ...
	],
  industryTypes: [
    {
		  key: string; // 'ENERGY' | 'CHEMICAL' | ...
		  name: string; // 'Енергетика' | ...
		  alterText?: string;
		  numberOfEnterprises?: number;
    },
       ...
  ],
  regionTypes: [
    {
      key: string; // 'VOLYN_OBLAST' | ...
		  name: string; // 'Волинська область' | ...
    },
        ...
  ]
}
```

#### `/vacancy/list`
##### Request (in )



### Dir/file structure   
#structure #file #dir 

```
/ 
  src/
	  components/
	    

    api.ts   // everything related to communication with the server
             // mainly hooks ??

```

