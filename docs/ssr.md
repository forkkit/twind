# Server Side Rendering (SSR)

Twind supports Server Side Rendering (SSR) out of the box.

> Please note the `twind/server` bundle is Node.JS only.

## Synchronous SSR

The following example assumes your app is using the `tw` named export from `twind`
but the same logic can be applied to custom instances.

```js
import { setup } from 'twind'
import { virtualSheet, getStyleTag } from 'twind/server'

const sheet = virtualSheet()

setup({ ...sharedOptions, sheet })

function ssr() {
  // 1. Reset the sheet for a new rendering
  sheet.reset()

  // 2. Render the app
  const body = renderTheApp()

  // 3. Create the style tag with all generated CSS rules
  const styleTag = getStyleTag(sheet)

  // 4. Generate the reponse html
  return `<!DOCTYPE html>
    <html lang="en">
    <head>${styleTag}</head>
    <body>${body}</body>
    </html>
  `
}
```

In order to prevent harmful code injection on the web, a [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) may be put in place. During server-side rendering, a cryptographic nonce (number used once) may be embedded when generating a page on demand:

```js
// ... other code is the same as before ...

// Usage with webpack: https://webpack.js.org/guides/csp/
const styleTag = getStyleTag(sheet, { nonce: __webpack_nonce__ })
```

## Asynchronous SSR

> **Note**: This is an experimental feature and only supported for Node.JS >=12. Use with care and please [report any issue](https://github.com/tw-in-js/twind/issues/new) you find.
> Consider using the synchronous API when ever possible due to the relatively expensive nature of the [promise introspection API](https://docs.google.com/document/d/1rda3yKGHimKIhg5YeoAmCOtyURgsbTH_qaYR79FELlk/edit) provided by V8.
> Async server side rendering is implemented using [async_hooks](https://nodejs.org/docs/latest-v14.x/api/async_hooks.html). Callback-based APIs and event emitters may not work or need special handling.

```js
import { setup } from 'twind'
import { asyncVirtualSheet, getStyleTag } from 'twind/server'

const sheet = asyncVirtualSheet()

setup({ ...sharedOptions, sheet })

async function ssr() {
  // 1. Reset the sheet for a new rendering
  sheet.reset()

  // 2. Render the app
  const body = await renderTheApp()

  // 3. Create the style tag with all generated CSS rules
  const styleTag = getStyleTag(sheet)

  // 4. Generate the reponse html
  return `<!DOCTYPE html>
    <html lang="en">
    <head>${styleTag}</head>
    <body>${body}</body>
    </html>
  `
}
```

## Streaming SSR

> Supporting ReactDOM.renderToNodeStream and Vue.renderToStream is still on the roadmap...

## Frameworks

### React

```js
import { renderToString } from 'react-dom/server'

import { setup } from 'twind'
import { virtualSheet, getStyleTag } from 'twind/server'

import App from './app'

const sheet = virtualSheet()

setup({ ...sharedOptions, sheet })

function ssr() {
  // 1. Reset the sheet for a new rendering
  sheet.reset()

  // 2. Render the app
  const body = renderToString(<App />)

  // 3. Create the style tag with all generated CSS rules
  const styleTag = getStyleTag(sheet)

  // 4. Generate the reponse html
  return `<!DOCTYPE html>
    <html lang="en">
    <head>${styleTag}</head>
    <body>${body}</body>
    </html>
  `
}
```

### Preact

```js
import renderToString from 'preact-render-to-string'

import { setup } from 'twind'
import { virtualSheet, getStyleTag } from 'twind/server'

import App from './app'

const sheet = virtualSheet()

setup({ ...sharedOptions, sheet })

function ssr() {
  // 1. Reset the sheet for a new rendering
  sheet.reset()

  // 2. Render the app
  const body = renderToString(<App />)

  // 3. Create the style tag with all generated CSS rules
  const styleTag = getStyleTag(sheet)

  // 4. Generate the reponse html
  return `<!DOCTYPE html>
    <html lang="en">
    <head>${styleTag}</head>
    <body>${body}</body>
    </html>
  `
}
```

### Svelte

```js
import { setup } from 'twind'
import { virtualSheet, getStyleTag } from 'twind/server'

import App from './app.svelte'

const sheet = virtualSheet()

setup({ ...sharedOptions, sheet })

function ssr() {
  // 1. Reset the sheet for a new rendering
  sheet.reset()

  // 2. Render the app
  const { head = '', html, css } = App.render({})

  if (css && css.code) {
    head += `<style>${css.code}</style>`
  }

  // 3. Create the style tag with all generated CSS rules
  head += getStyleTag(sheet)

  // 4. Generate the reponse html
  return `<!DOCTYPE html>
    <html lang="en">
    <head>${head}</head>
    <body>${html}</body>
    </html>
  `
}
```

### Vue

```js
// createBundleRenderer works the same
import { createRenderer } from 'vue-server-renderer'

import { setup } from 'twind'
import { asyncVirtualSheet, getStyleTag } from 'twind/server'

import { createApp } from './app'

const sheet = asyncVirtualSheet()

setup({ ...sharedOptions, sheet })

const renderer = createRenderer({
  /* options */
})

async function ssr() {
  // 1. Reset the sheet for a new rendering
  sheet.reset()

  // 2. Render the app
  const body = await renderer.renderToString(createApp())

  // 3. Create the style tag with all generated CSS rules
  const styleTag = getStyleTag(sheet)

  // 4. Generate the reponse html
  return `<!DOCTYPE html>
    <html lang="en">
    <head>${styleTag}</head>
    <body>${body}</body>
    </html>
  `
}
```

## Next.js

```js
/* pages/_document.js */

import Document from 'next/document'

import { setup } from 'twind'
import { asyncVirtualSheet, getStyleTagProperties } from 'twind/server'

const sheet = asyncVirtualSheet()

setup({ ...sharedOptions, sheet })

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    sheet.reset()

    const initialProps = await Document.getInitialProps(ctx)

    const { id, textContent } = getStyleTagProperties(sheet)

    const styleProps = {
      id,
      dangerouslySetInnerHTML: {
        __html: textContent,
      },
    }

    return {
      ...initialProps,
      styles: (
        <>
          {initialProps.styles}
          <style {...styleProps} />
        </>
      ),
    }
  }
}
```

## Gatsby

> **Note** This has not been tested yet.

```js
/* gatsby-ssr.js */

const { setup } = require('twind')
const { asyncVirtualSheet, getStyleTagProperties } = require('twind/server')

const sheet = asyncVirtualSheet()

setup({ ...sharedOptions, sheet })

exports.wrapPageElement = ({ element }) => {
  sheet.reset()

  return element
}

exports.onRenderBody = ({ setHeadComponents, pathname }) => {
  const { id, textContent } = getStyleTagProperties(sheet)

  const styleProps = {
    id,
    dangerouslySetInnerHTML: {
      __html: textContent,
    },
  }

  setHeadComponents([
    React.createElement('style', {
      id,
      dangerouslySetInnerHTML: {
        __html: textContent,
      },
    }),
  ])
}
```

<hr/>

Continue to [Browser Support](./browser-support.md)
