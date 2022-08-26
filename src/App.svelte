<script>
	import {onMount} from 'svelte'

	const rowLimit = 7
	const columnLimit = rowLimit
	let rowsA = 4
	let columnsA = 3
	$: rowsB = columnsA
	let columnsB = 4
	let gridAlign = false

	let useComplexNumbers = true
	let complexAsMatrix = false
	
	$: showComplexMatrix = useComplexNumbers && complexAsMatrix
	$: maxFocus = (rowsA*columnsB) * (showComplexMatrix ? 4 : 1)

	let focus = null
	let inputFocus = 'a-3'

	$: focusNumber = focus ? (focus.row*columnsB*(showComplexMatrix?4:1) + (focus.column)*(showComplexMatrix?2:1))  : 0

	let validA = Array(rowLimit*columnLimit).fill(0).map(() => true)
	let validB = Array(rowLimit*columnLimit).fill(0).map(() => true)
	
	let valuesA = Array(rowLimit*columnLimit).fill(0).map(() => [
		Math.round(10*Math.random()),
		0*Math.round(10*Math.random()),
	])

	let valuesB = Array(rowLimit*columnLimit).fill(0).map(() => [
		Math.round(10*Math.random()),
		0*Math.round(10*Math.random()),
	])
	
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
		let sum = [0,0];
		
		for(let x=0;x<columnsA;x++) {
			sum = sumNumbers(
				sum, 
				multiplyNumbers(
					valuesA[r*columnLimit + x], 
					valuesB[x*columnLimit + c], 
					useComplexNumbers), 
				useComplexNumbers
			)
		}
		
		return sum
	}))
	
	$: sum = focus ? Array(columnsA).fill(0).map((_,x) => [
		focus.row*columnLimit + x, x*columnLimit + focus.column
	]) : null

	function sumNumbers(a,b,complex=false) {
		return [
			a[0] + b[0],
			a[1] + b[1],
		]
	}

	function multiplyNumbers(a,b,complex=false) {
		if(complex) {
			return [
				a[0] * b[0] - a[1] * b[1],
				a[0] * b[1] + a[1] * b[0],
			]
		} else {
			return [
				a[0] * b[0],
				a[1] * b[1],
			]
		}
	}

	function dropIfOne(num) {
		if(Math.abs(num) === 1) {
			return ''
		} else {
			return Math.abs(num)
		}
	}

	function onlySign(num, dropPlus = false) {
		return num > 0 ? (!dropPlus ? '+' : '') : '-'
	}

	function numberToString(num,complex=false) {
		if(complex) {
			if(num[0] && num[1]) {
				return `${num[0]}${onlySign(num[1])}${dropIfOne(num[1])}j`
			} else if(num[1]) {
				return `${onlySign(num[1], true)}${dropIfOne(num[1])}j`
			} else {
				return `${num[0]}`
			}
		} else {
			return `${num[0]}`
		}
	}

	const splitRegExp = new RegExp('(?=[+-])') 

	function parseImaginary(str) {
		str = str.trim()
		const isComplex = str[str.length-1] === 'j'
		let stripped = isComplex ? str.substr(0, str.length-1).trim() : str
		if(isComplex && (stripped === '' || stripped === '+' || stripped ==='-')) {
			stripped += '1'
		}
		return {
			isComplex: isComplex,
			value: parseFloat(stripped),
			invalid: !isComplex && (stripped === '' || stripped === '+' || stripped ==='-') || isNaN(parseFloat(stripped))
		}
	}

	function changeInput(matrix, id, evt) {
		if(matrix === 'a') {
			validA[id] = false
		} else if(matrix === 'b') {
			validB[id] = false
		}

		const value = evt.currentTarget.value;

		if(!value) {
			return
		}

		const summands = value.split(splitRegExp)


		if(summands.length > 2 || summands.length < 1) {
			return
		}

		const dimensions = summands.map(parseImaginary)

		if(!dimensions.reduce((isC, d) => isC ^ d.isComplex)) {
			return
		}

		if(dimensions.reduce((isC, d) => isC || d.invalid, false)) {
			return
		}

		const both = [
			summands[0] ? dimensions[0].value : 0,
			summands[1] ? dimensions[1].value : 0,
		]

		const realPart = !dimensions[0].isComplex ? both[0] : both[1]
		const complexPart = dimensions[0].isComplex ? both[0] : both[1]

		const newNumber = [realPart, complexPart]

		if(matrix === 'a') {
			valuesA[id] = newNumber
			validA[id] = true
		} else if(matrix === 'b') {
			valuesB[id] = newNumber
			validB[id] = true
		}

		evt.currentTarget.value = numberToString(newNumber, useComplexNumbers)
	}

	function changeInputSingle(matrix, id, dim, evt) {

		if(matrix === 'a') {
			validA[id] = false
		} else if(matrix === 'b') {
			validB[id] = false
		}

		const value = evt.currentTarget.value;

		if(Math.abs(dim) > 1) {
			return
		}

		const newNumber = (dim ? dim : 1) * parseFloat(value)

		if(isNaN(newNumber)) {
			return
		}

		if(matrix === 'a') {
			valuesA[id][Math.abs(dim)] = newNumber
			validA[id] = true
		} else if(matrix === 'b') {
			valuesB[id][Math.abs(dim)] = newNumber
			validB[id] = true
		}

		evt.currentTarget.value = newNumber
	}

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
		focus = {
			row: 1*evt.currentTarget.dataset.row, 
			column: 1*evt.currentTarget.dataset.column,
			dim: Object.hasOwnProperty(evt.currentTarget.dataset, ) ? evt.currentTarget.dataset.column : 0
		}
	}
	
	function setFocusStep(num) {
		const multiplyer = showComplexMatrix ? 2 : 1
		if(num < 0) {
			focus = null
		} else {
			focus = {
				column: Math.round((num%(columnsB*multiplyer))/multiplyer), 
				row: Math.round(num/columnsB/multiplyer/multiplyer),
				dim: ((num%(showComplexMatrix*showComplexMatrix)+4)%4),
			}
		}
	}

	function jumpStep(evt) {
		setFocusStep(1*evt.currentTarget.dataset.step)
	}
	
	function shuffleA() {
		valuesA = Array(rowLimit*columnLimit).fill(0).map(() => [
			Math.round(10*Math.random()),
			Math.round(10*Math.random())
		])
		validA = Array(rowLimit*columnLimit).fill(0).map(() => true)
	}
	
	function shuffleB() {
		valuesB = Array(rowLimit*columnLimit).fill(0).map(() => [
			Math.round(10*Math.random()),
			Math.round(10*Math.random())
		])
		validB = Array(rowLimit*columnLimit).fill(0).map(() => true)
	}
	
	function clearA() {
		valuesA = Array(rowLimit*columnLimit).fill([0,0])
		validA = Array(rowLimit*columnLimit).fill(0).map(() => true)
	}
	
	function clearB() {
		valuesB = Array(rowLimit*columnLimit).fill([0,0])
		validB = Array(rowLimit*columnLimit).fill(0).map(() => true)
	}
	
	function diagonalA() {
		valuesA = Array(rowLimit*columnLimit).fill([0,0]).map((x,i) => i%columnLimit == Math.floor(i/columnLimit) ? [1,0] : x)
		validA = Array(rowLimit*columnLimit).fill(0).map(() => true)
	}
	
	function diagonalB() {
		valuesB = Array(rowLimit*columnLimit).fill([0,0]).map((x,i) => i%columnLimit == Math.floor(i/columnLimit) ? [1,0] : x)
		validB = Array(rowLimit*columnLimit).fill(0).map(() => true)
	}

	function conjugate(number) {
		return [number[0],-number[1]]
	}

	function transpose() {
		const newValuesA = Array(rowLimit*columnLimit).fill(0).map((_,num) => {
			const column = num%columnLimit
			const row = Math.floor(num/columnLimit)

			return conjugate(valuesB[column*columnLimit + row])
		})
		const newValuesB = Array(rowLimit*columnLimit).fill(0).map((_,num) => {
			const column = num%columnLimit
			const row = Math.floor(num/columnLimit)

			return conjugate(valuesA[column*columnLimit + row])
		})

		valuesA = newValuesA
		valuesB = newValuesB
		validA = Array(rowLimit*columnLimit).fill(0).map(() => true)
		validB = Array(rowLimit*columnLimit).fill(0).map(() => true)

		const oldRowsA = rowsA
		rowsA = columnsB
		columnsA = rowsB
		columnsB = oldRowsA
	}

	const hashparser = new RegExp('^(\\d+):(\\d+):(\\d+):(\\d+)(a?)((?:;[^;\\*]*)*)(\\*?[^;\\*]*(?:;[^;\\*]*)*)$','i')
	function loadHash(hash) {
		const m = hashparser.exec(hash)
		if(m !== null) {
			const [parsedRowsA, parsedColumnsA, parsedColumnsB, parsedFocus] = m.slice(1, 5).map((x) => parseInt(x, 10))
			const align = m[5] == 'a'

			const parsedValuesA = m[6].split(';').map((x) => x?parseFloat(x):0).slice(1)
			const parsedValuesB = m[7] ? m[7].slice(1).split(';').map((x) => x?parseFloat(x):0) : []

			let valid = true
			valid &&= parsedRowsA < rowLimit
			valid &&= parsedColumnsA < rowLimit
			valid &&= parsedColumnsA < columnLimit
			valid &&= parsedColumnsB < columnLimit
			valid &&= parsedFocus <= parsedRowsA*parsedColumnsB

			if(valid) {
				rowsA = parsedRowsA
				columnsA = parsedColumnsA
				columnsB = parsedColumnsB
				gridAlign = align
				focus = parsedFocus < 1 ? null : {column: (parsedFocus-1)%columnsB, row: Math.floor((parsedFocus-1)/columnsB)}
				valuesA = valuesA.map((_,i) => parsedValuesA[i]||0)
				valuesB = valuesB.map((_,i) => parsedValuesB[i]||0)
				validA = Array(rowLimit*columnLimit).fill(0).map(() => true)
				validB = Array(rowLimit*columnLimit).fill(0).map(() => true)
			}
		}
	}

	onMount(() => {
		const listener = () => {
			loadHash(window.location.hash.substr(1))
		}

		window.addEventListener('hashchange', listener)
		listener()
		return () => window.removeEventListener('hashchange', listener)
	})

	$: shareUrl = window.location.origin + window.location.pathname + '#' 
	+ rowsA + ':' 
	+ columnsA + ':' 
	+ columnsB + ':' 
	+ (focus ? focus.row*columnsB + focus.column + 1 : 0) 
	+ (gridAlign?'a':'') 
	+ ';' + valuesA.join(';') + '*' + valuesB.join(';')
