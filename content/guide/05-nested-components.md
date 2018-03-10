---
title: Componentes anidados
---

Ademas de contener elementos (y bloques `if` y `each`), los componentes de Svelte pueden contener *otros* componentes de Svelte.

```html-no-repl
<div class='widget-container'>
	<Widget foo bar='static' baz='{{dynamic}}'/>
</div>

<script>
	import Widget from './Widget.html';

	export default {
		data() {
			return {
				dynamic: 'this can change'
			}
		},

		components: {
			Widget
		}
	};
</script>
```

El ejemplo anterior es equivalente a lo siguiente...

```js
import Widget from './Widget.html';

const widget = new Widget({
	target: document.querySelector('.widget-container'),
	data: {
		foo: true,
		bar: 'static',
		baz: 'this can change'
	}
});
```

...excepto que Svelte se asegurara que el valor de `baz` se mantenga sincronizado con el valor de `dynamic` en el componente padre, y tambien se encargara de destruir el componente hijo cuando el padre sea destruido.

En el caso donde el valor en el componente hijo tiene el mismo nombre que en el componente principal, hay una manera mas corta de escribir esto. En lugar de

```html-no-repl
<Widget foo='{{foo}}'/>
```

Puede utilizar

```html-no-repl
<Widget :foo/>
```

> Los nombres de los componentes deben de estar en mayusculas, siguiendo la convención de JavaScript ampliamente utilizada de capitalizar los nombres de los constructores. Tambien es una manera facil de distinguir los componentes de los elementos en su plantilla.

### Componiendo con `<slot>`

Un componente puede contener un elemento `<slot></slot>` que permite que el componente padre inyecte contenido:

```html
<Box>
	<h2>Hello!</h2>
	<p>This is a box. It can contain anything.</p>
</Box>

<script>
	import Box from './Box.html';

	export default {
		components: { Box }
	};
</script>
```

```html-nested-Box
<div class='box'>
	<slot><!-- content is injected here --></slot>
</div>

<style>
	.box {
		border: 2px solid black;
		padding: 0.5em;
	}
</style>
```

```hidden-data
{}
```

El elemento `<slot>` puede contener 'contenido fallback', que se usara si no se proporcionan hijos para el componente:

```html
<Box></Box>

<script>
	import Box from './Box.html';

	export default {
		components: { Box }
	};
</script>
```

```html-nested-Box
<div class='box'>
	<slot>
		<p class='fallback'>the box is empty!</p>
	</slot>
</div>

<style>
	.box {
		border: 2px solid black;
		padding: 0.5em;
	}

	.fallback {
		color: #999;
	}
</style>
```

```hidden-data
{}
```

Tambien puede tener *slots nombrados*. Cualquier elemento con un correspondiente atributo `slot` llenara estos espacios:

```html
<ContactCard>
	<span slot='name'>P. Sherman</span>
	<span slot='address'>42 Wallaby Way, Sydney</span>
</ContactCard>

<script>
	import ContactCard from './ContactCard.html';

	export default {
		components: { ContactCard }
	};
</script>
```

```html-nested-ContactCard
<div class='contact-card'>
	<h2><slot name='name'></slot></h2>
	<slot name='address'>Unknown address</slot>
	<br>
	<slot name='email'>Unknown email</slot>
</div>

<style>
	.contact-card {
		border: 2px solid black;
		padding: 0.5em;
	}
</style>
```

```hidden-data
{}
```
