---
title: Plugins
---

Svelte se puede extender con plugins y metodos extra.


### Plugin de transicion.

El paquete [svelte-transitions](https://github.com/sveltejs/svelte-transitions) incluye una selección de complementos de transición admitidos oficialmente, tales como [fade](https://github.com/sveltejs/svelte-transitions-fade), [fly](https://github.com/sveltejs/svelte-transitions-fly) y [slide](https://github.com/sveltejs/svelte-transitions-slide). Puede incluirlos en un componente como tal:

```html
<label>
	<input type='checkbox' bind:checked='visible'> visible
</label>

{{#if visible}}
	<!-- use `in`, `out`, or `transition` (bidirectional) -->
	<div transition:fly='{y:20}'>hello!</div>
{{/if}}

<script>
	import { fly } from 'svelte-transitions';

	export default {
		transitions: { fly }
	};
</script>
```

```hidden-data
{
	"visible": true
}
```


### Metodos extra

El paquete [svelte-extras](https://github.com/sveltejs/svelte-extras) incluye un puñado de metodos para interpolar (animar), manipular arrays etc...

```html
<input bind:value='newTodo' placeholder='buy milk'>
<button on:click='push("todos", newTodo)'>add todo</button>

<ul>
	{{#each todos as todo, i}}
		<li>
			<button on:click='splice("todos", i, 1)'>x</button>
			{{todo}}
		</li>
	{{/each}}
</ul>

<style>
	ul {
		list-style: none;
		padding: 0;
	}

	li button {
		color: rgb(200,0,0);
		background: rgba(200,0,0,0.1);
		border-color: rgba(200,0,0,0.2);
		padding: 0.2em 0.5em;
	}
</style>

<script>
	import { push, splice } from 'svelte-extras';

	export default {
		data() {
			return {
				newTodo: '',
				todos: []
			};
		},

		methods: {
			push,
			splice
		}
	};
</script>
```

```hidden-data
{
	"todos": [
		"wash the car",
		"take the dog for a walk",
		"mow the lawn"
	]
}
```
