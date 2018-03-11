---
title: Avanzado
---


### Keyed each blocks

Asociar una *clave* con un bloque le permite a Svelte ser mas inteligente sobre como agrega y quita elementos hacia y desde una lista. Para acerlo, agregue `@key` hasta el final de la etiqueta de apertura para cada bloque, donde `key` es una propiedad que identifica de manera unica a cada miembro de la lista:

```html-no-repl
{{#each people as person @name}}
	<div>{{person.name}}</div>
{{/each}}
```

Es mas facil mostrar el efecto de esto que describirlo. Habra el siguiente ejemplo de esto en el REPL:

```html
<button on:click='update()'>update</button>

<section>
	<h2>Keyed</h2>
	{{#each people as person @name}}
		<div transition:slide>{{person.name}}</div>
	{{/each}}
</section>

<section>
	<h2>Non-keyed</h2>
	{{#each people as person}}
		<div transition:slide>{{person.name}}</div>
	{{/each}}
</section>

<style>
	button {
		display: block;
	}

	section {
		width: 20em;
		float: left;
	}
</style>

<script>
	import { slide } from 'svelte-transitions';

	var people = ['Alice', 'Barry', 'Cecilia', 'Douglas', 'Eleanor', 'Felix', 'Grace', 'Horatio', 'Isabelle'];

	function random() {
		return people
			.filter(() => Math.random() < 0.5)
			.map(name => ({ name }))
	}

	export default {
		data() {
			return { people: random() };
		},

		methods: {
			update() {
				this.set({ people: random() });
			}
		},

		transitions: { slide }
	};
</script>
```


### Hidratación

Si esta usando [server-side rendering](#server-side-rendering), es probable que deba crear una version del lado del cliente de la aplicación *encima de* la versión procesada por el servidor. Una forma ingenua de hacerlo seria eliminar todo el DOM existente y renderizar la aplicación en el lado del cliente:

```js
import App from './App.html';

const target = document.querySelector('#element-with-server-rendered-html');

// avoid doing this!
target.innerHTML = '';
new App({
	target
});
```

Idealmente queremos reusar el DOM existente en su lugar. Este proceso se llama *hydration*. Primero, necesitamos decirle al compilador que incluya el código necesario para que la hidratación funcióne pasando la opcion `hydratable: true`:

```js
const { code } = svelte.compile(source, {
	hydratable: true
});
```

(Lo mas probable es que pase esta opción al [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) o al [svelte-loader](https://github.com/sveltejs/svelte-loader).)


Entonces, cuando instanciamos el componente en el lado del cliente, le decimos que use el DOM existente con `hydrate: true`: 

```js
import App from './App.html';

const target = document.querySelector('#element-with-server-rendered-html');

new App({
	target,
	hydrate: true
});
```

> No importa si la aplicación del lado del cliente no coincide perfectamente con el HTML procesado por el servidor - Svelte reparara el DOM como va.
