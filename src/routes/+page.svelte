<script lang="ts">
	import { cn } from '$lib/utils';
	import { tick } from 'svelte';
	import { SvelteMap, SvelteSet } from 'svelte/reactivity';

	type Node = {
		value: string;
	};
	let nodes = new SvelteMap<string, Node>();
	nodes.set('A1', { value: 'Date' });
	nodes.set('B1', { value: 'Amount' });
	nodes.set('A2', { value: '2024-12-23' });
	nodes.set('B2', { value: '5000' });
	nodes.set('A3', { value: '2024-12-24' });
	nodes.set('B3', { value: '1200' });
	nodes.set('A4', { value: '2024-12-26' });
	nodes.set('B4', { value: '140' });
	nodes.set('B5', { value: '=SUM(B2:B4)' });

	let selectedNodes = new SvelteSet<string>(['0,0']);

	function getKey(row: number, col: number) {
		const rowName = row;
		const colName = String.fromCharCode(col + 65);
		return `${colName}${rowName}`;
	}

	function calculateFunction(value: string) {
		if (value.includes('=SUM')) {
			const start = value.indexOf('(') + 1;
			const end = value.indexOf(')');
			const range = value.substring(start, end);
			const values = range.split(':');
			let sum = 0;
			const startRow = parseInt(values[0].substring(1));
			const startCol = values[0].charCodeAt(0) - 65;
			const endRow = parseInt(values[1].substring(1));
			const endCol = values[1].charCodeAt(0) - 65;

			for (let row = startRow; row <= endRow; row++) {
				for (let col = startCol; col <= endCol; col++) {
					const node = nodes.get(getKey(row, col));
					if (node === undefined) continue;
					const value = parseInt(node.value);
					if (!isNaN(value)) {
						sum += value;
					}
				}
			}
			return sum;
		}
	}
</script>

<div class="flex h-0 w-full grow flex-col overflow-auto border-border/60 p-4">
	{#each Array.from({ length: 100 }) as _, row}
		<div class="flex w-min first:border-t">
			<button
				class={cn(
					'flex h-6 w-12 items-center justify-center border-b border-l border-border text-right text-xs font-semibold last:border-r'
				)}>{row}</button
			>
			{#each Array.from({ length: 100 }) as _, col}
				{#if row === 0}
					<button
						class={cn(
							'flex h-6 w-24 items-center justify-center border-b border-l border-border text-right text-xs font-semibold last:border-r'
						)}>{String.fromCharCode(col + 65)}</button
					>
				{:else}
					<button
						onclick={async (event) => {
							selectedNodes.add(getKey(row, col));
							const element = event.currentTarget;
							if (element.firstElementChild) {
								await tick();
								(element.firstElementChild as HTMLInputElement).focus();
							}
						}}
						onkeypress={async (event) => {}}
						onfocusin={(event) => {}}
						class={cn(
							'flex h-6 w-24 items-center justify-end border-b border-l border-border text-right text-xs font-semibold last:border-r',
							nodes.has(getKey(row, col)) &&
								isNaN(parseInt(nodes.get(getKey(row, col))!.value)) &&
								'justify-start'
						)}
					>
						{#if selectedNodes.has(getKey(row, col))}
							<input
								value={nodes.get(getKey(row, col)) ? nodes.get(getKey(row, col))!.value : ''}
								type="text"
								class="h-full w-full p-1"
								onfocusout={(e) => {
									selectedNodes.delete(getKey(row, col));
								}}
								onchange={(e) => {
									nodes.set(getKey(row, col), { value: e.currentTarget.value });
									selectedNodes.delete(getKey(row, col));
								}}
							/>
						{:else if nodes.get(getKey(row, col))}
							{#if nodes.get(getKey(row, col))!.value.startsWith('=')}
								<span class="p-1">{calculateFunction(nodes.get(getKey(row, col))!.value)}</span>
							{:else}
								<span class="p-1">{nodes.get(getKey(row, col))!.value}</span>
							{/if}
						{:else}
							<span></span>
						{/if}
					</button>
				{/if}
			{/each}
		</div>
	{/each}
</div>
