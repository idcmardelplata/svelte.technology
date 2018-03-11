---
title: Server-side rendering
---

Hasta aquí, hemos discutido la creación de componentes de Svelte *en el cliente*, es decir en el navegador. Pero también puede renderizar componentes Svelte en Node.js. Esto puede dar como resultado una mejor percepción del rendimiento, ya que significa que la aplicación comienza a renderizarse mientras la página aún se esta descargando, antes de que se ejecute JavaScript. Tambien tiene ventajas de SEO en algunos casos, y puede ser beneficioso para las personas que utilizan navegadores antiguos que no pueden o no ejecutan su codigo javascript por el motivo que sea.


### Usando el compilador.

Si esta usando el compilador de Svelte, ya sea directamente o a trabes de una itegración como: rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) o [svelte-loader](https://github.com/sveltejs/svelte-loader), puede decirle a Svelte que genere un componente del lado del servidor pasando la opción `generate: 'ssr'`al compilador:

```js
const { code } = svelte.compile(source, {
	generate: 'ssr' // as opposed to 'dom', the default
});
```


### Registrando Svelte

Alternativamente, una forma facíl de usar ssr es *registrandolo*:

```js
require('svelte/ssr/register');
```

Ahora puede *requerir* sus componentes:

```js
const Thing = require('./components/Thing.html');
```

If you prefer to use a different file extension, or need to use [stores](#state-management), you can pass options like so:

```js
require('svelte/ssr/register')({
	extensions: ['.svelte'],
	store: true
});
```


### Server-side API

Components have a different API in Node.js – rather than creating instances with `set(...)` and `get(...)` methods, a component is an object with a `render(data, options)` method:

```js
require('svelte/ssr/register');
const Thing = require('./components/Thing.html');

const data = { answer: 42 };
const { html, css, head } = Thing.render(data);
```

All your [default data](#default-data), [computed properties](#computed-properties), [helpers](#helpers) and [nested components](#nested-components) will work as expected.

Any `oncreate` functions or component methods will *not* run — these only apply to client-side components.

> The SSR compiler will generate a CommonJS module for each of your components – meaning that `import` and `export` statements are converted into their `require` and `module.exports` equivalents. If your components have non-component dependencies, they must also work as CommonJS modules in Node. If you're using ES2015 modules, we recommend [@std/esm](https://github.com/standard-things/esm) for automatically converting them to CommonJS.



#### Using stores

If your components use [stores](#state-management), use the second argument:

```js
const { Store } = require('svelte/store.umd.js');

const { html } = Thing.render(data, {
	store: new Store({
		foo: 'bar'
	})
});
```


#### Rendering styles

You can also extract any [scoped styles](#scoped-styles) that are used by the component or its children:

```js
const { css } = Thing.render(data);
```

You could put the resulting `css` in a separate stylesheet, or include them in the page inside a `<style>` tag. If you do this, you will probably want to prevent the client-side compiler from including the CSS again. For `svelte-cli`, use the `--no-css` flag. In build tool integrations like `rollup-plugin-svelte`, pass the `css: false` option.



#### Rendering `<head>` contents

If your component, any of its children, use the `<:Head>` [component](#-head-tags), you can extract the contents:

```js
const { head } = Thing.render(data);
```
