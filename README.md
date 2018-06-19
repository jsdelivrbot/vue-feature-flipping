# vue-feature-flipping

[Vue.js](https://vuejs.org) plugin providing a set of components used to introduce ["feature flipping"](https://en.wikipedia.org/wiki/Feature_toggle) into your project.

## Why use this plugin?

The **build** of the application packaged **into bundle** the *configuration file* (`config/*.env.js` constants) and *environment variables* (`process.env.*` usages).
So to modify any value, you have to re-build the application.

When you have to enable or disable a feature, the update must be *easy* and *instantaneous*.

This plugin solve this problem.

## How it works?

All feature flags (list of keys of type `string`) are stored into application.  
This list is dynamically initialized at startup (by `init` config function).  
Components use this list to define if action can be done (DOM can be shown, route is accessible, ...).

## Usage

Install NPM dependency:
```shell
npm install --save vue-feature-flipping
```

### Register plugin

```javascript
import Vue from 'vue'
import FeatureFlipping from 'vue-feature-flipping'

Vue.use(FeatureFlipping, {
    init: (consumer) => consumer([])
})
```

### Feature list register

The `init` configuration field can be used to define the feature list.  
The parameter is a consumer who accept a `string[]`.
 
```javascript
Vue.use(FeatureFlipping, {
    init: (consumer) => consumer(['FF1', 'FF2', 'FF3'])
})
```

You can dynamically refresh the list, using `socket.io` or `setInterval` like this example:
```javascript
Vue.use(FeatureFlipping, {
    init: async (consumer) => {
        setInterval(
            async () => consumer(await getFeaturesFromBackend()),
            60000
        )
    }
})
```

### Conditional feature

#### Service

A **function** is defined to check a feature.  
If the feature is not enabled, the function returns `false`.

```javascript
import { isEnabled } from 'vue-feature-flipping'

if (isEnabled('XXX')) {
    // ...
}
```
