---
title: Preprocesadores
---

A algunos desarrolladores les gusta usar lenguajes no estandar como  [Pug](https://pugjs.org/api/getting-started.html), [Sass](http://sass-lang.com/) o [CoffeeScript](http://coffeescript.org/).

Es posible usar esos lenguajes, o cualquier otra cosa que se pueda convertir a HTML, CSS y JavaScript, usando *preprocesadores*.

### svelte.preprocess

Svelte exporta una funcion llamada `preprocess` que toma un codigo fuente de entrada y devuelve un Promise para un componente estandar de Svelte, listo para ser utilizado con `svelte.compile`:

```js
const svelte = require('svelte');

const input = fs.readFileSync('App.html', 'utf-8');

svelte.preprocess(input, {
	filename: 'App.html', // this is passed to each preprocessor

	markup: ({ content, filename }) => {
		return {
			code: '<!-- some HTML -->',
			map: {...}
		};
	},

	style: ({ content, attributes, filename }) => {
		return {
			code: '/* some CSS */',
			map: {...}
		};
	},

	script: ({ content, attributes, filename }) => {
		return {
			code: '// some JavaScript',
			map: {...}
		};
	}
}).then(preprocessed => {
	fs.writeFileSync('preprocessed/App.html', preprocessed.toString());

	const { code } = svelte.compile(preprocessed);
	fs.writeFileSync('compiled/App.js', code);
});
```

El preprocesador `markup`, si se especifica, corre primero. La propiedad `content` representa todo el string de entrada.

Los preprocesadores `style` y `script` reciven el contenido de los elementos `<style>` y `<script>` respectivamente, junto con cualquier atributo en esos elementos (ej `<style lang='scss'>`). 

Los tres preprocesadores son opcionales. Cada uno deve devolver un objeto `{ code, map }` o una Promise que se resuelve en un objeto `{ code, map }`, donde `code` es la cadena resultante y `map` es un sourcemap representando la transformación.

> Los objetos `map` retornados actualmente no son utilizados por Svelte, pero lo seran en futuras versiones


### Usando herramientas de compilacipon

Cualquier complemento de herramientas de compilación, tales como [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) y [svelte-loader](https://github.com/sveltejs/svelte-loader), le permiten especificar las opciones `preprocess`, en cuyo caso la herramienta de compilación funcionara a la perfección.
