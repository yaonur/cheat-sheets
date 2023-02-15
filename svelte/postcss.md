``` shell
pnpm i -D postcss-preset-env postcss-load-config @types/postcss-preset-env
```

create a postcss.config.cjs file

```js
const postcssPresetEnv = require('postcss-preset-env')

const config = {
  plugins: [
    postcssPresetEnv({
      stage: 3,
      features: {
        'nesting-rules': true,
        'custom-media-queries': true,
        'media-query-ranges': true
      }
    })
  ]
}

module.exports = config

```

Here we specified the CSS features using the `stage` option and enabled the nesting rule and you can [learn more from the documentation](https://github.com/csstools/postcss-plugins/tree/main/plugin-packs/postcss-preset-env).

The last step is to enable loading the config in `svelte.config.js`.

``` js

import adapter from '@sveltejs/adapter-auto'
import preprocess from 'svelte-preprocess'

/** @type {import('@sveltejs/kit').Config} */
const config = {
	preprocess: preprocess({ postcss: true }),
	kit: {
		adapter: adapter()
	}
}

export default config

```
Hope you take advantage of future CSS today and you can find the example on [GitHub](https://github.com/JoysOfCode/svelte-future-css) or play with it on [StackBlitz](https://stackblitz.com/github/joysofcode/svelte-future-css).