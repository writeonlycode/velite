# Integration with Next.js

Velite is a framework agnostic library, it can be used in any JavaScript framework or library, including Next.js.

Here is some recipes for help you better integrate Velite with Next.js.

## Start Velite with Next.js Plugin

You can use the Next.js plugin to call Velite's programmatic API to start Velite with better integration.

in your `next.config.js`:

::: code-group

```js [CommonJS]
/** @type {import('next').NextConfig} */
module.exports = {
  // othor next config here...
  webpack: config => {
    config.plugins.push(new VeliteWebpackPlugin())
    return config
  }
}

class VeliteWebpackPlugin {
  static started = false
  apply(/** @type {import('webpack').Compiler} */ compiler) {
    // executed three times in nextjs
    // twice for the server (nodejs / edge runtime) and once for the client
    compiler.hooks.beforeCompile.tapPromise('VeliteWebpackPlugin', async () => {
      if (VeliteWebpackPlugin.started) return
      VeliteWebpackPlugin.started = true
      const dev = compiler.options.mode === 'development'
      const { build } = await import('velite')
      await build({ watch: dev, clean: !dev })
    })
  }
}
```

```js [ESM]
import { build } from 'velite'

/** @type {import('next').NextConfig} */
export default {
  // othor next config here...
  webpack: config => {
    config.plugins.push(new VeliteWebpackPlugin())
    return config
  }
}

class VeliteWebpackPlugin {
  static started = false
  apply(/** @type {import('webpack').Compiler} */ compiler) {
    // executed three times in nextjs
    // twice for the server (nodejs / edge runtime) and once for the client
    compiler.hooks.beforeCompile.tapPromise('VeliteWebpackPlugin', async () => {
      if (VeliteWebpackPlugin.started) return
      VeliteWebpackPlugin.started = true
      const dev = compiler.options.mode === 'development'
      await build({ watch: dev, clean: !dev })
    })
  }
}
```

:::

::: info

ESM `import { build } from 'velite'` may be got a `[webpack.cache.PackFileCacheStrategy/webpack.FileSystemInfo]` warning generated during the `next build` process, which has little impact,
refer to https://github.com/webpack/webpack/pull/15688

:::

::: tip

The Next.js plugin is still under development...

:::


### Start Velite in npm script with `npm-run-all`:

::: info

`VeliteWebpackPlugin` is recommended, but if your project is deployed on Vercel, there may be an error of `free(): invalid size` or `munmap_chunk(): invalid pointer
`, which is usually related to the sharp module. Please refer to: https://github.com/zce/velite/issues/52#issuecomment-2016789204

:::

**package.json**:

```json
{
  "scripts": {
    "dev:content": "velite --watch",
    "build:content": "velite --clean",
    "dev:next": "next dev",
    "build:next": "next build",
    "dev": "run-p dev:*",
    "build": "run-s build:*",
    "start": "next start"
  }
}
```

## Typed Routes

When you use the `typedRoutes` experimental feature, you can get the typed routes in your Next.js app.

In this case, you can specify a more specific type for the relevant schema to make it easier to use on `next/link` or `next/router`.

e.g.

```ts
import type { Route } from 'next'
import type { Schema } from 'velite'

const options = defineCollection({
  // ...
  schema: s.object({
    // ...
    link: z.string() as Schema<Route<'/posts/${string}'>>
  })
})
```

Then you can use it like this:

```tsx
import Link from 'next/link'

import { options } from '@/.velite'

const Post = async () => {
  return (
    <div>
      {/* typed route */}
      <Link href={options.link}>Read more</Link>
    </div>
  )
}
```

## Example

- [examples/nextjs](https://github.com/zce/velite/tree/main/examples/nextjs)
