<script lang="ts">
	import { cn } from '$lib/utils';
	import { onMount, tick } from 'svelte';
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
	console.log(getKey(1, 1));

	let selectedNodes = new SvelteSet<string>(['0,0']);

	let activeNode = $state<[number, number]>([3, 3]);
	let editing = $state(false);

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

	onMount(() => {
		document.addEventListener('keydown', async (e) => {
			if (editing) return;

			editing = false;
			if (e.key === 'ArrowUp') {
				activeNode[0]--;
			} else if (e.key === 'ArrowDown') {
				activeNode[0]++;
			} else if (e.key === 'ArrowLeft') {
				activeNode[1]--;
			} else if (e.key === 'ArrowRight') {
				activeNode[1]++;
			} else if (e.key === 'Enter') {
				editing = true;
				await tick();
				const element = document.getElementById(getKey(...activeNode)) as
					| HTMLInputElement
					| undefined;

				if (element) {
					element.focus();
				}
			}
		});
	});
</script>

<div class="flex h-0 w-full grow flex-col overflow-hidden border-border/60 p-4">
	{#each Array.from({ length: 10 }) as _, row}
		{console.log('row')}
		<div class="flex w-min first:border-t">
			<button
				class={cn(
					'flex h-6 w-12 items-center justify-center border-b border-l border-border text-right text-xs font-semibold last:border-r'
				)}
				>{row}
			</button>
			{#each Array.from({ length: 10 }) as _, col}
				{#if row === 0}
					<button
						class={cn(
							'flex h-6 w-24 items-center justify-center border-b border-l border-border text-right text-xs font-semibold last:border-r'
						)}>{String.fromCharCode(col + 65)}</button
					>
				{:else}
					<button
						ondblclickcapture={async (e) => {
							if (editing) return;
							editing = true;
							activeNode = [row, col];

							const element = e.currentTarget;
							if (element.firstElementChild) {
								await tick();
								const input = element.firstElementChild as HTMLInputElement | undefined;
								if (input) {
									input.focus();
								}
							}
						}}
						onclick={async (event) => {
							if (editing) return;
							activeNode = [row, col];
							editing = false;
						}}
						onkeypress={async (e) => {
							if (editing) return;
							editing = true;
							activeNode = [row, col];

							const element = e.currentTarget;
							if (element.firstElementChild) {
								await tick();
								const input = element.firstElementChild as HTMLInputElement | undefined;
								if (input) {
									input.value = '';
									input.focus();
								}
							}
						}}
						onfocusout={(e) => {
							//console.log('focusout', getKey(row, col));
							/*nodes.set(getKey(row, col), { value: e.currentTarget.value });
							editing = false;*/
						}}
						class={cn(
							'focus-within:outline-none',
							'flex h-6 w-24 items-center justify-end border-b border-l border-border text-right text-xs font-semibold last:border-r',
							nodes.has(getKey(row, col)) &&
								isNaN(parseInt(nodes.get(getKey(row, col))!.value)) &&
								'justify-start',
							getKey(...activeNode) === getKey(row, col) && 'border-2 border-primary'
						)}
					>
						{#if getKey(...activeNode) === getKey(row, col) && editing}
							<input
								id={getKey(row, col)}
								value={nodes.get(getKey(row, col))?.value ?? ''}
								type="text"
								class="h-full w-full p-1"
								onfocusout={(e) => {
									//selectedNodes.delete(getKey(row, col));
									editing = false;
								}}
								onchange={(e) => {
									nodes.set(getKey(row, col), { value: e.currentTarget.value });
									//selectedNodes.delete(getKey(row, col));
									editing = false;
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
