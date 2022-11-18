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
	$: dogYears = age * 7;

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
</script>
```

# <a name="guidelines" href="#guidelines">#</a> <i>guidelines</i>

## <a name="function-scope" href="#function-scope">#</a> <i>function scope</i>

In general, svelte projects get confusing when the functions start to take parameters and mutate global variables.

If possible, try not to mutate global variables within function definitions.

> __Warning__ Try to avoid this below if you can

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

> __Note__ Try this below instead
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

### TODO GUIDELINES

1. `$:` should not be anonymous
2. ~~decide if function definitions should reference global variables or not~~
3. better guidelines for `createEventDispatcher` and `dispatch` (especially for ts)
4. when to put function definitions in other files
5. How to define types
6. Optimal folder structure (nested? flat? import pattern?)
7. bind guidelines
8. When to use global stores vs context vs props
