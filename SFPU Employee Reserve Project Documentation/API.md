
[link to Google Sheet with API](https://docs.google.com/spreadsheets/d/1ihWrgdj_U8cEzfwruCdzkGEAfEKQ7FbzWwb7i-LOoP0/edit#gid=0)

#### `/start`
##### Response: 
```ts
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
##### Request (in `body`)
```json
{
	"ownershipKeys": [ "DP", "AT", "TOV" ],
	"industryKeys": [ "ENERGY", "SEA" ],
	"regionKeys": [],
	"page":"1"
}
```
##### Response
```ts
{
  vacancyCards: [
    {
		  enterpriseName: string;
		  workPlace: string;
		  edrpou: string;
		  site: URL;
		  hasPhoto: boolean;
		  vacancyId: number;
    },
       ...
  ],
  totalPages: number;
  totalVacancies: number;
}

```
