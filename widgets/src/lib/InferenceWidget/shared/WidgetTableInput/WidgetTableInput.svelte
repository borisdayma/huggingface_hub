<script>
	import { onMount, tick } from "svelte";
	import { scrollToMax } from "../ViewUtils";
	import IconRow from "../../../Icons/IconRow.svelte";

	export let onChange: (table: string[][]) => void;
	export let highlighted: [y: number, x: number][] = [];
	export let table: string[][] = [[]];

	let initialTable: string[][] = [[]];
	let tableContainerEl: HTMLElement;

	onMount(() => {
		initialTable = table.map((row) => row.map((cell) => cell));
	});

	async function addCol() {
		const updatedTable = table.map((row, colIndex) => [
			...row,
			colIndex === 0 ? `Header ${table[0].length + 1}` : "",
		]);
		onChange(updatedTable);
		await tick();
		scrollToMax(tableContainerEl, "x");
	}

	function addRow() {
		const updatedTable = [...table, Array(table[0].length).fill("")];
		onChange(updatedTable);
	}

	function editCell(e: Event, [x, y]) {
		const value = (e.target as HTMLElement)?.innerText;

		const updatedTable = table.map((row, rowIndex) =>
			rowIndex === y
				? row.map((col, colIndex) => (colIndex === x ? value : col))
				: row
		);
		onChange(updatedTable);
	}

	function resetTable() {
		const updatedTable = initialTable;
		onChange(updatedTable);
	}
</script>

<div class="overflow-auto" bind:this={tableContainerEl}>
	<table class="table-question-answering">
		<thead>
			<tr>
				{#each table[0] as header, x}
					<th contenteditable on:input={(e) => editCell(e, [x, 0])}>
						{header}
					</th>
				{/each}
			</tr>
		</thead>
		<tbody>
			{#each table.slice(1) as row, y}
				<tr
					class={highlighted.some(([yCor]) => yCor === y)
						? "bg-green-50"
						: "bg-white"}
				>
					{#each row as cell, x}
						<td
							class={highlighted.some(
								([yCor, xCor]) => yCor === y && xCor === x
							)
								? "bg-green-100 border-green-100"
								: "border-gray-100"}
							contenteditable
							on:input={(e) => editCell(e, [x, y + 1])}>{cell}</td
						>
					{/each}
				</tr>
			{/each}
		</tbody>
	</table>
</div>

<div class="flex mb-1 flex-wrap">
	<button
		class="btn-widget flex-1 lg:flex-none mt-2  mr-1.5"
		on:click={addRow}
		type="button"
	>
		<IconRow classNames="mr-2" />
		Add row
	</button>
	<button
		class="btn-widget flex-1 lg:flex-none mt-2 lg:mr-1.5"
		on:click={addCol}
		type="button"
	>
		<IconRow classNames="transform rotate-90 mr-1" />
		Add col
	</button>
	<button
		class="btn-widget flex-1 mt-2 lg:flex-none lg:ml-auto"
		on:click={resetTable}
		type="button"
	>
		Reset table
	</button>
</div>
