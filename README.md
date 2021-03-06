# vue-feature-flipping

[![Build Status](https://travis-ci.org/pinguet62/vue-feature-flipping.svg?branch=master)](https://travis-ci.org/pinguet62/vue-feature-flipping) 
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/70c3d26abe2047a3a6ca0183ec73421b)](https://app.codacy.com/app/pinguet62/vue-feature-flipping?utm_source=github.com&utm_medium=referral&utm_content=pinguet62/vue-feature-flipping&utm_campaign=badger)
[![Codacy Badge](https://api.codacy.com/project/badge/Coverage/bdbafbe565e04d42ab67b5432980ea89)](https://www.codacy.com/app/pinguet62/vue-feature-flipping?utm_source=github.com&utm_medium=referral&utm_content=pinguet62/vue-feature-flipping&utm_campaign=Badge_Coverage)
[![Demo Badge](https://img.shields.io/badge/demo-JSFiddle-blue.svg)](http://jsfiddle.net/gh/get/library/pure/pinguet62/vue-feature-flipping/tree/master/demo)

[Vue.js](https://vuejs.org) plugin providing a set of components used to introduce ["feature flipping"](https://en.wikipedia.org/wiki/Feature_toggle) into your project.

## Why use this plugin?

The **build** of the application packaged **into bundle** the *configuration file* (`config/*.env.js` constants) and *environment variables* (`process.env.*` usages).
So to modify any value, you have to re-build the application.

When you have to enable or disable a feature, the update must be *easy* and *instantaneous*.

This plugin solve this problem.

## How it works?

All feature flags (list of keys of type `string`) are stored into plugin.  
This list is dynamically initialized at startup (by `setEnabledFeatures()` function).  
Components use this list to define if action can be done (DOM can be shown, route is accessible, ...).

## Configuration

### 1. NPM module

Install NPM dependency:

```shell
npm install --save vue-feature-flipping
```

### 2. Plugin registration

Register all Vue.js components (directive, guard, ...) calling `Vue.use()`:

```javascript
import Vue from 'vue'
import FeatureFlipping from 'vue-feature-flipping'

Vue.use(FeatureFlipping)
```

### 3. Features list registration

The `setEnabledFeatures(string[])` function can be used to define the feature list.

```javascript
import { setEnabledFeatures } from 'vue-feature-flipping'

setEnabledFeatures(['FF1', 'FF2', 'FF3'])
```

You can dynamically refresh the list, using `socket.io` or `setInterval` like this example:
```javascript
setInterval(
    async () => setEnabledFeatures(await getFeaturesFromBackend('http://localhost:8081')),
    60000
)
```

### Usage

#### Service

A **function** is defined to check a feature.  
If the feature is not enabled, the function returns `false`.

```javascript
import { isEnabled } from 'vue-feature-flipping'

if (isEnabled('XXXXX')) {
    // ...
}

if (isEnabled('XXXXX', true)) {
    // ...
}
```

#### Directive

A [**directive**](https://vuejs.org/v2/guide/custom-directive.html) named `feature-flipping` can be used into `<template>`.

##### Rendering

Without argument, the directive works like `v-if`. If the feature is not enabled, the DOM is removed.

```html
<menu>
    <entry>First</entry>
    <entry v-feature-flipping="'XXXXX'">Second</entry>
    <entry v-feature-flipping.not="'XXXXX'">Third</entry>
    <entry v-feature-flipping.default="'XXXXX'">Fourth</entry>
</menu>
```

##### Class

The argument `class` allow the directive to work like `v-bind:class`. If the feature is enabled, the classes are apply to element.

```html
<menu>
    <entry>First</entry>
    <entry v-feature-flipping:class="{ key: 'XXXXX', value: 'class1' }">Second</entry>
    <entry v-feature-flipping:class="{ key: 'XXXXX', value: ['class1', ['class2'], { 'class3': true }] }">Third</entry>
    <entry v-feature-flipping:class.not="{ key: 'XXXXX', value: 'class1' }">Fourth</entry>
    <entry v-feature-flipping:class.default="{ key: 'XXXXX', value: 'class1' }">Fifth</entry>
</menu>
```

##### Style

The argument `style` allow the directive to work like `v-bind:style`. If the feature is enabled, the styles are apply to element.

```html
<menu>
    <entry>First</entry>
    <entry v-feature-flipping:style="{ key: 'XXXXX', value: { style1: 'value1', style2: 'value2' } }">Second</entry>
    <entry v-feature-flipping:style="{ key: 'XXXXX', value: [{ style1: 'value1' }, { style2: 'value2' }] }">Third</entry>
    <entry v-feature-flipping:style.not="{ key: 'XXXXX', value: { style1: 'value1' } }">Fourth</entry>
    <entry v-feature-flipping:style.default="{ key: 'XXXXX', value: { style1: 'value1' } }">Fifth</entry>
</menu>
```

#### Route

A [**guard**](https://router.vuejs.org/guide/advanced/navigation-guards.html) is defined to intercept all routes defining `featureFlipping` [**meta field**](https://router.vuejs.org/guide/advanced/meta.html).  
If the feature is not enabled, the router redirect to `"/"` route.

```javascript
import VueRouter from 'vue-router'
import { Test1Component, Test2Component, Test3Component } from '...'

new VueRouter({
    routes: [
        { path: '/test1', component: Test1Component, meta: { featureFlipping: { key: 'XXXXX' } } },
        { path: '/test2', component: Test2Component, meta: { featureFlipping: { key: 'XXXXX' }, redirect: '/error' } },
        { path: '/test3', component: Test3Component, meta: { featureFlipping: { key: 'XXXXX', not: true } } },
        { path: '/test4', component: Test4Component, meta: { featureFlipping: { key: 'XXXXX', default: true } } },
    ]
})
```

## Options

### `default`: default behavior

When the plugin is *not initialized*, or when any *error occurs* when user try to initialize this plugin, it's necessary to define a **default behavior**: should we activate the function or should we disable it?

The **default value** defines this behavior: the value is used when plugin is not initialized or initialized with `null`.

Example:
```javascript
try {
    let features = await getFeaturesFromBackend()
    setEnabledFeatures(features)
} catch (e) {
    setEnabledFeatures(null) // use default
}
```

### `not`: reversed rendering

In some cases, we have to define a behavior when the feature is disabled.

The `not` option activate this behavior.

## Best practices

* **Independent** - Avoid behavior depending multiples features.  
    Bad: `if (isEnabled('F1') && isEnabled('F2') || isEnabled('F3')) ...`
