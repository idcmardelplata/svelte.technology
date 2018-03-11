---
title: Directivas de elementos
---

Las directivas son instrucciónes a nivel de elemento o componente para Svelte. Se ven como atributos, excepto por el caracter `:`.

### Manejadores de eventos

En la mayoria de las aplicaciónes, debera responder a las acciónes del usuario. En Svelte, esto se hace con la directiva `on:[event]`.

```html
<p>Count: {{count}}</p>
<button on:click='set({ count: count + 1 })'>+1</button>
```

```hidden-data
{
	"count": 0
}
```

Cuando el usuario hace click en el boton, Svelte llama a `component.set(...)`  con los argumentos proporcionados. Puede llamar a cualquier metodo que pertenezca al componente (ya sea [built-in](#component-api) o [personalizado](#custom-methods)), y cualquier propiedad de datos (o propiedad computada) que este dentro del scope:

```html
<p>Select a category:</p>

{{#each categories as category}}
	<button on:click='select(category)'>select {{category}}</button>
{{/each}}

<script>
	export default {
		data() {
			return {
				categories: [
					'animal',
					'vegetable',
					'mineral'
				]
			}
		},

		methods: {
			select(name) {
				alert(`selected ${name}`); // seriously, please don't do this
			}
		}
	};
</script>
```

Tambien puede acceder al objeto `event` en la llamada al metodo:

```html
<div on:mousemove='set({ x: event.clientX, y: event.clientY })'>
	coords: {{x}},{{y}}
</div>

<style>
	div {
		border: 1px solid purple;
		width: 100%;
		height: 100%;
	}
</style>
```

El nodo de destino puede ser referenciado como `this`, lo que significa que puede hacer este tipo de cosas:

```html
<input on:focus='this.select()' value='click to select'>
```

### Eventos personalizados

Usted puede definir sus propios eventos personalizados para manejar interacciónes de usuario complejas tales como drag and drop o arrastrar y deslizar. Consulte la sección anterior en [manejadores de eventos personalizados](#custom-event-handlers) para mas información.


### Eventos de componente

Los eventos son una exelente manera de que los [componentes anidados](#nested-components) se comuniquen con sus padres. Revisemos nuestro ejemplo anterior, pero convertido en un componente llamado `<CategoryChooser>`:

```html-no-repl
<!-- CategoryChooser.html -->
<p>Select a category:</p>

{{#each categories as category}}
	<button on:click='fire("select", { category })'>select {{category}}</button>
{{/each}}

<script>
	export default {
		data() {
			return {
				categories: [
					'animal',
					'vegetable',
					'mineral'
				]
			}
		}
	};
</script>
```

Cuando el usuario hace click en el boton, el componente disparara un evento llamado `select`, donde el objeto `event` tiene una propiedad `category`. Cualquier componente que anida `<CategoryChooser>` puede escuchar eventos como este:

```html-no-repl
<CategoryChooser on:select='playTwentyQuestions(event.category)'/>

<script>
	import CategoryChooser from './CategoryChooser.html';

	export default {
		components: {
			CategoryChooser
		},

		methods: {
			playTwentyQuestions(category) {
				// TODO implement
			}
		}
	};
</script>
```

Asi como `this` en el controlador de eventos de un elemento se refiere al elemento mismo, en un controlador de eventos de componente `this` hace referencia al componente que disparo el evento.

Tambien existe una forma abreviada de escuchar y volver a disparar un evento sin cambios. `<Widget on:foo/>` es equivalente a `<Widget on:foo="fire('foo', event)"/>`. Dado que los eventos de los componentes no se propagan como eventos DOM, esto se puede usar para pasar eventos a trabes de componentes intermedios.


### Refs

Los Refs son una forma conveniente de almacenar una referencia a determinados nodos del DOM o a componentes. Declare un ref con `ref:[name]`, y acceda a ella dentro de los metodos de su componente con `this.refs.[name]`:

```html
<canvas ref:canvas width='200' height='200'></canvas>

<script>
	export default {
		oncreate() {
			const canvas = this.refs.canvas;
			const ctx = canvas.getContext('2d');

			let destroyed = false;
			this.on('destroy', () => destroyed = true);

			function loop() {
				if (destroyed) return;
				requestAnimationFrame(loop);

				const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

				for (let p = 0; p < imageData.data.length; p += 4) {
					const i = p / 4;
					const x = i % canvas.width;
					const y = i / canvas.height >>> 0;

					const t = window.performance.now();

					const r = 64 + (128 * x / canvas.width) + (64 * Math.sin(t / 1000));
					const g = 64 + (128 * y / canvas.height) + (64 * Math.cos(t / 1000));
					const b = 128;

					imageData.data[p + 0] = r;
					imageData.data[p + 1] = g;
					imageData.data[p + 2] = b;
					imageData.data[p + 3] = 255;
				}

				ctx.putImageData(imageData, 0, 0);
			}

			loop();
		}
	}
</script>
```

> Dado que solo un elemento o un componente puede ocupar un `ref` especificado, no los use en bloques `{{#each ...}}`. Sin embargo, esta bien usarlos en bloques `{{#if ...}}`.


### Transiciones

Las transiciones permiten a los elementos entrar y salir del DOM con gracia, en lugar de aparecer y desaparecer de repente.

```html
<input type='checkbox' bind:checked=visible> visible

{{#if visible}}
	<p transition:fade>fades in and out</p>
{{/if}}

<script>
	import { fade } from 'svelte-transitions';

	export default {
		transitions: { fade }
	};
</script>
```

Las transiciones pueden tener parametros - tipicamente `delay` y `duration`, pero tambien otros, dependiendo de la transición en question. Por ejemplo, aqui esta la transición `fly` del paquete [svelte-transitions](https://github.com/sveltejs/svelte-transitions):

```html
<input type='checkbox' bind:checked=visible> visible

{{#if visible}}
	<p transition:fly='{y: 200, duration: 1000}'>flies 200 pixels up, slowly</p>
{{/if}}

<script>
	import { fly } from 'svelte-transitions';

	export default {
		transitions: { fly }
	};
</script>
```

Un elemento puede tener transiciónes separadas para `in` y `out`:

```html
<input type='checkbox' bind:checked=visible> visible

{{#if visible}}
	<p in:fly='{y: 50}' out:fade>flies up, fades out</p>
{{/if}}

<script>
	import { fade, fly } from 'svelte-transitions';

	export default {
		transitions: { fade, fly }
	};
</script>
```

Las transiciones son funciones simples que toman un `nodo` y cualquier `parametro` provisto y retornan un objeto con las siguientes propiedades:

* `duration` — Cuanto demora la transición en milisegundos
* `delay` — milisegundos antes de que la transición comiense.
* `easing` — una [easing function](https://github.com/rollup/eases-jsnext)
* `css` — una función que acepta un argumento `t` entre `0` y `1`y retorna  los estilos que deberian aplicarse en ese momento
* `tick` — una función que se llamara en cada frame, con el mismo argumento `t`, mientras que la transicion esta en progreso

De estos, se require `duration`, como `css` oo `tick`. El resto es opcional. Asi es como se implementa la transición `fade` por ejemplo:

```html
<input type='checkbox' bind:checked=visible> visible

{{#if visible}}
	<p transition:fade>fades in and out</p>
{{/if}}

<script>
	export default {
		transitions: {
			fade(node, { delay = 0, duration = 400 }) {
				const o = +getComputedStyle(node).opacity;

				return {
					delay,
					duration,
					css: t => `opacity: ${t * o}`
				};
			}
		}
	};
</script>
```

> Si se usa la opción `css`, Svelte creara una animación css que se ejecutara de manera eficiente fuera del thread principal. Por lo tanto, si puede lograr un efecto usando `css` en lugar de `tick` deberia hacerlo.


### Enlace bidireccional

It's currently fashionable to avoid two-way binding on the grounds that it creates all sorts of hard-to-debug problems and slows your application down, and that a one-way top-down data flow is 'easier to reason about'. This is in fact high grade nonsense. It's true that two-way binding done *badly* has all sorts of issues, and that very large apps benefit from the discipline of a not permitting deeply nested components to muck about with state that might affect distant parts of the app. But when used correctly, two-way binding simplifies things greatly.

Bindings are declared with the `bind:[attribute]` directive:

```html
<input bind:value='name' placeholder='enter your name'>
<p>Hello {{name || 'stranger'}}!</p>
```

Here are the current bindable attributes and properties for each element:

- `<input>`, `<textarea>`, `<select>`, `<option>`
	- `value`
- `<input type="checkbox">`, `<input type="radio">`
	- `checked`, `group`
- `<audio>`, `<video>`
	- `buffered`, `currentTime`, `duration`, `paused`, `played`, `seekable`, `volume`

As well as DOM elements, you can bind to component data properties:

```html-no-repl
<CategoryChooser bind:category='category'/>
```

If the attribute and the bound property share a name, you can use this shorthand:

```html-no-repl
<CategoryChooser bind:category/>
```

Here is a complete example of using two way bindings with a form:

```html
<form on:submit='handleSubmit( event )'>
	<input bind:value='test' type='text'>
	<button type='submit'>Store</button>
</form>

<script>
export default {
	methods: {
		handleSubmit(event) {
			// prevent the page from reloading
			event.preventDefault();

			var value = this.get('test');
			console.log('value', value);
		}
	}
};
</script>
```

> Do not confuse the `<Widget bind:foo/>` syntax (two-way binding) with the `<Widget :foo/>` syntax (one-way binding)
