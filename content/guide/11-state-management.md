---
title: Manejo del estado
---

Los componentes de Svelte tienen una administracion de estado incorporada mediante los metodos `get`, `set` y `observe`. Pero a medida que su aplicación crece más alla de un cierto tamaño, puede encontrar que pasar los datos entre los componentes se vuelve una tarea laboriosa.

Por ejemplo, es posible que tenga un componente `<Options>` dentro de un componente `<Sidebar>` que le permite al usuario controlar el comportamiento de un componente `<MainView>`. Podria usar bindings o eventos para 'enviar' información desde `<Options>` mediante `<Sidebar>` a un antecesor común digamos `<App>` - que luego tendria la responsabilidad de enviarlo a `<MainView>`. Pero eso es engorroso, especialmente si decides que quieres romper `<MainView>` en un puñado de componentes mas pequeños

En cambio, una solución popular a este problema es usar un *almacen global* de datos que atraviesa la jerarquia de los componentes. 
React tiene [Redux](https://redux.js.org/) y [MobX](https://mobx.js.org/index.html) ( aunque esas librerias se pueden usar en cualquier lugar, incluso en Svelte), y Vue tiene [Vuex](https://vuex.vuejs.org/en/).

Svelte tiene `Store`. `Store` se puede utilizar en cualquier aplicación de JavaScript, pero es particularmente adecuado para las aplicaciónes de Svelte.

### Lo Básico

Import `Store` from `svelte/store.js` (remember to include the curly braces, as it's a *named import*), then create a new store with some (optional) data:

```js
import { Store } from 'svelte/store.js';

const store = new Store({
	name: 'world'
});
```

Each instance of `Store` has `get`, `set` and `observe` methods that behave identically to their counterparts on a Svelte component:

```js
store.get('name'); // 'world'

store.observe('name', name => {
	console.log(`hello ${name}`);
}); // 'hello world'

store.set({ name: 'everybody' }); // 'hello everybody'
```



### Creating components with stores

Let's adapt our [very first example](#understanding-svelte-components):

```html-no-repl
<!-- App.html -->
<h1>Hello {{$name}}!</h1>
<Greeting/>

<script>
	import Greeting from './Greeting.html';
	export default {
		components: { Greeting }
	};
</script>
```

```html-no-repl
<!-- Greeting.html -->
<p>It's nice to see you, {{$name}}</p>
```

```js
// main.js
import App from './App.html';
import { Store } from 'svelte/store.js';

const store = new Store({
	name: 'world'
});

const app = new App({
	target: document.querySelector('main'),
	store
});

window.store = store; // useful for debugging!
```

There are three important things to notice:

* We're passing `store` to `new App(...)` instead of `data`
* The template refers to `$name` instead of `name`. The `$` prefix tells Svelte that `name` is a *store property*
* Because `<Greeting>` is a child of `<App>`, it also has access to the store. Without it, `<App>` would have to pass the `name` property down as a component property (`<Greeting name='{{name}}'/>`)

Components that depend on store properties will re-render whenever they change.

> To tell Svelte that you're going to be using `Store`, you must pass the `store: true` compiler option. This will change in version 2, when the compiler will automatically generate store-aware components.


### Declarative stores

As an alternative to adding the `store` option when instantiating, the component itself can declare a dependency on a store:

```html
<!-- App.html -->
<h1>Hello {{$name}}!</h1>
<Greeting/>

<script>
	import Greeting from './Greeting.html';
	import store from './store.js';

	export default {
		store: () => store,
		components: { Greeting }
	};
</script>
```

Note that the `store` option is a function that *returns* a store, rather than the store itself — this provides greater flexibility.


### Computed store properties

Just like components, stores can have computed properties:

```js
store = new Store({
	width: 10,
	height: 10,
	depth: 10,
	density: 3
});

store.compute(
	'volume',
	['width', 'height', 'depth'],
	(width, height, depth) => width * height * depth
);

store.get('volume'); // 1000

store.set({ width: 20 });
store.get('volume'); // 2000

store.compute(
	'mass',
	['volume', 'density'],
	(volume, density) => volume * density
);

store.get('mass'); // 6000
```

The first argument is the name of the computed property. The second is an array of *dependencies* — these can be data properties or other computed properties. The third argument is a function that recomputes the value whenever the dependencies change.

A component that was connected to this store could reference `{{$volume}}` and `{{$mass}}`, just like any other store property.


### Accessing the store inside components

Each component gets a reference to `this.store`. This allows you to attach behaviours in `oncreate`...

```html-no-repl
<script>
	export default {
		oncreate() {
			this.store.observe('foo', foo => {
				// ...
			});
		}
	};
</script>
```

...or call store methods in your event handlers:

```html-no-repl
<button on:click='store.set({ muted: true })'>
	Mute audio
</button>
```


### Custom store methods

`Store` doesn't have a concept of *actions* or *commits*, like Redux and Vuex. Instead, state is always updated with `store.set(...)`.

You can implement custom logic by subclassing `Store`:

```js
class TodoStore extends Store {
	addTodo(description) {
		const todo = {
			id: generateUniqueId(),
			done: false,
			description
		};

		const todos = this.get('todos').concat(todo);
		this.set({ todos });
	}

	toggleTodo(id) {
		const todos = this.get('todos').map(todo => {
			if (todo.id === id) {
				return {
					id,
					done: !todo.done,
					description: todo.description
				};
			}

			return todo;
		});

		this.set({ todos });
	}
}

const store = new TodoStore({
	todos: []
});

store.addTodo('Finish writing this documentation');
```

Methods can update the store asynchronously:

```js
class NasdaqTracker extends Store {
	async fetchStockPrices(ticker) {
		const token = this.token = {};
		const prices = await fetch(`/api/prices/${ticker}`).then(r => r.json());
		if (token !== this.token) return; // invalidated by subsequent request

		this.set({ prices });
	}
}

const store = new NasdaqTracker();
store.fetchStockPrices('AMZN');
```

You can call these methods in your components, just like the built-in methods:


```html-no-repl
<input
	placeholder='Enter a stock ticker'
	on:change='store.fetchStockPrices(this.value)'
>
```

### Two-way store bindings

You can bind to store properties just like you bind to component properties — just add the `$` prefix:

```html-no-repl
<!-- global audio volume control -->
<input bind:value=$volume type=range min=0 max=1 step=0.01>
```


### Using store properties in computed properties

Just as in templates, you can access store properties in component computed properties by prefixing them with `$`:

```html-no-repl
<!-- Todo.html -->
{{#if isVisible}}
	<div class='todo {{todo.done ? "done": ""}}'>
		{{todo.description}}
	</div>
{{/if}}

<script>
	export default {
		computed: {
			// `todo` is a component property, `$filter` is
			// a store property
			isVisible: (todo, $filter) => {
				if ($filter === 'all') return true;
				if ($filter === 'done') return todo.done;
				if ($filter === 'pending') return !todo.done;
			}
		}
	};
</script>
```


### The onchange method

In addition to `get`, `set`, `observe` and `compute`, each `Store` instance also has an `onchange` method. Every time the state changes, callbacks will receive a copy of the state, and an object indicating which properties have changed:

```js
store.set({ foo: 1, bar: 2, baz: 3, qux: 4 });
store.onchange((state, changed) => {
	console.log(`These properties changed: ${Object.keys(changed).join(', ')}`);
});

store.set({ bar: 3, baz: 3, qux: 3 });
// -> 'These properties changed: bar, qux'
```

This is useful for attaching generic behaviours to stores — for example, you could create a function that persisted the contents of a store to `localStorage`:

```js
function useLocalStorage(store, key) {
	const json = localStorage.getItem(key);
	if (json) {
		store.set(JSON.parse(json));
	}

	store.onchange(state => {
		localStorage.setItem(key, JSON.stringify(state));
	});
}

useLocalStorage(store, 'my-key');
```

Note that `Store` is conservative about objects and arrays, because there is no easy way to know if they have been mutated:

```js
const object = {};

store.onchange(() => console.log('something changed'));

store.set({ object }); // 'something changed'
store.set({ object }); // 'something changed' (even though it's the same value)
```


### Built-in optimisations

The Svelte compiler knows which store properties your components are interested in (because of the `$` prefix), and writes code that only listens for changes to those properties. Because of that, you needn't worry about having many properties on your store, even ones that update frequently — components that don't use them will be unaffected.

