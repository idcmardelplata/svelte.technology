---
title: Introducción
---

### Que es Svelte?

Si alguna vez has creado una aplicación en JavaScript, las posibilidades que habrás encontrado son - o al menos de las que  habrás oído - frameworks como React, Angular, Vue y Reactive. Como Svelte, todas esas herramientas comparten el objetivo de facilitar la creación de interfaces de usuario interactivas.

Pero Svelte tiene una diferencia crucial: en lugar de interpretar el código de su aplicación en *tiempo de ejecución*, su  aplicación se convierte en el JavaScript ideal en *tiempo de compilación*. Esto significa que no paga el costo de performance de las abstracciones de los frameworks, ni tampoco incurre en una penalización cuando su aplicación se carga por primera vez.

Y al no existir la sobrecarga, se puede adoptar fácilmente Svelte de forma incremental en una aplicación ya existente, o puede enviar widgets como paquetes independientes que funcionaran en cualquier lugar.

[Lea la publicación introductoria del blog](/blog/frameworks-without-the-framework) para aprender mas sobre los objetivos y la filosofía de Svelte.


### Comprendiendo los componentes de Svelte

En Svelte, una aplicación es compuesta por uno o mas *componentes*. Un componente es un bloque de código autocontenido y reutilizable que encapsula el marcado, los estilos y el comportamiento que pertenece a dicho componente todo junto.

Al igual que Reactive y Vue, Svelte promueve el concepto de *single-file components*: donde un componente es solo un archivo `.html`. Aquí hay un ejemplo simple:

```html
<!-- App.html -->
<h1>Hello {{name}}!</h1>
```

```hidden-data
{
	"name": "world"
}
```
Svelte convierte esto en un modulo de JavaScript que puede importar a su aplicación:

```js
// main.js
import App from './App.html';

const app = new App({
	target: document.querySelector('main'),
	data: { name: 'world' }
});

// change the data associated with the template
app.set({ name: 'everybody' });

// detach the component and clean everything up
app.destroy();
```

Felicitaciones, acaba de aprender acerca de la mitad de la API de Svelte!


### Comenzando

Normalmente, esta es la parte donde las instrucciones podrían indicarle que agregue el framework a su pagina usando la etiqueta `<script>`. Pero debido a que Svelte se ejecuta en tiempo de compilación, funciona un poco diferente.

La mejor manera de utilizar Svelte es integrándolo en su sistema de compilación - hay complementos para Rollup, Browserify, Gulp y otros, y muchos mas en camino. Vea [aquí](https://github.com/sveltejs/svelte/#svelte) para una lista actualizada.

> Usted necesitara tener [Node.js](https://nodejs.org/en/) instalado, y estar familiarizado con la linea de comandos.

#### Comenzar a usar una plantilla de proyecto

Yendo a [REPL](/repl) y presionando el botón *download* en cualquiera de los ejemplos usted obtendrá un archivo .zip conteniendo todo lo que necesita para poder ejecutar el ejemplo de forma local. Simplemente descomprimalo, `cd` al directorio, y ejecute `npm install` y `npm run dev`. Vea [este post del blog](/blog/the-easiest-way-to-get-started) para mas información.

#### Comenzar usando svelte-cli 

Otra opción es utilizar [svelte-cli](https://github.com/sveltejs/svelte-cli), la interface de linea de comandos.

Primero, instale el CLI:

```bash
npm install -g svelte-cli
```

Luego, cree un directorio para el proyecto:

```bash
mkdir my-svelte-project
cd my-svelte-project
```

Dentro de `my-svelte-project`, cree un archivo llamado `HelloWorld.html` con el siguiente contenido:

```html-no-repl
<h1>Hello {{name}}</h1>
```

Compilelo:

```bash
svelte compile --format iife HelloWorld.html > HelloWorld.js
```

El `--format iife` significa 'generar una expresión de función invocada inmediatamente' - esto nos permite usar el componente como una simple etiqueta `<script>`. (Por defecto, Svelte creara un modulo de JavaScript en su lugar, que se recomienda para aplicaciones mas serias pero que requiere de pasos adicionales.)

Cree una pagina `index.html` e incluya el script que acaba de generar:

```html-no-repl
<!doctype html>
<html>
<head>
	<title>My first Svelte app</title>
</head>
<body>
	<main></main>
	<script src='HelloWorld.js'></script>
	<script>
		var app = new HelloWorld({
			target: document.querySelector('main'),
			data: {
				name: 'world'
			}
		});
	</script>
</body>
</html>
```

Finalmente, abra la pagina en su navegador - `open index.html` - y interactué con la `app` a trabes de la consola usando la API.
