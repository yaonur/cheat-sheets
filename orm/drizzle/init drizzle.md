#### install drizzle packages
```bash
	pnpm add drizzle-orm pg 
	pnpm add -D drizzle-kit @types/pg
```
#### set the drizzle.config.ts
```ts
	import { defineConfig } from 'drizzle-kit'
	export default defineConfig({
	 schema: "./schema.ts",
	  driver: 'pg',
	  dbCredentials: {
	    connectionString: process.env.DATABASE_URL,
	  },
	  verbose: true,
	  strict: true,
	})
```

#### drizzle 
#### drizzle migration file
```ts
	import { Pool } from "pg";
	import { drizzle } from "drizzle-orm/node-postgres";
	import {migrate} from "drizzle-orm/node-postgres/migrator"
	import "dotenv/config";
	  
	
	const url = process.env.DATABASE_URL
	const pool = new Pool({
	    connectionString: url
	})
	const db = drizzle(pool)
	async function main() {
	    console.log("migration started...")
	    await migrate(db,{migrationsFolder: "./drizzle"})
	    console.log("migration ended...")
	    process.exit(0)
	}
	main().catch((e) => {
	    console.error(e)
	    process.exit(1)
	})
```
