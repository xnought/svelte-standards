<p align="center">
  <img src="https://user-images.githubusercontent.com/65095341/202578753-211e0bb7-81dd-462f-a4d9-a0e188e5d127.png" width="100%">
  </p>

[Svelte](https://svelte.dev/) is awesome due to its simplicity, but it also can be abused such that it becomes unreadable. This repo houses some design standards to abide by for simpler and readable svelte.

1. [layout](#layout)
2. [guidelines](#guidelines)

# <a name="layout" href="#layout">#</a> <i>layout</i>

## <a name="tag-order" href="#tag-order">#</a> <i>tag order</i>

Lets just go by the order of `script`, `html` stuff, then `style` at the end of the file. This is mostly arbitrary, but being consistent will be nice.

For example

```html
<script>
	// js
</script>

<!-- html -->

<style>
	/* css */
</style>
```

## <a name="script-order" href="#script-order">#</a> <i>script order</i>

Within the `<script>` tag, roughly organize the code in the following order:

1.  Imports
    1.  CSS (if you have it)
    2.  Svelte components
    3.  Code (ts or js)
    4.  Type only (ts)
2.  constants (`const` or `export const`)
3.  props (`export let`)
4.  mutable variables (`let`)
5.  reactive statements (`$:`)
6.  function calls (`onMount`, `onDestroy`, etc.)
7.  function definitions

For example

```html
<script>
	// 1.1 CSS imports
	import "app.css";

	// 1.2 Svelte components
	import Scatterplot from "../components/Scatterplot.svelte";
	import Slider from "../components/Slider.svelte";

	// 1.3 Code imports
	import { onMount } from "svelte";

	// 2. constants
	const name = "Donny";

	// 3. props
	export let height = 73;

	// 4. mutable variables
	let age = 20;

	// 5. reactive statements
	$: dogYears = humanToDogYears(age);

	// 6. function calls
	onMount(() => {
		age = birthday(age);
		printPerson();
	});

	// 7. function definitions
	function printPerson() {
		console.log(name, age);
	}
	function birthday(age) {
		const newAge = age + 1;
		return newAge;
	}
	function humanToDogYears(age) {
		return age * 7;
	}
</script>
```

# <a name="guidelines" href="#guidelines">#</a> <i>guidelines</i>

## <a name="function-scope" href="#function-scope">#</a> <i>function scope</i>

In general, svelte projects get confusing when the functions start to take parameters and mutate global variables.

If possible, try not to mutate global variables within function definitions.

> **Warning** Try to avoid this below if you can

```html
<script>
	let age = 20;

	// mutates global variable age
	function birthday() {
		age++;
	}

	// when birthday needs to be called (could be anywhere)
	birthday();
</script>
```

> **Note** Try this below instead

```html
<script>
	let age = 20;

	// takes in age as a parameter and returns the new age
	// then, elsewhere redefine age as the new age
	function birthday(age) {
		const newAge = age + 1;
		return newAge;
	}

	// when birthday needs to be called (could be anywhere)
	const newAge = birthday(age);
	age = newAge; // and then do reassignment to the global
</script>
```

then you get the benefit of

1. (organization) being able to put those functions in modules and import them
2. (simplicity) Mutating outside variables and also taking in parameters, and also returning stuff is confusing

## <a name="reactive-$" href="#reactive-$">#</a> <i>reactive $:{ }</i>

In my experience, the `$: {}` parts always lead to the most confusion to what is actually happening.

When using `$: {}` it can be hard to know

1. what variable should it be reacting to
2. what is the purpose of this reactive statement

To clear this up, lets try to use the `$:` statements only and do more complex operations within a function definition instead of a `$: {}` statement.

For example,

> **Warning** Try to avoid this below if you can

```html
<script>
	let dogAge, chickenAge, cowAge;
	$: {
		cowAge = age * 3.64;
		chickenAge = age * 5.33;
		dogAge = age * 7;
	}
</script>
```

> **Note** Try this below instead

```html
<script>
	$: dogAge = humanToDogYears(age);
	$: chickenAge = humanToChickenYears(age);
	$: cowAge = humanToCowYears(age);

	function humanToDogYears(age) {
		return age * 7;
	}
	function humanToChickenYears(age) {
		return age * 5.33;
	}
	function humanToCowYears(age) {
		return age * 3.64;
	}
</script>
```

The parameters of the function should always be why the `$:` should react / be reran. Now it should be clear what the reactive statement is doing (from function name) and why its reacting (from function parameters).

This also allows you to ignore variables that should not rerun the `$:`

In the example below, I only redraw points when the points array changes or when canvasEl changes. I don't react to point `size` changes. If this were all done in a `$: {}` statement, then it would rerun every time `size` changes.

```html
<script>
	export let points = [
		{ x: 1, y: 2 },
		{ x: 2, y: 3 },
		{ x: 3, y: 4 },
	];
	export let size = 3;

	let canvasEl;

	$: redrawPoints(canvasEl, points);

	function redrawPoints(canvasEl, points) {
		if (canvasEl) {
			const ctx = canvasEl.getContext("2d");
			ctx.clearRect(0, 0, canvasEl.width, canvasEl.height);

			points.forEach((point) => {
				ctx.fillRect(point.x, point.y, size, size);
			});
		}
	}
</script>

<canvas width="{100}" height="{100}" bind:this="{canvasEl}"></canvas>
```

But now you see! You get control over reactive statements and readability by not using anonymous statements `$: {}` and instead using function definitions for english readability and control over what variables should trigger the function.

### TODO GUIDELINES

1. ~~`$:` should not be anonymous~~
2. ~~decide if function definitions should reference global variables or not~~
3. better guidelines for `createEventDispatcher` and `dispatch` (especially for ts)
4. when to put function definitions in other files
5. How to define types
6. Optimal folder structure (nested? flat? import pattern?)
7. bind guidelines
8. When to use global stores vs context vs props
