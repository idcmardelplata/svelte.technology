---
title: Propiedades estaticas
---


### Setup

En algunas situaciónes, es posible que desee agregar propiedades estaticas al constructor de su componente. Para ello, se utiliza la propiedad `setup`:

```html
<p>check the console!</p>

<script>
	export default {
		setup(MyComponent) {
			// someone importing this component will be able
			// to access any properties or methods defined here:
			//
			//   import MyComponent from './MyComponent.html';
			//   console.log(MyComponent.ANSWER); // 42
			MyComponent.ANSWER = 42;
		},

		oncreate() {
			console.log('the answer is', this.constructor.ANSWER);
			console.dir(this.constructor);
		}
	};
</script>
```
