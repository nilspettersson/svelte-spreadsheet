<script lang="ts">
	import { cn } from '$lib/utils';
	import { onMount, tick } from 'svelte';
	import MaterialSymbolsFunctionRounded from '~icons/material-symbols/function-rounded';

	type Node = {
		value: string;
		editing: 'notEditing' | 'addingText' | 'fullyEditing';
		selected: boolean;
	};

	let cells = $state<Node[][]>(
		Array.from({ length: 1001 }, () =>
			Array.from({ length: 26 }, () => ({ value: '', editing: 'notEditing', selected: false }))
		)
	);

	if (typeof localStorage !== 'undefined' && localStorage.getItem('cells')) {
		const values = JSON.parse(localStorage.getItem('cells')!) as string[][];
		values.forEach((row, i) =>
			row.forEach((value, j) => {
				if (value !== '') {
					cells[i][j].value = value;
				}
			})
		);
	}

	$effect(() => {
		localStorage.setItem(
			'cells',
			JSON.stringify(cells.map((row) => row.map((cell) => cell.value)))
		);
	});

	function getKey(row: number, col: number) {
		const rowName = row;
		const colName = String.fromCharCode(col + 65);
		return `${colName}${rowName}`;
	}

	function calculateFunction(value: string) {
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

		if (value.includes('=SUM')) {
			return sum;
		} else if (value.includes('=AVERAGE')) {
			const totalCellCount = (startCol - endCol + 1) * (endRow - startRow + 1);
			return sum / totalCellCount;
		}
	}

	let selectedCell: [number, number] | null = $state(null);
	//setSelectedCell(4, 1);
	function setSelectedCell(row: number, col: number) {
		if (selectedCell !== null) {
			cells[selectedCell[0]][selectedCell[1]].selected = false;
		}
		cells[row][col].selected = true;
		selectedCell = [row, col];
	}

	onMount(() => {
		async function focusCell() {
			if (selectedCell === null) return;
			await tick();
			const id = 'button-' + getKey(...selectedCell);
			const element = document.getElementById(id) as HTMLButtonElement;
			element?.focus();
		}
		document.addEventListener('keydown', async (e) => {
			if (selectedCell === null) return;
			const [row, col] = selectedCell;
			const cell = cells[row][col];
			if (cell.editing === 'fullyEditing') return;

			if (e.key === 'ArrowUp' && cells[row - 2]?.[col] !== undefined) {
				setSelectedCell(row - 1, col);
			} else if (e.key === 'ArrowDown' && cells[row + 1]?.[col] !== undefined) {
				setSelectedCell(row + 1, col);
			} else if (e.key === 'ArrowLeft' && cells[row]?.[col - 1] !== undefined) {
				setSelectedCell(row, col - 1);
			} else if (e.key === 'ArrowRight' && cells[row]?.[col + 1] !== undefined) {
				setSelectedCell(row, col + 1);
			} else if (e.key === 'Enter' && cell.editing !== 'addingText') {
				cell.editing = 'fullyEditing';
				await tick();
				const element = document.getElementById(getKey(row, col)) as HTMLInputElement;
				element?.focus();
				console.log(element);

				return;
			}

			if (
				e.key === 'ArrowUp' ||
				e.key === 'ArrowDown' ||
				e.key === 'ArrowLeft' ||
				e.key === 'ArrowRight'
			) {
				cell.editing = 'notEditing';
				e.preventDefault();
				e.stopImmediatePropagation();
				focusCell();
			}
		});
	});
</script>

<div class="flex gap-2 py-2">
	<p class="w-12 border-r text-center text-sm">{getKey(...(selectedCell ?? [0, 0]))}</p>
	<div class="flex items-center">
		<MaterialSymbolsFunctionRounded class="size-5 text-muted-foreground" />
		<p class="text-sm text-foreground/80">
			{cells[selectedCell?.[0] ?? 0][selectedCell?.[1] ?? 0].value}
		</p>
	</div>
</div>
<div class="flex h-0 w-full grow flex-col overflow-auto border-border/60">
	{#each cells as row, rowIndex}
		<div class="flex w-min first:border-t">
			<button
				class={cn(
					'flex h-6 w-12 items-center justify-center border-b border-l border-border text-right text-sm last:border-r'
				)}
				>{rowIndex}
			</button>
			{#each row as cell, colIndex}
				{#if rowIndex === 0}
					<button
						class={cn(
							'flex h-6 w-24 items-center justify-center border-b border-l border-border text-right text-sm last:border-r'
						)}>{String.fromCharCode(colIndex + 65)}</button
					>
				{:else}
					<button
						id={'button-' + getKey(rowIndex, colIndex)}
						ondblclickcapture={async (e) => {
							if (cell.editing !== 'notEditing') return;
							cell.editing = 'fullyEditing';
							setSelectedCell(rowIndex, colIndex);

							const element = e.currentTarget;
							if (element.firstElementChild) {
								await tick();
								const input = element.firstElementChild as HTMLInputElement | undefined;
								if (input) {
									input.focus();
								}
							}
						}}
						onmousedown={async (e) => {
							if (cell.editing !== 'notEditing') return;
							setSelectedCell(rowIndex, colIndex);
							cell.editing = 'notEditing';
						}}
						onkeydown={async (e) => {
							if (cell.editing !== 'notEditing' || e.key === 'Enter') return;

							if (e.key === 'Backspace') {
								cell.value = '';
								return;
							}
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
							'flex h-6 w-24 items-center justify-end overflow-hidden whitespace-nowrap border-b border-l border-border text-right text-sm last:border-r',
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
								onkeydown={async (e) => {
									if (e.key === 'Enter' && cell.editing !== 'notEditing') {
										e.stopImmediatePropagation();
										cell.editing = 'notEditing';
										cell.value = e.currentTarget.value;
										setSelectedCell(rowIndex + 1, colIndex);
										await tick();
										const element = document.getElementById(
											'button-' + getKey(rowIndex + 1, colIndex)
										) as HTMLButtonElement | undefined;
										if (element) {
											if (element) {
												element.focus();
											}
										}
									}
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
