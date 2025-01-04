<script lang="ts">
	import { cn } from '$lib/utils';
	import { onMount, tick } from 'svelte';

	type Node = {
		value: string;
		editing: 'notEditing' | 'addingText' | 'fullyEditing';
		selected: boolean;
	};

	let cells = $state<Node[][]>(
		Array.from({ length: 200 }, () =>
			Array.from({ length: 200 }, () => ({ value: '', editing: 'notEditing', selected: false }))
		)
	);
	cells[1][0].value = 'Date';
	cells[1][1].value = 'Amount';

	cells[2][0].value = '2024-12-23';
	cells[2][1].value = '5000';

	cells[3][0].value = '2024-12-24';
	cells[3][1].value = '1200';

	cells[4][0].value = '2024-12-26';
	cells[4][1].value = '140';
	cells[4][1].selected = true;

	cells[5][1].value = '=SUM(B2:B4)';

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
					const node = cells[row][col];
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

	function getCurrentCellPosition(cells: Node[][]): [number, number] {
		let row = 0;
		let col = 0;

		cells.forEach((cellRow, i) => {
			cellRow.forEach((cell, j) => {
				if (cell.selected) {
					row = i;
					col = j;
				}
			});
		});
		return [row, col];
	}

	function getCurrentCell(cells: Node[][]) {
		return cells.find((row) => row.find((cell) => cell.selected));
	}

	onMount(() => {
		async function focusCell() {
			await tick();
			const id = 'button-' + getKey(...getCurrentCellPosition(cells));
			const element = document.getElementById(id) as HTMLButtonElement;
			element?.focus();
		}
		document.addEventListener('keydown', async (e) => {
			const [row, col] = getCurrentCellPosition(cells);
			const cell = cells[row][col];

			if (cell.editing === 'fullyEditing') return;

			if (e.key === 'ArrowUp' && cells[row - 2]?.[col] !== undefined) {
				cell.selected = false;
				cell.editing = 'notEditing';
				cells[row - 1][col].selected = true;
				focusCell();
			} else if (e.key === 'ArrowDown' && cells[row + 1]?.[col] !== undefined) {
				cell.selected = false;
				cell.editing = 'notEditing';
				cells[row + 1][col].selected = true;
				focusCell();
			} else if (e.key === 'ArrowLeft' && cells[row]?.[col - 1] !== undefined) {
				cell.selected = false;
				cell.editing = 'notEditing';
				cells[row][col - 1].selected = true;
				focusCell();
			} else if (e.key === 'ArrowRight' && cells[row]?.[col + 1] !== undefined) {
				cell.selected = false;
				cell.editing = 'notEditing';
				cells[row][col + 1].selected = true;
				focusCell();
			} else if (e.key === 'Enter') {
				cell.editing = 'fullyEditing';
				await tick();
				const element = document.getElementById(getKey(row, col)) as HTMLInputElement;
				element?.focus();
				return;
			}
		});
	});
</script>

<div class="flex h-0 w-full grow flex-col overflow-hidden border-border/60 p-4">
	{#each cells as row, rowIndex}
		<div class="flex w-min first:border-t">
			<button
				class={cn(
					'flex h-6 w-12 items-center justify-center border-b border-l border-border text-right text-xs font-semibold last:border-r'
				)}
				>{rowIndex}
			</button>
			{#each row as cell, colIndex}
				{#if rowIndex === 0}
					<button
						class={cn(
							'flex h-6 w-24 items-center justify-center border-b border-l border-border text-right text-xs font-semibold last:border-r'
						)}>{String.fromCharCode(colIndex + 65)}</button
					>
				{:else}
					<button
						id={'button-' + getKey(rowIndex, colIndex)}
						ondblclickcapture={async (e) => {
							if (cell.editing !== 'notEditing') return;
							cell.editing = 'fullyEditing';
							cell.selected = true;

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
							if (cell.editing !== 'notEditing') return;
							const [row, col] = getCurrentCellPosition(cells);
							cells[row][col].selected = false;
							cell.selected = true;
							cell.editing = 'notEditing';
						}}
						onkeypress={async (e) => {
							if (cell.editing !== 'notEditing') return;
							cell.editing = 'addingText';

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
						class={cn(
							'focus-within:outline-none',
							'flex h-6 w-24 items-center justify-end border-b border-l border-border text-right text-xs font-semibold last:border-r',
							isNaN(parseInt(cell.value)) && 'justify-start',
							cell.selected && 'border-2 border-primary'
						)}
					>
						{#if cell.selected && cell.editing !== 'notEditing'}
							<input
								id={getKey(rowIndex, colIndex)}
								value={cell.value ?? ''}
								type="text"
								class="h-full w-full p-1"
								onfocusout={(e) => {
									cell.editing = 'notEditing';
								}}
								onchange={(e) => {
									cell.value = e.currentTarget.value;
									cell.editing = 'notEditing';
								}}
							/>
						{:else if cell.value.length > 0}
							{#if cell.value.startsWith('=')}
								<span class="p-1">{calculateFunction(cell.value)}</span>
							{:else}
								<span class="p-1">{cell.value}</span>
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
