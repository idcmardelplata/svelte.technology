---
title: Componentes especiales
---

Svelte incluye un puñado de componentes incorporados con un comportamiento especial.


### Etiquetas <:Self>

A veces, un componente necesita incrustrarse recursivamente - por ejemplo si usted tiene una estructura de datos similar a un arbol. En Svelte, esto se logra con la etiqueta `<:Self>`:

```html
{{#if countdown > 0}}
	<p>{{countdown}}</p>
	<:Self countdown='{{countdown - 1}}'/>
{{else}}
	<p>liftoff!</p>
{{/if}}
```

```hidden-data
{
	"countdown": 5
}
```


### Etiqueta <:Component>

Si no sabe que tipo de componente se renderizara hasta que la aplicación se ejecute - en otras palabras, es manejado por el estado - puede utilizar `<:Component>`:

```html
<input type=checkbox bind:checked=foo> foo
<:Component {foo ? Red : Blue} name='thing'/>

<script>
	import Red from './Red.html';
	import Blue from './Blue.html';

	export default {
		data() {
			return { Red, Blue }
		}
	};
</script>
```

```html-nested-Red
<p style='color: red'>Red {{name}}</p>
```

```html-nested-Blue
<p style='color: blue'>Blue {{name}}</p>
```

> Tenga en cuenta que `Red` y `Blue` son elementos en `data`, *no* en `components`, a diferencia de que si estuvieramos haciendo `<Red>` o `<Blue>`.

La expresión dentro de `{...}` puede ser cualquier expresión valida de JavaScript. Por ejemplo, podria ser una [propiedad calculada](#computed-properties): 

```html
<label><input type=radio bind:group=size value='small'> small</label>
<label><input type=radio bind:group=size value='medium'> medium</label>
<label><input type=radio bind:group=size value='large'> large</label>

<:Component {Size}/>

<script>
	import Small from './Small.html';
	import Medium from './Medium.html';
	import Large from './Large.html';

	export default {
		computed: {
			Size: size => {
				if (size === 'small') return Small;
				if (size === 'medium') return Medium;
				return Large;
			}
		}
	};
</script>
```

```html-nested-Small
<p style='font-size: 12px'>small</p>
```

```html-nested-Medium
<p style='font-size: 18px'>medium</p>
```

```html-nested-Large
<p style='font-size: 32px'>LARGE</p>
```

```hidden-data
{
	"size": "medium"
}
```


### <:Window> tags

The `<:Window>` tag gives you a convenient way to declaratively add event listeners to `window`. Event listeners are automatically removed when the component is destroyed.

```html
<:Window on:keydown='set({ key: event.key, keyCode: event.keyCode })'/>

{{#if key}}
	<p><kbd>{{key === ' ' ? 'Space' : key}}</kbd> (code {{keyCode}})</p>
{{else}}
	<p>click in this window and press any key</p>
{{/if}}

<style>
	kbd {
		background-color: #eee;
		border: 2px solid #f4f4f4;
		border-right-color: #ddd;
		border-bottom-color: #ddd;
		font-size: 2em;
		margin: 0 0.5em 0 0;
		padding: 0.5em 0.8em;
		font-family: Inconsolata;
	}
</style>
```

You can also bind to certain values — so far `innerWidth`, `outerWidth`, `innerHeight`, `outerHeight`, `scrollX`, `scrollY` and `online`:

```html
<:Window bind:scrollY='y'/>

<div class='background'></div>
<p class='fixed'>user has scrolled {{y}} pixels</p>

<style>
	.background {
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		height: 9999px;
		background: linear-gradient(to bottom, #7db9e8 0%,#0a1d33 100%);
	}

	.fixed {
		position: fixed;
		top: 1em;
		left: 1em;
		color: white;
	}
</style>
```


### <:Head> tags

If you're building an application with Svelte — particularly if you're using [Sapper](https://sapper.svelte.technology) — then it's likely you'll need to add some content to the `<head>` of your page, such as adding a `<title>` element.

You can do that with the `<:Head>` tag:

```html
<:Head>
	<title>{{post.title}} • My blog</title>
</:Head>
```

When [server rendering](#server-side-rendering), the `<head>` contents can be extracted separately to the rest of the markup.
