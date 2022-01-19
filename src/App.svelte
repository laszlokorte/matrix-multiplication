<script>
	const rowLimit = 5
	const columnLimit = 5
	let rowsA = 4
	let columnsA = 3
	$: rowsB = columnsA
	let columnsB = 4
	let gridAlign = false
	
	let focus = null
	
	let valuesA = Array(rowLimit*columnLimit).fill(0).map(() => Math.round(10*Math.random()))
	let valuesB = Array(rowLimit*columnLimit).fill(0).map(() => Math.round(10*Math.random()))
	
	$: columnsResult = columnsB
	$: rowsResult = rowsA
		
	$: rowsACanIncrease = rowsA < rowLimit
	$: columnsACanIncrease = columnsA < columnLimit &&  rowsB < rowLimit
	$: rowsBCanIncrease = rowsB < rowLimit && columnsA < columnLimit
	$: columnsBCanIncrease = columnsB < columnLimit
	
	$: rowsACanDecrease = rowsA > 1
	$: columnsACanDecrease = columnsA > 1
	$: rowsBCanDecrease = rowsB > 1
	$: columnsBCanDecrease = columnsB >1
	
	$: matrixA = Array(rowsA).fill(0).map((v,r) => Array(columnsA).fill(0).map((_,c) => r*columnLimit + c))
	$: matrixB = Array(rowsB).fill(0).map((v,r) => Array(columnsB).fill(0).map((_,c) => r*columnLimit + c))
	
  $: result = Array(rowsA).fill(0).map((v,r) => Array(columnsB).fill(0).map((_, c) => {
		let sum = 0;
		
		for(let x=0;x<columnsA;x++) {
			sum += valuesA[r*columnLimit + x]*valuesB[x*columnLimit + c]
		}
		
		return sum
	}))
	
	$: sum = focus ? Array(columnsA).fill(0).map((_,x) => [
		focus.row*columnLimit + x, x*columnLimit + focus.column
	]) : null

	function setRowsA(num) {
		resetFocus()
		rowsA = num
	}
	
	function setColumnsA(num) {
		resetFocus()
		columnsA = num
	}
	
	function setRowsB(num) {
		resetFocus()
		setColumnsA(num)
	}
	
	function setColumnsB(num) {
		resetFocus()
		columnsB = num
	}
	
	function increaseRowsA() {
		resetFocus()
		rowsA++
	}
	
	function decreaseRowsA() {
		resetFocus()
		rowsA--
	}
	
	function increaseColumnsA() {
		resetFocus()
		columnsA++
	}
	
	function decreaseColumnsA() {
		resetFocus()
		columnsA--
	}
	
	function increaseRowsB() {
		resetFocus()
		columnsA++
	}
	
	function decreaseRowsB() {
		resetFocus()
		columnsA--
	}
	
	function increaseColumnsB() {
		resetFocus()
		columnsB++
	}
	
	function decreaseColumnsB() {
		resetFocus()
		columnsB--
	}
	
	function resetFocus() {
		focus = null
	}
	
	function setFocus(evt) {
		focus = {row: 1*evt.currentTarget.dataset.row, column: 1*evt.currentTarget.dataset.column}
	}
	
	function setFocusStep(num) {
		if(num < 0) {
			focus = null
		} else {
			focus = {column: num%columnsB, row: Math.floor(num/columnsB)}
		}
	}

	function jumpStep(evt) {
		console.log(evt.currentTarget.dataset.step)
		setFocusStep(1*evt.currentTarget.dataset.step)
	}
	
	function shuffleA() {
		valuesA = Array(rowLimit*columnLimit).fill(0).map(() => Math.round(10*Math.random()))
	}
	
	function shuffleB() {
		valuesB = Array(rowLimit*columnLimit).fill(0).map(() => Math.round(10*Math.random()))
	}
	
	function clearA() {
		valuesA = Array(rowLimit*columnLimit).fill(0)
	}
	
	function clearB() {
		valuesB = Array(rowLimit*columnLimit).fill(0)
	}
	
	function diagonalA() {
		valuesA = Array(rowLimit*columnLimit).fill(0).map((x,i) => i%columnLimit == Math.floor(i/columnLimit) ? 1 : x)
	}
	
	function diagonalB() {
		valuesB = Array(rowLimit*columnLimit).fill(0).map((x,i) => i%columnLimit == Math.floor(i/columnLimit) ? 1 : x)
	}