</script>

<style>
	:root {
		--cellsize: 3.5em;
	}

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
		--subdivision: 1;
		display: grid;
		grid-template-columns: [full-start start-start] 0.3em [start-end inner-start] repeat(var(--columns, 1), calc(var(--cellsize, 3em) * var(--subdivision, 1))) [end-start inner-end] 0.3em [end-end full-end];
		grid-template-rows:  [full-start inner-start] repeat(var(--rows, 1), calc(var(--cellsize, 3em) * var(--subdivision, 1))) [inner-end full-end];
		gap: 0.2em;
		padding: 0.5em;
		grid-area: content;
		justify-self: stretch;
		align-self: stretch;
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
		background: none;
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
		grid-template-columns: [vertical-start label-start] minmax(2em, 1fr) [vertical-end content-start horizontal-start] auto [content-end horizontal-end] minmax(2em, 1fr) [label-end];
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
		margin: 0 auto;
	}

	.sharelink {
		width: 100%;
		max-width: 20em;
		text-align: left;
	}

	h3 {
		margin: 3em 0 0;
	}

	.inline-list {
		display: flex;
		list-style: none;
		margin: 0;
		padding: 0.5em;
		gap: 0.5em 2em;
		margin: auto;
	}

	.inline-list label {
		font-weight: normal;
	}

	.hidden {
		visibility: hidden;
	}

	.complex-matrix {
		display: grid;
		grid-template-columns: var(--cellsize, 3em) var(--cellsize, 3em);
		grid-template-rows: var(--cellsize, 3em) var(--cellsize, 3em);
		gap: 0.1em;
	}

	.focused {
		background: #ffaa;
	}

	.invalid {
		outline: 2px solid red;
		background: #faa5;
	}

	fieldset {
		border: 2px solid #ccc;
	}

	legend {
		background: #333;
		color: #fff;
		display: inline-block;
		padding: 0.3em 0.7em;
		margin: 0.3em;
	}

	fieldset label {
		padding: 0.3em;
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
<p>
	The Result Matrix can be tranposed (⬔, rows and columns swapped) by swapping Matrix A and Matrix B and transposing each of them.
</p>

<fieldset>
	<legend>Domain</legend>

	<ul class="inline-list">
		<li><label><input type="radio" bind:group={useComplexNumbers} value={false}> Real Numbers</label></li>
		<li><label><input type="radio" bind:group={useComplexNumbers} value={true}> Complex Numbers</label></li>
	</ul>

	<ul class="inline-list" class:hidden={!useComplexNumbers}>
		<li><label><input type="radio" bind:group={complexAsMatrix} value={false}> Display complex as algebraic</label></li>
		<li><label><input type="radio" bind:group={complexAsMatrix} value={true}> Display complex as matrix</label></li>
	</ul>
</fieldset>
	
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
		<div class="matrix" style={`--rows: ${rowsA}; --columns: ${columnsA}; --subdivision: ${showComplexMatrix ? 2 : 1}`}>
			{#each matrixA as rows}
			{#each rows as id}
			{#if showComplexMatrix}
			<div class="complex-matrix" class:focused={inputFocus == `a-${id}`}>
			<input maxlength="6" on:focus={() => {inputFocus = `a-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validA[id]} type="text" on:input={(evt) => changeInputSingle('a', id, 0, evt)} value={valuesA[id][0]} />
			<input maxlength="6" on:focus={() => {inputFocus = `a-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validA[id]} type="text" on:input={(evt) => changeInputSingle('a', id, -1, evt)} value={-valuesA[id][1]} />
			<input maxlength="6" on:focus={() => {inputFocus = `a-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validA[id]} type="text" on:input={(evt) => changeInputSingle('a', id, 1, evt)} value={valuesA[id][1]} />
			<input maxlength="6" on:focus={() => {inputFocus = `a-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validA[id]} type="text" on:input={(evt) => changeInputSingle('a', id, 0, evt)} value={valuesA[id][0]} />
			</div>
			{:else}
			<input maxlength="6" on:focus={() => {inputFocus = `a-${id}`}} on:blur={() => {inputFocus = null}} class:focused={inputFocus == `a-${id}`} class:invalid={!validA[id]} type="text" on:input={(evt) => changeInput('a', id, evt)} value={numberToString(valuesA[id], useComplexNumbers)} />
			{/if}
			{/each}
			{/each}
		</div>
		<div class="matrix ghost" style={`--rows: ${rowsA}; --columns: ${columnsA};--subdivision: ${showComplexMatrix ? 2 : 1}`}>
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
		<div class="matrix" style={`--rows: ${rowsB}; --columns: ${columnsB};--subdivision: ${showComplexMatrix ? 2 : 1}`}>
		{#each matrixB as rows}
			{#each rows as id}
			{#if complexAsMatrix && useComplexNumbers}
			<div class="complex-matrix" class:focused={inputFocus == `b-${id}`}>
			<input maxlength="6" on:focus={() => {inputFocus = `b-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validB[id]} type="text" on:input={(evt) => changeInputSingle('b', id, 0, evt)} value={valuesB[id][0]} />
			<input maxlength="6" on:focus={() => {inputFocus = `b-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validB[id]} type="text" on:input={(evt) => changeInputSingle('b', id, -1, evt)} value={-valuesB[id][1]} />
			<input maxlength="6" on:focus={() => {inputFocus = `b-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validB[id]} type="text" on:input={(evt) => changeInputSingle('b', id, 1, evt)} value={valuesB[id][1]} />
			<input maxlength="6" on:focus={() => {inputFocus = `b-${id}`}} on:blur={() => {inputFocus = null}} class:invalid={!validB[id]} type="text" on:input={(evt) => changeInputSingle('b', id, 0, evt)} value={valuesB[id][0]} />
			</div>
			{:else}
			<input maxlength="6" on:focus={() => {inputFocus = `b-${id}`}} on:blur={() => {inputFocus = null}} class:focused={inputFocus == `b-${id}`} class:invalid={!validB[id]} type="text" on:input={(evt) => changeInput('b', id, evt)} value={numberToString(valuesB[id], useComplexNumbers)} />
			{/if}
			{/each}
			{/each}
		</div>
		<div class="matrix ghost" style={`--rows: ${rowsB}; --columns: ${columnsB};--subdivision: ${showComplexMatrix ? 2 : 1}`}>
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
		<div class="matrix-label">
			<span>Result</span>
			<span>({rowsResult} rows, {columnsResult} columns)</span>
			<span><button title="Transpose and Swap Matrix A and B to Transpose the Result" on:click={transpose}>⬔</button></span>
		</div>
	<div class="matrix" style={`--rows: ${rowsA}; --columns: ${columnsB};--subdivision: ${showComplexMatrix ? 2 : 1}`}>
	{#each result as rows, r}
		{#each rows as v, c}
		{#if complexAsMatrix && useComplexNumbers}
		<div class="complex-matrix">
		<output data-row={r} data-column={c} data-dim={0} on:click={setFocus}>{v[0]}</output>
		<output data-row={r} data-column={c} data-dim={1} on:click={setFocus}>{-v[1]}</output>
		<output data-row={r} data-column={c} data-dim={2} on:click={setFocus}>{v[1]}</output>
		<output data-row={r} data-column={c} data-dim={3} on:click={setFocus}>{v[0]}</output>
		</div>
		{:else}
		<output data-row={r} data-column={c} on:click={setFocus}>{numberToString(v, useComplexNumbers)}</output>
		{/if}
		{/each}
		{/each}
	</div>
		<div class="matrix ghost" style={`--rows: ${rowsA}; --columns: ${columnsB};--subdivision: ${showComplexMatrix ? 2 : 1}`}>
			{#if focus}
			<div class="focus" style={`grid-column: ${focus.column+2} / span 1;grid-row: ${focus.row+1} / span 1;`}>
			</div>
			{/if}
		</div>
	</div>
</div>
	
<dl>
	<dt><label for="step">Step: <span style="display: inline-block; min-width: 4em; text-align: center;">{focus ? focusNumber : '-'}/{maxFocus}</span></label></dt>
	<dd><input id="step" type="range" min="0" max={maxFocus} value={focusNumber} on:input={(evt) => setFocusStep(evt.currentTarget.value - 1)} /></dd>
	<dd>
		<button data-step={focus ? focusNumber - 1 : -1} on:click={jumpStep} disabled={!focus || focus.row*columnsB + focus.column + 1 < 1}>⮜</button>
		<button data-step={focus ? focusNumber + 1 : 0} disabled={focus && focus.row*columnsB + focus.column + 1 + 1 > maxFocus} on:click={jumpStep}>⮞</button></dd>
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
			{numberToString(valuesA[a], useComplexNumbers)}
		</div>
		<div class="operator">
			&times;
		</div>
		<div class="focus vertical">
			{numberToString(valuesA[a], useComplexNumbers)}
		</div>
	</div>
	{/each}
	
	<div class="operator">
		=
	</div>
	<div class="focus">
		{numberToString(result[focus.row][focus.column], useComplexNumbers)}
	</div>
</div>
{:else}
<p>
	Select a cell in the result matrix to see how it is calculated or use the step slider to iterate through all cells in the result matrix.
</p>
{/if}

<h3>Share this Matrix</h3>
<input class="sharelink" on:click={(evt) => {
	const e = evt.currentTarget
	e.setSelectionRange(0, e.value.length); 
	evt.preventDefault();
	setTimeout(() => {
		e.scrollLeft=0;
	})
}} type="text" readonly={true} value={shareUrl}>
</div>
