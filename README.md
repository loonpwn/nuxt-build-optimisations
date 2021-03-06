![](https://laravel-og.beyondco.de/Nuxt%20Build%20Optimisations.png?theme=light&packageManager=yarn+add&packageName=nuxt-build-optimisations&pattern=texture&style=style_1&description=Instantly+speed+up+your+Nuxt+v2+build+times.&md=1&showWatermark=0&fontSize=100px&images=lightning-bolt)

<h1 align='center'><samp>nuxt-build-optimisations</samp></h2>

<p align="center">
  <a href="https://github.com/harlan-zw/nuxt-build-optimisations/actions"><img src="https://github.com/harlan-zw/nuxt-build-optimisations/actions/workflows/test.yml/badge.svg" alt="builder"></a>
  <a href="https://npmjs.com/package/nuxt-build-optimisations"><img src="https://img.shields.io/npm/v/nuxt-build-optimisations.svg" alt="npm package"></a>
</p>

<p align='center'>Instantly speed up your Nuxt.js v2 build times.</p>


## About

### Why do I need this?

Nuxt.js is fast but is limited by its webpack build, when your app grows things slow down.

Nuxt build optimisations abstracts the complexities of optimising your Nuxt.js app so anyone can instantly speed up their builds
without having to learn webpack. The focus is primarily on the development build, as the optimisations are safer.

For the best possible performance, consider using: [Nuxt Vite](https://vite.nuxtjs.org/). This package is for webpack stuck projects. 

### How fast is it?

**Development**: :snowman: **2-5x** quicker cold starts, :fire: almost instant hot starts (with "risky" profile)

**Production**: Should be a slight performance improvement depending on profile.

## Features

Features are enabled by their risk profile. The risk profile is the likelihood of issues coming up.

**Safe (safest)**

- Development: Super quick js/ts transpiling with [esbuild](https://esbuild.github.io/) :zap:
- Development: Images only use `file-loader`
- webpack benchmarking with [speed-measure-webpack-plugin](https://github.com/stephencookdev/speed-measure-webpack-plugin)

**Experimental (mostly safe)**
- Development: Disables [postcss-preset-env](https://github.com/csstools/postcss-preset-env) pollyfills
- Replaces [Terser](https://github.com/terser/terser) minification with [esbuild](https://esbuild.github.io/)
- Enable [Nuxt build cache](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-build#cache)
- webpack's [best practices for performance](https://webpack.js.org/guides/build-performance/)
- Disables Nuxt features that aren't used (layouts, store)

**Risky (may throw errors)**
- Enable [Nuxt parallel](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-build#parallel)
- Enable [Nuxt hard source](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-build#hardsource)


## Setup

Install using yarn or npm. (Nuxt.js 2.10+ is required)

```bash
yarn add nuxt-build-optimisations
```

```bash
npm i nuxt-build-optimisations
```

:warning: This package makes optimisations with the assumption you're developing on the latest chrome.

---

## Usage

Within your `nuxt.config.js` add the following.

```js
// nuxt.config.js
buildModules: [
  'nuxt-build-optimisations',
],
```

It's recommended you start with the default configuration, which is the `experimental` profile.

However if you'd like to try and get more performance you can try the following:


```js
// nuxt.config.js
buildOptimisations: {
  profile: process.env.NODE_ENV === 'development' ? 'risky' : 'experimental'
},
```

⚠️ Note: The risky profile uses [HardSource](https://github.com/mzgoddard/hard-source-webpack-plugin) caching, if you use it in your production CI with node / npm caching then you need to make sure it caches per branch.

A lot of the speed improvements are from heavy caching, if you have any issues the first thing you should
do is clear your cache.

```shell
rm -rf node_modules/.cache

//windows
rd /s  "node_modules/.cache"
```


## Configuration

### Profile

*Type:* `risky` | `experimental` | `safe` | `false`

*Default:* `experimental`

If you have errors on any mode you should increment down in profiles until you find one that works.

Setting the profile to false will disable the optimisations, useful when you want to measure your build time without optimisations.


### Measure

*Type:* `boolean` or `object`

*Default:* `false`

When measure is enabled with true (options or environment variable), it will use the `speed-measure-webpack-plugin`.

If the measure option is an object it is assumed to be [speed-measure-webpack-plugin options](https://github.com/stephencookdev/speed-measure-webpack-plugin#options).

```js
buildOptimisations: {
  measure: {
    outputFormat: 'humanVerbose',
    granularLoaderData: true,
    loaderTopFiles: 10
  }
}
```

You can use an environment variable to enable the measure as well.

**package.json**

```json
{
  "scripts": {
    "measure": "export NUXT_MEASURE=true; nuxt dev"
  }
}
```

Note: Some features are disabled with measure on, such as caching.

### Measure Mode

*Type:* `client` | `server` | `modern` | `all`

*Default:* `client`

Configure which build will be measured. Note that non-client builds may be buggy and mess with HMR.

```javascript
buildOptimisations: {
  measureMode: 'all'
}
```

### Feature Flags

*Type:*  `object`

*Default:*
```js
// uses esbuild loader
esbuildLoader: boolean
// uses esbuild as a minifier
esbuildMinifier: boolean
// swaps url-loader for file-loader
imageFileLoader: boolean
// misc webpack optimisations
webpackOptimisations: boolean
// no polyfilling css in development
postcssNoPolyfills: boolean
// inject the webpack cache-loader loader
cacheLoader: boolean
// use the hardsource plugin
hardSourcePlugin: boolean
// use the parallel thread plugin
parallelPlugin: boolean
```

You can disable features if you'd like to skip optimisations.

```js
buildOptimisations: {
  features: {
    // use url-loader
    imageFileLoader: false
  }
}
```

### esbuildLoaderOptions

*Type:*  `object` or `(args) => object`

*Default:*
```javascript
{
  target: 'es2015'
}
```

See [esbuild-loader](https://github.com/privatenumber/esbuild-loader).

### esbuildMinifyOptions

*Type:*  `object` or `(args) => object`

*Default:*
```javascript
{
  target: 'es2015'
}
```

See [esbuild-loader](https://github.com/privatenumber/esbuild-loader).



### Gotchas

#### Vue Property Decorator / Vue Class Component

Your babel-loader will be replaced with esbuild, which doesn't support class decorators in js.

You can either migrate your scripts to typescript or disabled the esbuild loader.

**Disable Loader**
```js
buildOptimisations: {
  features: {
    esbuildLoader: false
  }
}
```

**Migrate to TypeScript**

*tsconfig.json*
```json
{
  "experimentalDecorators": true
}
```

```vue
<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export default class HelloWorld extends Vue {
  data () {
    return {
      hello: 'test'
    }
  }
}
</script>
```


## Credits

- https://github.com/galvez/nuxt-esbuild-module

## License

[MIT](LICENSE)
