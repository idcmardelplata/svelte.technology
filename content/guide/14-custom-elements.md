---
title: Elementos personalizados
---

[Custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements) son un estandar web emergente para crear elementos DOM que encapsulan estilos y comportamientos, similar a los componentes de Svelte. Ellos son parte de  la familia de especificaciones de [web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components).

> La mayoria de los navegadores necesitan [polyfills](https://www.webcomponents.org/polyfills) para elementos personalizados. Vea  [caniuse](https://caniuse.com/#feat=custom-elementsv1) para mas detalles.

Los componentes de Svelte pueden ser usados como elementos personalizados haciendo lo siguiente:

1. Declarando un nombre de etiqueta. El valor debe contener un guion  (`hello-world` en el ejemplo siguiente)
2. Especificando `customElement: true` en la configuración del compilador.


```html-no-repl
<!-- HelloWorld.html -->
<h1>Hello {{name}}!</h1>

<script>
	export default {
		tag: 'hello-world'
	};
</script>
```

Importando este archivo se registrara globalmente el elemento personalizado `<hello-world>` que acepta una propiedad `name`: 

```js
import './HelloWorld.html';
document.body.innerHTML = `<hello-world name="world"/>`;

const el = document.querySelector('hello-world');
el.name = 'everybody';
```

Vea [svelte-custom-elements.surge.sh](http://svelte-custom-elements.surge.sh/) ([codigo fuente aqui](https://github.com/sveltejs/template-custom-element)) para un ejemplo mas grande.

Los elementos personalizados compilados siguen siendo componentes Svelte de pleno derecho y se pueden usar como tales:

```js
el.get('name') === el.name; // true
el.set({ name: 'folks' }); // equivalent to el.name = 'folks'
```

Una diferencia crucial es que los estilos estan *completamente encapsulados* - mientras que Svelte evitara que los estilos de los componentes se filtren *fuera*, los elementos personalizados utilizan [shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Shadow_DOM) que tambien evita fugas en los estilos.

### Usando `<slot>`

Los elementos personalizados pueden usar [slots](#composing-with-slot-) para colocar elementos secundarios, al igual que los componentes regulares de Svelte.

### Disparando eventos

Puede enviar eventos dentro de elementos personalizados para pasar datos:

```js
// inside a component method
const event = new CustomEvent('message', {
	detail: 'Hello parent!',
	bubbles: true,
	cancelable: true,
	composed: true // makes the event jump shadow DOM boundary
});

this.dispatchEvent(event);
```

Otras partes de la aplicación pueden escuchar estos eventos con `addEventListener`:

```js
const el = document.querySelector('hello-world');
el.addEventListener('message', event => {
	alert(event.detail);
});
```

> Tenga en cuenta el atributo `composed: true` del evento personalizado. Permite que el evento DOM personalizado atraviese el limite del shadow DOM y entre en el arbol del DOM principal.

### Observando propiedades

Svelte determinara, de la plantilla y los valores `computados`, que propiedades tiene el elemento personalizado - por ejemplo, `name` en nuestro `<hello-world>` de ejemplo. Puede especificar esta lista de propiedades manualmente, por ejemplo para restringir que propiedades son 'visibles' para el resto de su aplicación:

```js
export default {
	tag: 'my-thing',
	props: ['foo', 'bar']
};
```

### Optiones del compilador

Earlier, we saw the use of `customElement: true` to instruct the Svelte compiler to generate a custom element using the `tag` and (optional) `props` declared inside the component file.

Alternatively, you can pass `tag` and `props` direct to the compiler:

```js
const { code } = svelte.compile(source, {
	customElement: {
		tag: 'overridden-tag-name',
		props: ['yar', 'boo']
	}
});
```

These options will override the component's own settings, if any.

### Transpiling

* Custom elements use ES2015 classes (`MyThing extends HTMLElement`). Make sure you don't transpile the custom element code to ES5, and use a ES2015-aware minifier such as [uglify-es](https://www.npmjs.com/package/uglify-es).

* If you do need ES5 support, make sure to use `Reflect.construct` aware transpiler plugin such as [babel-plugin-transform-builtin-classes](https://github.com/WebReflection/babel-plugin-transform-builtin-classes) and a polyfill like [custom-elements-es5-adapterjs](https://github.com/webcomponents/webcomponentsjs#custom-elements-es5-adapterjs).
