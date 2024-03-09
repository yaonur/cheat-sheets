#### Deno and TypeScript (ts) same time
```json
{
    "folders": [
        {   "name":"root",
            "path": ".",
            "settings": {
            }
        },
        {
            "name":"DenoServer",
            "path": "./server",
            // "settings": {
            //  "deno.enable":true
            // }
        },
    ],
    "settings": {
        "deno.enable": true,
        "deno.lint": true,
        "deno.unstable": false,
        "deno.enablePaths": [
            "./server"
        ],
    }
}
```
folder structre is 
/root
indesx.ts
	/server
		server.ts

#### ts config for deno 
```json
{
    "compilerOptions": {
      "module": "esnext",
      "target": "esnext",
      "esModuleInterop": true,
      "moduleResolution": "node"
    },
    "include": ["./**/*"],
    "exclude": ["./server/**/*"]
  }
```