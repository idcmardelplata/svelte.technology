---
title: Component API
---

Como hemos visto previamente, usted crea una instancia de un componente con la palabra clave `new`.

```js
import MyComponent from './MyComponent.html';

const component = new MyComponent({
	// `target` is the only required option – the element
	// to render the component to
	target: document.querySelector('main'),

	// `data` is optional. A component can also have
	// default data – we'll learn about that later.
	data: {
		questions: [
			'life',
			'the universe',
			'everything'
		],
		answer: 42
	}
});
```

Cada instancia de componente de Svelte tiene una pequeña cantidad de métodos que pueden utilizarse para controlarlo, ademas de cualquier [método personalizado](#custom-methods) que usted agregue.

### component.set(data)

Esto actualiza el estado del componente con los nuevos valores proporcionados y hace que el DOM se actualice. `data` debe ser un simple objeto JavaScript antiguo (POJO). Cualquier propiedad *no* incluida en `data` permanecerá como estaba.

```js
component.set({
	questions: [
		'why is the sky blue?',
		'how do planes fly?',
		'where do babies come from?'
	],
	answer: 'ask your mother'
});
```

> Si ha usando Reactive en el pasado, esto se vera muy similar a `reactive.set(...)`, excepto que en lugar de hacer `ractive.set('foo', 'bar')` siempre debe hacer `ractive.set({foo: 'bar'})`, y no puede establecer keypaths anidados directamente. También es muy similar al `setState` de React, excepto que causa actualizaciones sincrónicas, lo que significa que el DOM siempre estará en un estado predecible.


### component.get(key)

Retorna el valor actual de `key`:

```js
console.log(component.get('answer')); // 'ask your mother'
```

Esto tambien recuperara el valor de las [propiedades computadas](#computed-properties).

Tambíen puede llamar a `component.get()` sin ningún argumento para recuperar un objeto con todas las claves y valores, incluidas las propiedades computadas. Esto funciona particularmente bien con la desestructuración de ES6:

```js
const { foo, bar, baz } = component.get();
```


### component.observe(key, callback[, options])

Este método le permite responder a cambios en el estado, que es particularmente útil cuando se combina con: [lifecycle hooks](#lifecycle-hooks) y [two-way bindings](#two-way-binding).

```js
const observer = component.observe('answer', answer => {
	console.log( `the answer is ${answer}` );
});
// fires immediately with current answer:
// -> 'the answer is ask your mother'

component.set({ answer: 'google it' });
// -> 'the answer is google it'

observer.cancel(); // further changes will be ignored
```

El callback toma dos argumentos - el valor actual y el valor previo. (La primera vez que se llama, el segundo argumento sera `undefined`):

```js
thermometer.observe('temperature', (newValue, oldValue) => {
	if (oldValue === undefined) return;
	console.log(`it's getting ${newValue > oldValue ? 'warmer' : 'colder'}`);
});
```

Si usted no desea que el callback se dispare al conectar por primera vez con el observer, use `init: false`:

```js
thermometer.observe('temperature', (newValue, oldValue) => {
	console.log(`it's getting ${newValue > oldValue ? 'warmer' : 'colder'}`);
}, { init: false });
```

> Para valores *primitivos* como strings y numeros, los callbacks del observer se invocaran solamente cuando el valor cambie. Pero debido a que es posible mutar un objeto o array preservando la *igualdad referencial*, Svelte se equivocara en el lado de la precaución. En otras palabras, si haces `component.set({foo: component.get('foo')})`, y `foo` es un objeto o un array, cualquier observer de `foo` se activara.

Por defecto, los observers serán llamados *antes* que el DOM se actualice, dándole la posibilidad de realizar cualquier actualización adicional sin tener que tocar el DOM mas de lo necesario. En algunos casos - por ejemplo, si necesita calcular un elemento despúes de que el DOM se haya actualizado - use `defer: true`:

```js
function redraw() {
	canvas.width = drawingApp.get('width');
	canvas.height = drawingApp.get('height');
	updateCanvas();
}

drawingApp.observe('width', redraw, { defer: true });
drawingApp.observe('height', redraw, { defer: true });
```

Para observar la propiedades de un componente anidado, use refs:

```html-no-repl
<Widget ref:widget/>
<script>
	export default {
		oncreate() {
			this.refs.widget.observe('xxx', () => {...});
		}
	};
</script>
```


### component.on(eventName, callback)

Le permite responder a *eventos*:

```js
const listener = component.on('thingHappened', event => {
	console.log(`A thing happened: ${event.thing}`);
});

// some time later...
listener.cancel();
```


### component.fire(eventName, event)

El acompañante de `component.on(...)`:

```js
component.fire('thingHappened', {
	thing: 'this event was fired'
});
```

A primera vista `component.on(...)` y `component.fire(...)` no son particularmente útiles, pero lo serán más cuando aprendamos sobre [componentes anidados](#nested-components).

> `component.on(...)` y `component.observe(...)` lucen similares, pero tienen diferentes propósitos. Los observers son útiles para reaccionar a los datos que fluyen a través de su aplicación y cambian continuamente a lo largo del tiempo, mientras que los eventos son buenos para modelar momentos discretos como 'el usuario hizo una selección y esto es lo que selecciono'.


### component.destroy()

Elimina el componente del DOM y elimina los observers y los detectores de eventos que se crearon. Esto también dispara un evento `destroy`:

```js
component.on('destroy', () => {
	alert('goodbye!'); // please don't do this
});

component.destroy();
```


### component.options

Las opciones usadas al instanciar el componente están disponibles en `component.options`.

```html
Check the console.

<script>
	export default {
		oncreate() {
			console.log(this.options);
		}
	}
</script>
```

Esto le da acceso a opciones estándar como `target` y `data`, pero también se puede usar para acceder a cualquier otra opción personalizada que haya decidido implementar para su componente.


### component.root

En [componentes anidados](#nested-components), cada componente tiene una propiedad `root` que apunta al componente raíz de nivel superior, es decir, el instanciado con `new MyComponent({ ... })`.
