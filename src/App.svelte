<script>
	let array32 = Array(32).fill(0);
	let a = 5;
	let b = 3;

	let isTwosComplement = true;

	// signed right shift returns unsigned 32-bit int!!
	$: purify = isTwosComplement ? (x => x | 0) : (x => x >>> 0);

	$: a = purify(a);
	$: b = purify(b);

	let ops = [
		{name: 'add (+)',   op: (x, y) => x + y},
		{name: 'sub (-)',   op: (x, y) => x - y},
		{name: 'and (&)',   op: (x, y) => x & y},
		{name: 'or (|)',    op: (x, y) => x | y},
		{name: 'xor (^)',   op: (x, y) => x ^ y},
		{name: 'sll (<<)',  op: (x, y) => x << y, shift: true },
		{name: 'srl (>>>)', op: (x, y) => x >>> y, shift: true },
		{name: 'sra (>>)',  op: (x, y) => x >> y, shift: true },
		{name: 'slt',  op: (x, y) => (x|0) < (y|0) ? 1 : 0, comparison: true, twosComplement: true},
		{name: 'sltu',  op: (x, y) => (x>>>0) < (y>>>0) ? 1 : 0, comparison: true, twosComplement: false},
	];
	let selectedOp = ops[0];

	const bitify = x => [...Array(32).keys()].map(i => (x & (1 << (31 - i)) ? 1 : 0));

	const hexify = x => {
		let s = (x >>> 0).toString(16).toUpperCase();
		return '0x' + '0'.repeat(8 - s.length) + s;
	};

	const flipA = (i) => { a = purify(a ^ (1 << i)); };
	const flipB = (i) => { b = purify(b ^ (1 << i)); };

	$: inputMin = isTwosComplement ? (1<<31) : 0;
	$: inputMax = isTwosComplement ? ~(1<<31) : (Math.pow(2, 32) - 1);

	$: abits = bitify(a);
	$: bbits = bitify(b);

	$: res = purify(selectedOp.op(a, b));

	$: resbits = bitify(res);
</script>

<main>
	<h1>Binary Operations</h1>
	<p>The set of operations and mnemonics are from base RISC-V. Click on a bit to toggle it.</p>
	<ul>
		<li>
			<label>
				<input type=radio bind:group={isTwosComplement} value={true}> Numbers are 32-bit two's complement integers
			</label>
		</li>
		<li>
			<label>
				<input type=radio bind:group={isTwosComplement} value={false}> Numbers are 32-bit unsigned integers
			</label>
		</li>
	</ul>
	<table>
		<tr>
			<td></td>
			<td>0b</td>
			{#each abits as bit, i}
				<td class="bit" on:click={() => flipA(31 - i)}>{bit}</td>
			{/each}
			<td>=</td><td><input type="number" bind:value={a} min={inputMin} max={inputMax}></td>
			<td>=</td><td>{hexify(a)}</td>
		</tr>
		<tr>
			<td><select bind:value={selectedOp}>
					{#each ops as op}
						<option value={op}>{op.name}</option>
					{/each}
			</select></td>
			<td>0b</td>
			{#each bbits as bit, i}
				<td class="bit" on:click={() => flipB(31 - i)}>{bit}</td>
			{/each}
			<td>=</td><td><input type="number" bind:value={b} min={inputMin} max={inputMax}></td>
			<td>=</td><td>{hexify(b)}</td>
			<td class="effective">
				{#if selectedOp.shift && !(0 <= b && b < 32)}
					(effectively {b & 31})
				{/if}
			</td>
		</tr>
		<tr class="result">
			<td>result:</td>
			<td>0b</td>
			{#each resbits as bit, i}
				<td>{bit}</td>
			{/each}
			<td>=</td><td class="result-dec">{res}</td>
			<td>=</td><td>{hexify(res)}</td>
		</tr>
	</table>
	{#if selectedOp.comparison && (selectedOp.twosComplement != isTwosComplement)}
		<p><strong>WARNING</strong>: The display setting is different from the signedness of the comparison! Comparing the displayed decimal numbers may not give the same result as what's computed.</p>
	{/if}
</main>

<style>
	table {
		font-family: "Source Code Pro", monospace;
	}

	table {
		border-collapse: collapse;
	}
	.bit { cursor: pointer; }
	.bit:hover { background-color: #ffa; }
	.result td { padding-top: 1em; border-top: 1px solid black; }
	.effective { font-size: 85%; }
	input, .result-dec {
		text-align: right;
	}
</style>
