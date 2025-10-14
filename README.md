# SvelteKit Remote Debug

This component allows you to debug your [remote functions](https://svelte.dev/docs/kit/remote-functions#form) in the style of sveltekit-superforms [SuperDebug](https://www.npmjs.com/package/sveltekit-remote-debug).

With the remote functions from SvelteKit, the superforms library isn't always necessary - but still needs a visual debugger when developing.

## How to use

First you simply import the debugger in your project together with your remote function

```ts
import { RemoteDebug } from 'sveltekit-remote-debug';
import { MyRemoteFormFunction } from '$lib/remote-functions/my-remote-form-function.ts';
```

The `RemoteDebug` takes the form as a property to collect it's data

```html
<RemoteDebug form="{MyRemoteFormFunction}" />
```

Having this in your code, will show a box with your fields and it's values, such as

```json
{
	"firstname": "foo",
	"lastname": "bar"
}
```

### Options

```ts
type Props<T = unknown> = {
	showIssues?: boolean;
	windowed?: boolean;
	theme?: BundledTheme;
	space?: number;
};
```

#### showIssues

Enabling this property will also show you a list of all the issues that is generated from your validation schema. This also separates the input data in it's own `data` property.

```json
{
	"data": {
		"firstname": undefined,
		"lastname": "bar"
	},
	"issues": {
		"firstname": ["Firstname is required"],
		"lastname": undefined
	}
}
```

#### windowed

Enabling this property will instead make the debugger an absolute div which you can move around. Here you can also minimize it and copy the content (json) from the box.

#### theme

Because of the simplicity with [shiki](https://github.com/shikijs/shiki?tab=readme-ov-file) that this debugger is using for highlighting, you can also change the theme of your output.

#### space

This option takes a number that defines the tab-spacing for each row of your output.
