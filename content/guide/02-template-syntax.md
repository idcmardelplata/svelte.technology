---
title: Template syntax
---

En lugar de reinventar la rueda, las plantillas de Svelte se basan en fundamentos que han resistido la pruebe del tiempo: HTML, CSS y JavaScript. Hay muy pocas cosas extra para aprender.


### Etiquetas

Las etiquetas le permiten vincular datos a su plantilla. Siempre que sus datos cambien (por ejemplo después de `component.set(...)`), el DOM se actualizara automáticamente. Puede usar cualquier expresion de JavaScript en sus plantillas, y tambien se actualizara automáticamente:

```html
<p>{{a}} + {{b}} = {{a + b}}</p>
```

```hidden-data
{
	"a": 1,
	"b": 2
}
```

Tambien puede usar etiquetas en los atributos:

```html
<h1 style='color: {{color}};'>{{color}}</h1>
<p hidden='{{hideParagraph}}'>You can hide this paragraph.</p>
```

```hidden-data
{
	"color": "steelblue",
	"hideParagraph": false
}
```
[Atributos boleanos](https://www.w3.org/TR/html5/infrastructure.html#sec-boolean-attributes) como `hidden` se omitirán si la expresión de la etiqueta se evalúa como falsa.

> Mientras que las etiquetas están delimitadas usando `{{` y `}}`, Svelte no usa la sintaxis de [Mustache](https://mustache.github.io/). Las etiquetas son solo expresiones de JavaScript.


### Triples

Las etiquetas ordinarias representan texto plano. Si necesita que su expresión sea interpretada como HTML, envuélvala en llaves triples, `{{{` y `}}}`.

```html
<p>This HTML: {{html}}</p>
<p>Renders as: {{{html}}}</p>
```

```hidden-data
{
	"html": "Some <b>bold</b> text."
}
```

Al igual que con las etiquetas, es posible usar cualquier expresión de JavaScript con triples, y actualizara automáticamente el documento cuando cambien los datos.

> Los triples *no* limpiara el html antes de representarlo! Si esta mostrando la entrada del usuario, usted es responsable de primero limpiarlo. No hacerlo habré su aplicación a todo tipo de ataques diferentes.


### Bloques if

Controle si una parte de su plantilla sera renderizada envolviéndola en un bloque if.

```html-no-repl
{{#if user.loggedIn}}
	<a href='/logout'>log out</a>
{{/if}}

{{#if !user.loggedIn}}
	<a href='/login'>log in</a>
{{/if}}
```

Puede combinar los dos bloques de arriba con `{{else}}`:

```html-no-repl
{{#if user.loggedIn}}
	<a href='/logout'>log out</a>
{{else}}
	<a href='/login'>log in</a>
{{/if}}
```

También puede usar `{{elseif ...}}`:

```html
{{#if x > 10}}
	<p>{{x}} is greater than 10</p>
{{elseif 5 > x}}
	<p>{{x}} is less than 5</p>
{{else}}
	<p>{{x}} is between 5 and 10</p>
{{/if}}
```

```hidden-data
{
	"x": 7
}
```

### Bloques Each.

Itera sobre una lista de datos:

```html
<h1>Cats of YouTube</h1>

<ul>
	{{#each cats as cat}}
		<li><a target='_blank' href='{{cat.video}}'>{{cat.name}}</a></li>
	{{/each}}
</ul>
```

```hidden-data
{
	"cats": [
		{
			"name": "Keyboard Cat",
			"video": "https://www.youtube.com/watch?v=J---aiyznGQ"
		},
		{
			"name": "Maru",
			"video": "https://www.youtube.com/watch?v=z_AbfPXTKms"
		},
		{
			"name": "Henri The Existential Cat",
			"video": "https://www.youtube.com/watch?v=OUtn3pvWmpg"
		}
	]
}
```

Puede acceder al indice del elemento actual con *expresión* as *name*, *index*:

```html
<div class='grid'>
	{{#each rows as row, y}}
		<div class='row'>
			{{#each columns as column, x}}
				<code class='cell'>
					{{x + 1}},{{y + 1}}:
					<strong>{{row[column]}}</strong>
				</code>
			{{/each}}
		</div>
	{{/each}}
</div>
```

```hidden-data
{
	"columns": [ "foo", "bar", "baz" ],
	"rows": [
		{ "foo": "a", "bar": "b", "baz": "c" },
		{ "foo": "d", "bar": "e", "baz": "f" },
		{ "foo": "g", "bar": "h", "baz": "i" }
  ]
}
```

> Por defecto, si la lista `a, b, c` se convierte en `a, c`, Svelte *eliminara* el tercer bloque y *cambiara* el segundo de `b` a `c`, en lugar de eliminar `b`. Si eso no es lo que quiere, use un [keyed each block](#keyed-each-blocks).

Ademas, si lo desea, puede realizar un nivel de desestructuración en los elementos del array directamente en el bloque each:

```html
<h1>It's the cats of YouTube again</h1>

<ul>
	{{#each cats as [name, video]}}
		<li><a target='_blank' href='{{video}}'>{{name}}</a></li>
	{{/each}}
</ul>
```

```hidden-data
{
	"cats": [
		[
			"Keyboard Cat",
			"https://www.youtube.com/watch?v=J---aiyznGQ"
		],
		[
			"Maru",
			"https://www.youtube.com/watch?v=z_AbfPXTKms"
		],
		[
			"Henri The Existential Cat",
			"https://www.youtube.com/watch?v=OUtn3pvWmpg"
		]
	]
}
```


### Await blocks

Puede representar los tres estados de una [Promesa](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) - pending, fulfilled y rejected - con un bloque `await`:

```html
{{#await promise}}
	<p>wait for it...</p>
{{then answer}}
	<p>the answer is {{answer}}!</p>
{{catch error}}
	<p>well that's odd</p>
{{/await}}

<script>
	export default {
		data() {
			return {
				promise: new Promise(fulfil => {
					setTimeout(() => fulfil(42), 3000);
				})
			};
		}
	};
</script>
```

Si la expresión en `{{#await expression}}` *no es* una promise, Svelte saltara directamente a la sección `then`.


### Directivas.

El último lugar donde la sintaxis de templates de Svelte difiere del HTML abitual: *directives*  le permite añadir instrucciones especiales para agregar [manejadores de eventos](#event-handlers), [two-way bindings](#two-way-binding), [refs](#refs) y así. Cubriremos cada uno de ellos en etapas posteriores de esta guía - Por ahora, todo lo que necesita saber es que las directivas pueden ser identificadas por el carácter `:`.

```html
<p>Count: {{count}}</p>
<button on:click='set({ count: count + 1 })'>+1</button>
```

```hidden-data
{
	"count": 0
}
```
> Técnicamente, el carácter `:` es usado para denotar atributos de namespaces en HTML. Estas *no* se trataran como directivas si se encuentran.