</script>

<style>
	.equation {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 1em 0;
	}

	@media (max-width: 60em) {
		.equation {
			flex-wrap: wrap;
		}
	}

	.equation.grid-align {
		display: grid;
		grid-template-columns: 
		[operator-start op1-start] auto 
		[equals-start op1-end ] auto 
		[operator-end equals-end op2-start result-start] auto 
		[op2-end result-end];
		grid-template-rows: [op2-start operator-start] auto [operator-end op2-end op1-start result-start equals-start] auto [equals-end op1-end result-end];
	}

	.grid-align > .grid-operand-1 {
		grid-area: op1;
	}

	.grid-align > .grid-operand-2 {
		grid-area: op2;
	}

	.grid-align > .grid-result {
		grid-area: result;
	}

	.grid-align > .grid-operator {
		grid-area: operator;
		align-self: center;
		justify-self: center;
		font-size: 3em;
	}

	.grid-align > .grid-equals {
		grid-area: equals;
		font-size: 3em;
	}
	
	.matrix {
		--columns: 1;
		--rows: 1;
		display: grid;
		grid-template-columns: [full-start start-start] 0.3em [start-end inner-start] repeat(var(--columns), 3em) [end-start inner-end] 0.3em [end-end full-end];
		grid-template-rows:  [full-start inner-start] repeat(var(--rows), 3em) [inner-end full-end];
		gap: 0.2em;
		padding: 0.5em;
		grid-area: content;
	}
	
	.matrix::before {
		content: '';
		display: block;
		grid-column: start;
		grid-row: 1 / -1;
		border-left: 2px solid black;
		border-top: 2px solid black;
		border-bottom: 2px solid black;
	}
	
	.matrix::after {
		content: '';
		display: block;
		grid-column: end;
		grid-row: 1 / -1;
		border-right: 2px solid black;
		border-top: 2px solid black;
		border-bottom: 2px solid black;
	}
	
	input {
		margin: 0;
		text-align: center;
		border: 1px solid #ddd;
	}
	
	output {
		margin: 0;
		text-align: center;
		display: flex;
		align-items: center;
		justify-content: center;
		border: 1px solid #ddd;
		cursor: pointer;
	}
	
	.operator {
		margin: 1em;
	}
	
	.matrix-control {
		display: grid;
		grid-template-columns: [vertical-start] minmax(2em, 1fr) [vertical-end content-start horizontal-start label-start] auto [content-end horizontal-end label-end] minmax(2em, 1fr);
		grid-template-rows: [horizontal-start] minmax(2em, 1fr) [horizontal-end vertical-start content-start] auto [ vertical-end content-end label-start] minmax(2em, 1fr) [label-end];
		align-items: start;
		justify-items: start;
		gap: 0.5em;
	}
	
	.matrix-label {
		grid-area: label;
		align-self: center;
		justify-self: center;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.matrix-label > span {
		white-space: nowrap;
	}
	
	.control-corner {
		grid-column: vertical;
		grid-row: horizontal;
		gap: 0.5em;
		padding: 0.2em;
		align-self: end;
	}
	
	.control-vertical {
		grid-area: vertical;
		display: flex;
		flex-direction: column;
		gap: 0.5em;
		padding: 0.2em;
	}
	
	.control-horizontal {
		grid-area: horizontal;
		display: flex;
		gap: 0.5em;
		padding: 0.2em;
		align-self: end;
	}
	
	.control-top {
		grid-area: horizontal;
		display: flex;
		gap: 0.5em;
		padding: 0.2em;
		align-self: end;
		justify-self: center;
	}
	
	button {
		width: 2em;
		height: 2em;
		padding: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		justify-items: center;
		justify-content: center;
		cursor: pointer;
		margin: 0;
		background: #0ad;
		color: #fff;
		font-weight: bold;
		border: none;
		box-shadow: 1px 1px 3px #4444;
		border-radius: 10%;
		line-height: 1;
		font-size: 1em;
		font-family: sans-serif;
	}
	
	button:disabled {
		cursor: not-allowed;
		background: #aaa;
	}
	
	button:focus {
		outline: 4px solid orange;
	}

	.button-row {
		display: flex;
		gap: 0.5em;
		margin: 1em;
	}
	
	.focus {
		grid-area: inner;
		background: #f0a7;
		padding: 0.5em;
	}
	
	.focus.vertical {
		background: #0af7;
	}
	
	.focus.horizontal {
		background: #0fa7;
	}
	
	.ghost {
		pointer-events: none;
	}
	
	.group {
		display: flex;
		align-items: center;
		border-left: 3px solid black;
		border-right: 3px solid black;
		border-top-left-radius: 0.5em 1em;
		border-top-right-radius: 0.5em 1em;
		border-bottom-left-radius: 0.5em 1em;
		border-bottom-right-radius: 0.5em 1em;
		padding: 0 0.5em;
	}
	
	dl {
		display: grid;
		grid-template-columns: max-content max-content max-content;
		align-items: center;
		justify-content: start;
		gap: 1em;
	}
	
	dt {
		margin: 0;
		padding: 0;
	}
	
	dd {
		margin: 0;
		padding: 0;
		display: flex;
		gap:  0.5em;
	}
	
	h1 {
		text-align: center;
		font-weight: normal;
	}
	
	label {
		font-weight: bold;
		cursor: pointer;
	}
	
	.controls {
		display: grid;
		max-width: max-content;
		justify-items: center;
		margin: auto;
		gap: 1em;
		padding: 0 3em 1em;
	}

	p {
		max-width: 40em;
		margin: 1em auto;
	}
</style>






<div class="controls">

<h1>
	Matrix Multiplication
</h1>
	
<p>
	Below you can enter two matrices to see how they are multiplied. Use the round buttons to configure the matrix dimensions. The height of Matrix B must be equal to the width of Matrix A.
</p>

<p>
	By checking the <strong>Align Matrices</strong> checkbox you can align the Matrix B above the result matrix so that the resulting cells match the rows and columns of their calculation. 
</p>
	
	<div class="equation" class:grid-align={gridAlign}>
	<div class="matrix-control grid-operand-1">
		<div class="matrix-label">
			<span>Matrix A</span>
			<span>({rowsA} rows, {columnsA} columns)</span>
			<span class="button-row">
				<button title="Clear Matrix A" on:click={clearA}>0</button>
				<button title="Make Matrix A diagonal" on:click={diagonalA}>⋱</button>
				<button title="Shuffle value of Matrix A" on:click={shuffleA}>⚄</button>
			</span>
		</div>
		<div class="control-corner">
		</div>
		<div class="control-vertical">
			<button title="Decrease rows of Matrix A" on:click={decreaseRowsA} disabled={!rowsACanDecrease}>－</button>
			<button title="Increase rows of Matrix A" on:click={increaseRowsA} disabled={!rowsACanIncrease}>＋</button>
		</div>
		<div class="control-horizontal">
			<button title="Decrease columns of Matrix A" on:click={decreaseColumnsA} disabled={!columnsACanDecrease}>－</button>
			<button title="Increase columns of Matrix A" on:click={increaseColumnsA} disabled={!columnsACanIncrease}>＋</button>
		</div>
		<div class="matrix" style={`--rows: ${rowsA}; --columns: ${columnsA};`}>
			{#each matrixA as rows}
			{#each rows as id}
			<input type="text" bind:value={valuesA[id]} />
			{/each}
			{/each}
		</div>
		<div class="matrix ghost" style={`--rows: ${rowsA}; --columns: ${columnsA};`}>
			{#if focus}
			<div class="focus horizontal" style={`grid-row: ${focus.row+1} / span 1`}>
				
			</div>
			{/if}
		</div>
	</div>
	<div class="operator grid-operator">
		<label for="alignment" titel="Toggle matrix alignment">&times;</label>
	</div>
	
	<div class="matrix-control grid-operand-2">
		<div class="matrix-label">
			<span>Matrix B</span>
			<span>({rowsB} rows, {columnsB} columns)</span>
			<span class="button-row">
				<button title="Clear Matrix B" on:click={clearB}>0</button>
				<button title="Make Matrix B diagonal" on:click={diagonalB}>⋱</button>
				<button on:click={shuffleB} title="Shuffle values of Matrix B">⚄</button>
			</span>
		</div>
		<div class="control-corner">
		</div>
		<div class="control-vertical">
			<button title="Decrease rows of Matrix B" on:click={decreaseRowsB} disabled={!rowsBCanDecrease}>－</button>
			<button title="Increase rows of Matrix B" on:click={increaseRowsB} disabled={!rowsBCanIncrease}>＋</button>
		</div>
		<div class="control-horizontal">
			<button title="Decrease columns of Matrix B" on:click={decreaseColumnsB} disabled={!columnsBCanDecrease}>－</button>
			<button title="Increase columns of Matrix B" on:click={increaseColumnsB} disabled={!columnsBCanIncrease}>＋</button>
		</div>
		<div class="matrix" style={`--rows: ${rowsB}; --columns: ${columnsB};`}>
		{#each matrixB as rows}
			{#each rows as id}
			<input type="text" bind:value={valuesB[id]} />
			{/each}
			{/each}
		</div>
		<div class="matrix ghost" style={`--rows: ${rowsB}; --columns: ${columnsB};`}>
			{#if focus}
			<div class="focus vertical" style={`grid-column: ${focus.column+2} / span 1;`}>
				
			</div>
			{/if}
		</div>
	</div>
	<div class="operator grid-equals">
		=
	</div>
	
	<div class="matrix-control grid-result">
		<div class="control-top">
			<label><input id="alignment" type="checkbox" bind:checked={gridAlign}> Align matrices</label>
		</div>
		<div class="matrix-label"><span>Result</span><span>({rowsResult} rows, {columnsResult} columns)</span></div>
	<div class="matrix" style={`--rows: ${rowsA}; --columns: ${columnsB};`}>
	{#each result as rows, r}
		{#each rows as v, c}
		<output data-row={r} data-column={c} on:click={setFocus}>{v}</output>
		{/each}
		{/each}
	</div>
		<div class="matrix ghost" style={`--rows: ${rowsA}; --columns: ${columnsB};`}>
			{#if focus}
			<div class="focus" style={`grid-column: ${focus.column+2} / span 1;grid-row: ${focus.row+1} / span 1;`}>
			</div>
			{/if}
		</div>
	</div>
</div>
	
<dl>
	<dt><label for="step">Step: <span>{focus ? focus.row*columnsB + focus.column + 1 : '-'}/{rowsA*columnsB}</span></label></dt>
	<dd><input id="step" type="range" min="0" max={rowsA*columnsB} value={focus ? focus.row*columnsB + focus.column + 1 : 0} on:input={(evt) => setFocusStep(evt.currentTarget.value - 1)} /></dd>
	<dd>
		<button data-step={focus ? focus.row*columnsB + focus.column - 1 : -1} on:click={jumpStep} disabled={!focus || focus.row*columnsB + focus.column + 1 < 1}>⮜</button>
		<button data-step={focus ? focus.row*columnsB + focus.column + 1 : 0} disabled={focus && focus.row*columnsB + focus.column + 1 + 1 > rowsA*columnsB} on:click={jumpStep}>⮞</button></dd>
</dl>

{#if sum}
<div class="equation">
	{#each sum as [a,b],i}
	{#if i > 0}
	<div class="operator">
		+
	</div>
	{/if}
	<div  class="group">
		<div class="focus horizontal">
			{valuesA[a]}
		</div>
		<div class="operator">
			&times;
		</div>
		<div class="focus vertical">
			{valuesB[b]}
		</div>
	</div>
	{/each}
	
	<div class="operator">
		=
	</div>
	<div class="focus">
		{result[focus.row][focus.column]}
	</div>
</div>
{:else}
<p>
	Select a cell in the result matrix to see how it is calculated or use the step slider to iterate through all cells in the result matrix.
</p>
{/if}
</div>
