---
title: Scoped styles
---

Uno de los principios clave de Svelte, es que los componentes deben ser autocontenidos y reutilizables en diferentes contextos. Por esto, tiene un mecanismo para el *ambito* de su CSS, para que no choque accidentalmente con otros selectores en la pagina.

### Agregando estilos.

Su plantilla de componente puede tener una etiqueta `<style>`, como esta:

```html
<div class='foo'>
	Big red Comic Sans
</div>

<style>
	.foo {
		color: red;
		font-size: 2em;
		font-family: 'Comic Sans MS';
	}
</style>
```


### ¿Como funciona?

Abra el ejemplo de arriba en el REPL e inspeccióne el elemento para ver lo que ha sucedido - Svelte ha agregado un atributo `svelte-[uniqueid]` al elemento, y transformo el selector de CSS en consecuencia. Como ningún otro elemento en la página puede compartir ese selector, cualquier otra cosa en la página que tenga `class="foo"` no se vera afectada por nuestros estilos.

Esto es mucho mas simple que lograr el mismo efecto usando el [Shadow DOM](http://caniuse.com/#search=shadow%20dom) y funciona en todas partes sin polyfills.

> Svelte agregara una etiqueta `<style>` a la pagina conteniendo sus scoped styles. La adición dinamica de estilos puede llegar a ser imposible si su sitio tiene una [Politica de seguridad de contenido](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP). Si ese es el caso puede usar estilos con ambito via [renderizado de CSS en el servidor](#rendering-css) y usando la opción del compilador  `css: false` (o `--no-css` en el CLI).


### Reglas en cascada.

Por defecto, el mecanismo en cascada normal todavia se aplica - cualquier estilo global `.foo` todavia se aplicaria, y si su plantilla tiene [componentes anidados](#nested-components) con elementos `class="foo"`, hellos heredarian nuestros estilos.

Si la opción `cascade: false` se pasa al compilador, los estilos *solo* se aplicaran al componente actual, a menos que opte por entrar en la cascada con el midificador `:global(...)` .

<!-- TODO `cascade: false` in the REPL -->

```html-no-repl
<div>
	<Widget/>
</div>

<style>
	p {
		/* this block will be disregarded, since
		   there are no <p> elements here */
		color: red;
	}

	div :global(p) {
		/* this block will be applied to any <p> elements
		   inside the <div>, i.e. in <Widget> */
		font-weight: bold;
	}
</style>

<script>
	import Widget from './Widget.html';

	export default {
		components: { Widget }
	};
</script>
```

Se recomienda el comportamiento `cascade: false` y se activara de manera predeterminada en futuras versiones de Svelte.

> Los ambitos de estilos no son dinamicos - se comparten entre todas las instancias de un componente. En otras palabras, no puede usar `{{tags}}` dentro de su CSS.

### Eliminación de estilos no utilizados

Si esta utilizando la opción recomendada `cascade: false`, Svelte identificara y eliminara cualquier estilo que no este siendo utilizado en su aplicación. Tambien emitira una advertencia para que pueda eliminarlos del codigo fuente.

### Selectores especiales.

Si tiene un [ref](#refs) en un elemento, puede utilizarlo como un selector de CSS. Un elemento con `ref:foo` se puede estilizar con `ref:foo { color: whatever; }`. El selector `ref:*` tiene la misma especificidad que un selector de clase o atributo.
