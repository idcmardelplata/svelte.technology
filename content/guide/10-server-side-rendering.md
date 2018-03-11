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

Tambien puede extraer cualquier [scoped styles](#scoped-styles) que se utilicen por el componente o sus hijos:

```js
const { css } = Thing.render(data);
```

Podria poner el `css` resultante en una hoja de estilo separada, o incluirlos en la pagina dentro de un tag `<style>`. Si hace esto, probablemente desee evitar que el compilador del lado del cliente vuelva a incluir el CSS. Para  `svelte-cli`, use el flag `no-css`. En integraciones de bundlers como `rollup-plugin-svelte`, pase la opción `css: false`


#### Renderizando el contenido de `<head>`

Si su componente o cualquiera de sus hijos, usa el componente `<:Head>` [component](#-head-tags), puede extraer los contenidos:

```js
const { head } = Thing.render(data);
```
