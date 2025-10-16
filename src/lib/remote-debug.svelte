<script lang="ts">
	import type { RemoteForm } from '@sveltejs/kit';
	import Minimize2 from 'lucide-svelte/icons/minimize-2';
	import Maximize2 from 'lucide-svelte/icons/maximize-2';
	import ClipboardCopy from 'lucide-svelte/icons/clipboard-copy';
	import { codeToHtml, type BundledTheme } from 'shiki';

	type Props<T = unknown> = {
		form: RemoteForm<any, T> | Omit<RemoteForm<any, T>, 'for'>;
		showIssues?: boolean;
		windowed?: boolean;
		theme?: BundledTheme;
		space?: number;
	};

	let { form, showIssues, windowed, theme = 'rose-pine', space = 2 }: Props = $props();

	let oldOutput = $state(''); // To prevent flickering when awaiting
	let dragging = $state(false);
	let minimized = $state(false);
	let windowFrame: HTMLDivElement | undefined = $state();
	let copyObject: Record<string, any> = $state({});
	let pos = $state({
		x: 100,
		y: 100
	});

	const data = $derived.by(async () => {
		let outputData: Record<string, any> = {};

		if (showIssues) {
			outputData.data = form.fields.value();
			outputData.issues = {};

			Object.keys(form.fields.value()).forEach(
				(key) => (outputData.issues[key] = form.fields[key].issues()?.map((issue) => issue.message))
			);
		} else {
			outputData = { ...form.fields.value() };
		}

		const clone = structuredClone(outputData);
		const outputHtml = await codeToHtml(JSON.stringify(clone, null, space), {
			lang: 'json',
			theme: theme
		});

		copyObject = clone;
		oldOutput = outputHtml;

		return outputHtml;
	});

	function handleDragWindow(e: MouseEvent) {
		if (!dragging || minimized) return;
		pos.x += e.movementX;
		pos.y += e.movementY;

		if (windowFrame) {
			windowFrame.style.left = `${pos.x}px`;
			windowFrame.style.top = `${pos.y}px`;
			windowFrame.style.right = `auto`;
		}
	}

	function handleMinimize() {
		if (!windowFrame) return;
		minimized = true;

		windowFrame.style.right = `20px`;
		windowFrame.style.top = `20px`;
		windowFrame.style.left = `auto`;
	}

	function handleMaximize() {
		if (!windowFrame) return;
		minimized = false;

		windowFrame.style.left = `${pos.x}px`;
		windowFrame.style.top = `${pos.y}px`;
		windowFrame.style.right = `auto`;
	}
</script>

{#snippet body()}
	{#await data}
		{#if oldOutput}
			{@html oldOutput}
		{/if}
	{:then output}
		{#if output}
			{@html output}
		{/if}
	{/await}
{/snippet}

<svelte:window on:mousemove={handleDragWindow} onmouseup={() => (dragging = false)} />

{#if windowed}
	<div bind:this={windowFrame} class="fixed z-[9999] flex flex-col">
		<div
			role="dialog"
			tabindex="0"
			class="text-muted-fg flex h-8 min-w-72 cursor-grab items-center justify-between rounded-t-md bg-black px-4 font-mono text-sm opacity-70 select-none {minimized
				? 'w-44! rounded-b-md'
				: ''}"
			onmousedown={() => (dragging = true)}
		>
			<span>Remote Debug</span>
			<div class="flex items-center gap-3">
				{#if minimized}
					<button
						class="flex cursor-pointer items-center gap-1 transition-colors hover:text-white"
						onclick={handleMaximize}
					>
						<Maximize2 class="size-3.5" />
						<span>Max</span>
					</button>
				{:else}
					<button
						class="flex cursor-pointer items-center gap-1 transition-colors hover:text-white"
						onclick={() => navigator.clipboard.writeText(JSON.stringify(copyObject, null, space))}
					>
						<ClipboardCopy class="size-3.5" />
						<span>Copy</span>
					</button>
					<button
						class="flex cursor-pointer items-center gap-1 transition-colors hover:text-white"
						onclick={handleMinimize}
					>
						<Minimize2 class="size-3.5" />
						<span>Min</span>
					</button>
				{/if}
			</div>
		</div>
		{#if !minimized}
			{@render body()}
		{/if}
	</div>
{:else}
	<div class="group">
		{@render body()}
	</div>
{/if}

<style>
	:global(.shiki) {
		padding: 1rem;
		border-bottom-left-radius: 0.5rem;
		border-bottom-right-radius: 0.5rem;
		overflow-x: auto;
		opacity: 0.9;
	}

	:global(.group > .shiki) {
		border-top-left-radius: 0.5rem;
		border-top-right-radius: 0.5rem;
		opacity: 1;
	}
</style>
