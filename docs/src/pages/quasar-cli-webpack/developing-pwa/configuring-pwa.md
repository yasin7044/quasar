---
title: Configuring PWA
desc: (@quasar/app-webpack) How to manage your Progressive Web Apps with Quasar CLI.
related:
  - /quasar-cli-webpack/quasar-config-file
scope:
  tree:
    l: src-pwa
    c:
    - l: register-service-worker.js
      e: "(or .ts) App-code *managing* service worker"
    - l: custom-service-worker.js
      e: "(or .ts) Optional custom service worker file (InjectManifest mode ONLY)"
---

## Service Worker
Adding PWA mode to a Quasar project means a new folder will be created: `/src-pwa`, which contains PWA specific files:

<DocTree :def="scope.tree" />

You can freely edit these files. Notice a few things:

1. `register-service-worker.[js|ts]` is automatically imported into your app (like any other /src file). It registers the service worker (created by Workbox or your custom one, depending on workbox plugin mode -- quasar.config file > pwa > workboxPluginMode) and you can listen for Service Worker's events. You can use ES6 code.
2. `custom-service-worker.[js|ts]` will be your service worker file ONLY if workbox plugin mode is set to "InjectManifest" (quasar.config file > pwa > workboxPluginMode: 'InjectManifest'). Otherwise, Workbox will create a service-worker file for you.
3. It makes sense to run [Lighthouse](https://developers.google.com/web/tools/lighthouse/) tests on production builds only.

::: tip
Read more on `register-service-worker.[js|ts]` and how to interact with the Service Worker on [Handling Service Worker](/quasar-cli-webpack/developing-pwa/handling-service-worker) documentation page.
:::

## quasar.config file
This is the place where you can configure Workbox behavior and also tweak your manifest.json.

```js
pwa: {
  // workboxPluginMode: 'InjectManifest',
  // workboxOptions: {},
  manifest: {
    // ...
  },

  // Use this OR metaVariablesFn, but not both;
  // variables used to inject specific PWA
  // meta tags (below are default values);
  metaVariables: {
    appleMobileWebAppCapable: 'yes',
    appleMobileWebAppStatusBarStyle: 'default',
    appleTouchIcon120: 'icons/apple-icon-120x120.png',
    appleTouchIcon180: 'icons/apple-icon-180x180.png',
    appleTouchIcon152: 'icons/apple-icon-152x152.png',
    appleTouchIcon167: 'icons/apple-icon-167x167.png',
    appleSafariPinnedTab: 'icons/safari-pinned-tab.svg',
    msapplicationTileImage: 'icons/ms-icon-144x144.png',
    msapplicationTileColor: '#000000'
  },

  // Optional, overrides metaVariables above;
  // Use this OR metaVariables, but not both;
  metaVariablesFn (manifest) {
    // ...
    return [
      {
        // this entry will generate:
        // <meta name="theme-color" content="ff0">

        tagName: 'meta',
        attributes: {
          name: 'theme-color',
          content: '#ff0'
        }
      },

      {
        // this entry will generate:
        // <link rel="apple-touch-icon" sizes="180x180" href="icons/icon-180.png">
        // references /public/icons/icon-180.png

        tagName: 'link',
        attributes: {
          rel: 'apple-touch-icon',
          sizes: '180x180',
          href: 'icons/icon-180.png'
        },
        closeTag: false // this is optional;
                        // specifies if tag also needs an explicit closing tag;
                        // it's Boolean false by default
      }
    ]
  },

  // optional; webpack config Object for
  // the custom service worker ONLY (/src-pwa/custom-service-worker.[js|ts])
  // if using workbox in InjectManifest mode
  extendWebpackCustomSW (cfg) {
    // directly change props of cfg;
    // no need to return anything
  },

  // optional; EQUIVALENT to extendWebpackCustomSW() but uses webpack-chain;
  // for the custom service worker ONLY (/src-pwa/custom-service-worker.[js|ts])
  // if using workbox in InjectManifest mode
  chainWebpackCustomSW (chain) {
    // chain is a webpack-chain instance
    // of the Webpack configuration

    // example:
    // chain.plugin('eslint-webpack-plugin')
    //   .use(ESLintPlugin, [{ extensions: [ 'js' ] }])
  }
}
```

More information: [Workbox Webpack Plugin](https://developer.chrome.com/docs/workbox/modules/workbox-webpack-plugin/), [Workbox](https://developer.chrome.com/docs/workbox/).

The `metaVariables` Object is used by Quasar itself only (has no meaning for Workbox) to inject specific value attributes to some PWA meta tags into the rendered HTML page. Example: `<meta name="apple-mobile-web-app-status-bar-style">` will have value attribute assigned to the content of `metaVariables.appleMobileWebAppStatusBarStyle`.

You can use an alternative to metaVariables: `metaVariablesFn(manifest)` which can return an Array of Objects (see their form in the code above). Should you configure this function to not return an Array or to return an empty Array, then Quasar App CLI will understand not to add any tags -- so you can manually add your custom tags directly in `/src/index.template.html`.

## Picking Workbox mode

There are two Workbox operating modes: **GenerateSW** (default) and **InjectManifest**. The first one generates a service worker automatically, based on quasar.config file > pwa > workboxOptions (if any), while the second mode allows you to write your own service worker file.

Setting the mode that you want to use is done through the `/quasar.config` file:

```js /quasar.config file
pwa: {
  // workboxPluginMode: 'InjectManifest',
  // workboxOptions: { ... }
}
```

::: danger
Make sure that your `workboxOptions` match the Workbox mode that you have picked, otherwise the workbox webpack plugin might [halt your app from compiling](https://github.com/quasarframework/quasar/issues/4998).
:::

### GenerateSW

When to use GenerateSW:

* You want to precache files.
* You have simple runtime configuration needs (e.g. the configuration allows you to define routes and strategies).

When NOT to use GenerateSW:

* You want to use other Service Worker features (i.e. Web Push).
* You want to import additional scripts or add additional logic.

::: tip
Please check the available workboxOptions for this mode on [Workbox website](https://developers.google.com/web/tools/workbox/modules/workbox-webpack-plugin#full_generatesw_config).
:::

### InjectManifest

When to use InjectManifest:

* You want more control over your service worker.
* You want to precache files.
* You have more complex needs in terms of routing.
* You would like to use your service worker with other APIs (e.g. Web Push).

When NOT to use InjectManifest:

* You want the easiest path to adding a service worker to your site.

::: tip TIPS
* If you want to use this mode, you will have to write the service worker (`/src-pwa/custom-service-worker.[js|ts]`) file by yourself.
* Please check the available workboxOptions for this mode on [Workbox website](https://developers.google.com/web/tools/workbox/modules/workbox-webpack-plugin#full_injectmanifest_config).
:::

The following snippet is the default code for a custom service worker (`/src-pwa/custom-service-worker.[js|ts]`):

```js
import { precacheAndRoute } from 'workbox-precaching'

// Use with precache injection
precacheAndRoute(self.__WB_MANIFEST)
```

## Configuring Manifest File
The Manifest file is generated by Quasar CLI with a default configuration for it. You can however tweak this configuration from the `/quasar.config` file:

```js /quasar.config file
pwa: {
  // workboxPluginMode: 'InjectManifest',
  // workboxOptions: {},
  manifest: {
    name: 'Quasar Play',
    short_name: 'Quasar-Play',
    description: 'Quasar Framework Showcase',
    icons: [
      {
        'src': 'icons/icon-128x128.png',
        'sizes': '128x128',
        'type': 'image/png'
      },
      {
        'src': 'icons/icon-192x192.png',
        'sizes': '192x192',
        'type': 'image/png'
      },
      {
        'src': 'icons/icon-256x256.png',
        'sizes': '256x256',
        'type': 'image/png'
      },
      {
        'src': 'icons/icon-384x384.png',
        'sizes': '384x384',
        'type': 'image/png'
      },
      {
        'src': 'icons/icon-512x512.png',
        'sizes': '512x512',
        'type': 'image/png'
      }
    ],
    display: 'standalone',
    orientation: 'portrait',
    background_color: '#ffffff',
    theme_color: '#027be3'
  }
}
```

Please read about the [manifest config](https://developer.mozilla.org/en-US/docs/Web/Manifest) before diving in.

::: warning
Note that you don't need to edit your index.html file (generated from `/src/index.template.html`) to link to the manifest file. Quasar CLI takes care of embedding the right things for you.
::::

::: tip
If your PWA is behind basic auth or requires an Authorization header, set quasar.config file > pwa > useCredentials to true to include `crossorigin="use-credentials"` on the manifest.json meta tag.
::::

## PWA Checklist
More info: [PWA Checklist](https://web.dev/pwa-checklist/)

::: danger
Do not run [Lighthouse](https://developers.google.com/web/tools/lighthouse/) on your development build because at this stage the code is intentionally not optimized and contains embedded source maps (among many other things). See the [Testing and Auditing](/quasar-cli-webpack/testing-and-auditing) section of these docs for more information.
:::

## Reload & Update Automatically
For those who don't want to manually reload the page when the service worker is updated **and are using the default GenerateSW workbox mode**, you can make it active at once. Update the workboxOptions config in the `/quasar.config` file as follows:

```js /quasar.config file
pwa: {
  workboxOptions: {
    skipWaiting: true,
    clientsClaim: true
  }
}
```

[Source](https://developers.google.com/web/tools/workbox/guides/codelabs/webpack)
