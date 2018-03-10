---
title: Comportamientos
---

Ademas de los ambitos de estilos y de los templates, los componentes encapsulan *comportamientos*. Para ello, necesitamos agregar una etiqueta `<script>` y exportar un objeto:

```html
<div>
	<!-- template goes here -->
</div>

<script>
	export default {
		// behaviours go here
	};
</script>
```


### Datos predeterminados

A menudo tiene sentido que un componente tenga datos predeterminados. Esto debe expresarse como una función que devuelva un objeto de JavaScript plano:

```html
<p>Count: {{count}}</p>
<button on:click='set({ count: count + 1 })'>+1</button>

<script>
	export default {
		data() {
			return {
				count: 0
			};
		}
	};
</script>
```

Las data suministrada en la instanciación tiene prioridad sobre los valores predeterminados. En otras palabras, si creamos una instancia del componente anterior como tal...  

```js
const counter = new Counter({
	data: {
		count: 99
	}
});
```

...entonces `{{count}}`, o `counter.get('count')`, seria inicialmente 99 en lugar de 0.

> El ejemplo anterior, al igual que muchos ejemplos a continuación, usan la sintaxis de ES2015 - es decir `data () {...}` en lugar de `data: function {...}`. Mientras que Svelte generara codigo ES5 que se ejecute en todas partes, no convertira su codigo ES1015 en ES5 - por lo que si usa ES2015 y necesita soportar navegadores mas antiguos, necesitara un paso de transpilación adicional en su proceso de construccion del sitio, usando [Babel](https://babeljs.io) o [Bublé](https://buble.surge.sh)

### Propiedades calculadas

A menudo, su programa usara valores que dependen de otros valores - por ejemplo, es posible que tenga una lista filtrada, que dependa de ambos, la lista *y* el filtro. Normalmente en JavaScript tendrias que agregar logica para actualizar la propiedad dependiente cuando *cualquiera* de las dependencias cambien. Esta es una fuente frecuente de errores, y empeora a medida que su aplicación crece.

Svelte le permite expresar estas dependencias en propiedades calculadas, que seran recalculadas cuando cambien esas dependencias:

```html
<p>
	The time is
	<strong>{{hours}}:{{minutes}}:{{seconds}}</strong>
</p>

<script>
	export default {
		data() {
			return {
				time: new Date()
			};
		},

		computed: {
			hours: time => time.getHours(),
			minutes: time => time.getMinutes(),
			seconds: time => time.getSeconds()
		}
	};
</script>
```

Tenga en cuenta que todo lo que tenemos que hacer para decirle a Svelte que `hours`, `minutes` y `seconds` son dependientes de `time` es simplemente incluirlo como parametro de la función. No hay un costoso seguimiento de dependencias involucrado - puesto que el grafico de dependencias es resulto en tiempo de compilación.

> `computed` debe ser un objeto literal, y las propiedades deben ser expresiones de función o expresiones de funciones de flecha. Cualquier funcion externa utilizada en computed debera ser envuelta _aqui_:

```js
import externalFunc from '_external_file';
export default {
  computed: {
    externalFunc: (dep1, dep2) => externalFunc(dep1, dep2);
  }
}
```

Las propiedades calculadas pueden, por supuesto devolver funciones. Por ejemplo, podriamos generar dinamicamente una función de filtro para una lista de items:

```html
<input bind:value=search>

{{#each words.filter(predicate) as word}}
	<p><strong>{{word.slice(0, search.length)}}</strong>{{word.slice(search.length)}}</p>
{{else}}
	<p>no matches!</p>
{{/each}}

<script>
	export default {
		computed: {
			predicate: search => {
				search = search.toLowerCase();
				return word => word.startsWith(search);
			}
		}
	};
</script>
```

```hidden-data
{
	"search": "",
	"items": [
		"about",
		"above",
		"abuse",
		"actor",
		"acute",
		"admit",
		"adopt",
		"adult",
		"after",
		"again",
		"agent",
		"agree",
		"ahead",
		"alarm",
		"album",
		"alert",
		"alike",
		"alive",
		"allow",
		"alone",
		"along",
		"alter",
		"among",
		"anger",
		"Angle",
		"angry",
		"apart",
		"apple",
		"apply",
		"arena",
		"argue",
		"arise",
		"array",
		"aside",
		"asset",
		"audio",
		"audit",
		"avoid",
		"award",
		"aware",
		"badly",
		"baker",
		"bases",
		"basic",
		"basis",
		"beach",
		"began",
		"begin",
		"begun",
		"being",
		"below",
		"bench",
		"billy",
		"birth",
		"black",
		"blame",
		"blind",
		"block",
		"blood",
		"board",
		"boost",
		"booth",
		"bound",
		"brain",
		"brand",
		"bread",
		"break",
		"breed",
		"brief",
		"bring",
		"broad",
		"broke",
		"brown",
		"build",
		"built",
		"buyer",
		"cable",
		"calif",
		"carry",
		"catch",
		"cause",
		"chain",
		"chair",
		"chart",
		"chase",
		"cheap",
		"check",
		"chest",
		"chief",
		"child",
		"china",
		"chose",
		"civil",
		"claim",
		"class",
		"clean",
		"clear",
		"click",
		"clock",
		"close",
		"coach",
		"coast",
		"could",
		"count",
		"court",
		"cover",
		"craft",
		"crash",
		"cream",
		"crime",
		"cross",
		"crowd",
		"crown",
		"curve",
		"cycle",
		"daily",
		"dance",
		"dated",
		"dealt",
		"death",
		"debut",
		"delay",
		"depth",
		"doing",
		"doubt",
		"dozen",
		"draft",
		"drama",
		"drawn",
		"dream",
		"dress",
		"drill",
		"drink",
		"drive",
		"drove",
		"dying",
		"eager",
		"early",
		"earth",
		"eight",
		"elite",
		"empty",
		"enemy",
		"enjoy",
		"enter",
		"entry",
		"equal",
		"error",
		"event",
		"every",
		"exact",
		"exist",
		"extra",
		"faith",
		"false",
		"fault",
		"fiber",
		"field",
		"fifth",
		"fifty",
		"fight",
		"final",
		"first",
		"fixed",
		"flash",
		"fleet",
		"floor",
		"fluid",
		"focus",
		"force",
		"forth",
		"forty",
		"forum",
		"found",
		"frame",
		"frank",
		"fraud",
		"fresh",
		"front",
		"fruit",
		"fully",
		"funny",
		"giant",
		"given",
		"glass",
		"globe",
		"going",
		"grace",
		"grade",
		"grand",
		"grant",
		"grass",
		"great",
		"green",
		"gross",
		"group",
		"grown",
		"guard",
		"guess",
		"guest",
		"guide",
		"happy",
		"harry",
		"heart",
		"heavy",
		"hence",
		"henry",
		"horse",
		"hotel",
		"house",
		"human",
		"ideal",
		"image",
		"index",
		"inner",
		"input",
		"issue",
		"japan",
		"jimmy",
		"joint",
		"jones",
		"judge",
		"known",
		"label",
		"large",
		"laser",
		"later",
		"laugh",
		"layer",
		"learn",
		"lease",
		"least",
		"leave",
		"legal",
		"level",
		"lewis",
		"light",
		"limit",
		"links",
		"lives",
		"local",
		"logic",
		"loose",
		"lower",
		"lucky",
		"lunch",
		"lying",
		"magic",
		"major",
		"maker",
		"march",
		"maria",
		"match",
		"maybe",
		"mayor",
		"meant",
		"media",
		"metal",
		"might",
		"minor",
		"minus",
		"mixed",
		"model",
		"money",
		"month",
		"moral",
		"motor",
		"mount",
		"mouse",
		"mouth",
		"movie",
		"music",
		"needs",
		"never",
		"newly",
		"night",
		"noise",
		"north",
		"noted",
		"novel",
		"nurse",
		"occur",
		"ocean",
		"offer",
		"often",
		"order",
		"other",
		"ought",
		"paint",
		"panel",
		"paper",
		"party",
		"peace",
		"peter",
		"phase",
		"phone",
		"photo",
		"piece",
		"pilot",
		"pitch",
		"place",
		"plain",
		"plane",
		"plant",
		"plate",
		"point",
		"pound",
		"power",
		"press",
		"price",
		"pride",
		"prime",
		"print",
		"prior",
		"prize",
		"proof",
		"proud",
		"prove",
		"queen",
		"quick",
		"quiet",
		"quite",
		"radio",
		"raise",
		"range",
		"rapid",
		"ratio",
		"reach",
		"ready",
		"refer",
		"right",
		"rival",
		"river",
		"robin",
		"roger",
		"roman",
		"rough",
		"round",
		"route",
		"royal",
		"rural",
		"scale",
		"scene",
		"scope",
		"score",
		"sense",
		"serve",
		"seven",
		"shall",
		"shape",
		"share",
		"sharp",
		"sheet",
		"shelf",
		"shell",
		"shift",
		"shirt",
		"shock",
		"shoot",
		"short",
		"shown",
		"sight",
		"since",
		"sixth",
		"sixty",
		"sized",
		"skill",
		"sleep",
		"slide",
		"small",
		"smart",
		"smile",
		"smith",
		"smoke",
		"solid",
		"solve",
		"sorry",
		"sound",
		"south",
		"space",
		"spare",
		"speak",
		"speed",
		"spend",
		"spent",
		"split",
		"spoke",
		"sport",
		"staff",
		"stage",
		"stake",
		"stand",
		"start",
		"state",
		"steam",
		"steel",
		"stick",
		"still",
		"stock",
		"stone",
		"stood",
		"store",
		"storm",
		"story",
		"strip",
		"stuck",
		"study",
		"stuff",
		"style",
		"sugar",
		"suite",
		"super",
		"sweet",
		"table",
		"taken",
		"taste",
		"taxes",
		"teach",
		"teeth",
		"terry",
		"texas",
		"thank",
		"theft",
		"their",
		"theme",
		"there",
		"these",
		"thick",
		"thing",
		"think",
		"third",
		"those",
		"three",
		"threw",
		"throw",
		"tight",
		"times",
		"tired",
		"title",
		"today",
		"topic",
		"total",
		"touch",
		"tough",
		"tower",
		"track",
		"trade",
		"train",
		"treat",
		"trend",
		"trial",
		"tried",
		"tries",
		"truck",
		"truly",
		"trust",
		"truth",
		"twice",
		"under",
		"undue",
		"union",
		"unity",
		"until",
		"upper",
		"upset",
		"urban",
		"usage",
		"usual",
		"valid",
		"value",
		"video",
		"virus",
		"visit",
		"vital",
		"voice",
		"waste",
		"watch",
		"water",
		"wheel",
		"where",
		"which",
		"while",
		"white",
		"whole",
		"whose",
		"woman",
		"women",
		"world",
		"worry",
		"worse",
		"worst",
		"worth",
		"would",
		"wound",
		"write",
		"wrong",
		"wrote",
		"yield",
		"young",
		"youth"
	]
}
```


### Lifecycle hooks

There are two 'hooks' provided by Svelte for adding control logic – `oncreate` and `ondestroy`:

```html
<p>
	The time is
	<strong>{{hours}}:{{minutes}}:{{seconds}}</strong>
</p>

<script>
	export default {
		oncreate() {
			this.interval = setInterval(() => {
				this.set({ time: new Date() });
			}, 1000);
		},

		ondestroy() {
			clearInterval(this.interval);
		},

		data() {
			return {
				time: new Date()
			};
		},

		computed: {
			hours: time => time.getHours(),
			minutes: time => time.getMinutes(),
			seconds: time => time.getSeconds()
		}
	};
</script>
```


### Helpers

Helpers are simple functions that are used in your template. In the example above, we want to ensure that `minutes` and `seconds` are preceded with a `0` if they only have one digit, so we add a `leftPad` helper:

```html
<p>
	The time is
	<strong>{{hours}}:{{leftPad(minutes, 2, '0')}}:{{leftPad(seconds, 2, '0')}}</strong>
</p>

<script>
	import leftPad from 'left-pad';

	export default {
		helpers: {
			leftPad
		},

		oncreate() {
			this.interval = setInterval(() => {
				this.set({ time: new Date() });
			}, 1000);
		},

		ondestroy() {
			clearInterval(this.interval);
		},

		data() {
			return {
				time: new Date()
			};
		},

		computed: {
			hours: time => time.getHours(),
			minutes: time => time.getMinutes(),
			seconds: time => time.getSeconds()
		}
	};
</script>
```

Of course, you could use `leftPad` inside the computed properties instead of in the template. There's no hard and fast rule about when you should use expressions with helpers versus when you should use computed properties – do whatever makes your component easier for the next developer to understand.

> Helper functions should be *pure* – in other words, they should not have side-effects, and their returned value should only depend on their arguments.


### Custom methods

In addition to the [built-in methods](#component-api), you can add methods of your own:

```html
<script>
	export default {
		methods: {
			say(message) {
				alert(message); // again, please don't do this
			}
		}
	};
</script>
```

These become part of the component's API:

```js
import MyComponent from './MyComponent.html';

var component = new MyComponent({
	target: document.querySelector('main')
});

component.say('👋');
```

Methods (whether built-in or custom) can also be called inside [event handlers](#event-handlers):

```html
<button on:click='say("hello")'>say hello!</button>
```


### Custom event handlers

Soon, we'll learn about [event handlers](#event-handlers) – if you want, skip ahead to that section first then come back here!

Most of the time you can make do with the standard DOM events (the sort you'd add via `element.addEventListener`, such as `click`) but sometimes you might need custom events to handle gestures, for example.

Custom events are just functions that take a node and a callback as their argument, and return an object with a `teardown` method that gets called when the element is removed from the page:

```html
<button on:longpress='set({ done: true })'>click and hold</button>

{{#if done}}
	<p>clicked and held</p>
{{/if}}

<script>
	export default {
		events: {
			longpress(node, callback) {
				function onmousedown(event) {
					const timeout = setTimeout(() => callback( event ), 1000);

					function cancel() {
						clearTimeout(timeout);
						node.removeEventListener('mouseup', cancel, false);
					}

					node.addEventListener('mouseup', cancel, false);
				}

				node.addEventListener('mousedown', onmousedown, false);

				return {
					teardown() {
						node.removeEventListener('mousedown', onmousedown, false);
					}
				};
			}
		}
	};
</script>
```


### Namespaces

Components are assumed to be in the HTML namespace. If your component is designed to be used inside an `<svg>` element, you need to specify the namespace:

```html
<svg viewBox='0 0 1000 1000' style='width: 100%; height: 100%;'>
	<SmileyFace x='70' y='280' size='100' fill='#f4d9c7'/>
	<SmileyFace x='800' y='250' size='150' fill='#40250f'/>
	<SmileyFace x='150' y='700' size='110' fill='#d2aa7a'/>
	<SmileyFace x='875' y='730' size='130' fill='#824e2e'/>
	<SmileyFace x='450' y='500' size='240' fill='#d2b198'/>
</svg>

<script>
	import SmileyFace from './SmileyFace.html';

	export default {
		components: { SmileyFace }
	};
</script>
```

```html-nested-SmileyFace
<!-- CC-BY-SA — https://commons.wikimedia.org/wiki/File:718smiley.svg -->
<g transform='translate({{x}},{{y}}) scale({{size / 366.5}})'>
	<circle r="366.5"/>
	<circle r="336.5" fill="{{fill}}"/>
	<path d="m-41.5 298.5c-121-21-194-115-212-233v-8l-25-1-1-18h481c6 13 10 27 13 41 13 94-38 146-114 193-45 23-93 29-142 26z"/>
	<path d="m5.5 280.5c52-6 98-28 138-62 28-25 46-56 51-87 4-20 1-57-5-70l-423-1c-2 56 39 118 74 157 31 34 72 54 116 63 11 2 38 2 49 0z" fill="#871945"/>
	<path d="m-290.5 -24.5c-13-26-13-57-9-85 6-27 18-52 35-68 21-20 50-23 77-18 15 4 28 12 39 23 18 17 30 40 36 67 4 20 4 41 0 60l-6 21z"/>
	<path d="m-132.5 -43.5c5-6 6-40 2-58-3-16-4-16-10-10-14 14-38 14-52 0-15-18-12-41 6-55 3-3 5-5 5-6-1-4-22-8-34-7-42 4-57.6 40-66.2 77-3 17-1 53 4 59h145.2z" fill="#fff"/>
	<path d="m11.5 -23.5c-2-3-6-20-7-29-5-28-1-57 11-83 15-30 41-52 72-60 29-7 57 0 82 15 26 17 45 49 50 82 2 12 2 33 0 45-1 10-5 26-8 30z"/>
	<path d="m198.5 -42.5c4-5 5-34 4-50-2-14-6-24-8-24-1 0-3 2-6 5-17 17-47 13-58-9-7-16-4-31 8-43 4-4 7-8 7-9 0 0-4-2-8-3-51-17-105 20-115 80-3 15 0 43 3 53z" fill="#fff"/>
	<path d="m137.5 223.5s-46 40-105 53c-66 15-114-7-114-7s14-76 93-95c76-18 126 49 126 49z" fill="#f9bedd"/>
</g>

<script>
	export default {
		// you can either use the shorthand 'svg', or the full
		// namespace: 'http://www.w3.org/2000/svg'. (I know
		// which one I prefer.)
		namespace: 'svg',

		data() {
			return {
				x: 100,
				y: 100,
				size: 100
			};
		}
	};
</script>
```
