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


### Hydration

If you're using [server-side rendering](#server-side-rendering), it's likely that you'll need to create a client-side version of your app *on top of* the server-rendered version. A naive way to do that would involve removing all the existing DOM and rendering the client-side app in its place:

```js
import App from './App.html';

const target = document.querySelector('#element-with-server-rendered-html');

// avoid doing this!
target.innerHTML = '';
new App({
	target
});
```

Ideally, we want to reuse the existing DOM instead. This process is called *hydration*. First, we need to tell the compiler to include the code necessary for hydration to work by passing the `hydratable: true` option:

```js
const { code } = svelte.compile(source, {
	hydratable: true
});
```

(Most likely, you'll be passing this option to [rollup-plugin-svelte](https://github.com/rollup/rollup-plugin-svelte) or [svelte-loader](https://github.com/sveltejs/svelte-loader).)

Then, when we instantiate the client-side component, we tell it to use the existing DOM with `hydrate: true`:

```js
import App from './App.html';

const target = document.querySelector('#element-with-server-rendered-html');

new App({
	target,
	hydrate: true
});
```

> It doesn't matter if the client-side app doesn't perfectly match the server-rendered HTML — Svelte will repair the DOM as it goes.
