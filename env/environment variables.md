### Type Safety on env (environmental variables)

create a type.ts file 
```js
declare namespace NodeJS{

    export interface ProcessEnv{

        DATABASE_URL: string

    }

}
```