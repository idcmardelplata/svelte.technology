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

Si prefiere usar una extensión de archivo diferente, o necesita usar [stores](#state-management), puede pasar opciónes como:

```js
require('svelte/ssr/register')({
	extensions: ['.svelte'],
	store: true
});
```


### Server-side API

Los componentes tienen una API diferente en Node.js - en lugar de crear instancias con los metodos `set(...)` y `get(...)`, un componente es un objeto con un metodo  `render(data, options)`:

```js
require('svelte/ssr/register');
const Thing = require('./components/Thing.html');

const data = { answer: 42 };
const { html, css, head } = Thing.render(data);
```

Todos sus [default data](#default-data),[computed properties](#computed-properties), [helpers](#helpers) y [nested components](#nested-components) funcionaran como espera.

Cualquier función `oncreate` o metodos de componente *no* se ejecutara - puesto que estos solo se aplican a los componentes del lado del cliente.

> El compilador SSR generara un modulo CommonJS para cada uno de sus componentes - lo que significa que las intrucciónes `import` y `export` se convertiran en sus equivalentes `require` y `module.exports`. Si sus componentens no tienen dependencias de otros componentes, tambien pueden funcionar como modulos CommonJS en Node. Si esta usando modulos ES2015, le recomendamos [@std/esm](https://github.com/standard-things/esm) para convertirlos automaticamente a CommonJS.



#### Usando Stores

Si sus componentes usan [stores](#state-management), use el segundo argumento:


```js
const { Store } = require('svelte/store.umd.js');

const { html } = Thing.render(data, {
	store: new Store({
		foo: 'bar'
	})
});
```


#### Renderizando estilos

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
