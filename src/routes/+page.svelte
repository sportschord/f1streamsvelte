<script>
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
	import teamcols from '../data/teamcols.json';
	import { draw, fade, fly } from 'svelte/transition';
	// import grouped from '../data/f1streamsorted.csv'
	// import f1keysdata from '../data/keys.csv'

	let f1keys = [];
	let data = [];
	let width = 3000;
	let height = width / 4;
	let stackedData = [];
    const margin = { 
		top: 20,
		right: 20,
		bottom: 20,
		left: 20	
	};

	onMount(() => {
		d3.csv('/keys.csv').then((d) => {
			f1keys = d;
			// Here, you can process the data and use it for visualization
		});

		d3.csv('/f1streamsorted.csv').then((d) => {
			data = d;
			// Here, you can process the data and use it for visualization
		});
	});

	$: data;
	$: f1keys;

	var season = (d) => d.season;
	$: seasons = Array.from(new Set(data.map(season)));
	$: keys = f1keys.map((d) => d.team);

	var maxdom = 2700 / 2;

	$: xScale = d3.scaleLinear()
    .domain(d3.extent(seasons))
    .range([margin.left, width - margin.right]);

	$: yScale = d3.scaleLinear()
    .domain([-maxdom, maxdom])
    .rangeRound([height - margin.bottom, margin.top]);

	$: stackedData = d3
		.stack()
		.keys(keys)
		.offset(d3.stackOffsetSilhouette)
		.order(d3.stackOrderReverse)(data);

	$: area = d3
		.area()
		.x((d) => xScale(d.data.season))
		.y0((d) => yScale(d[0]))
		.y1((d) => yScale(d[1]))
		.curve(d3.curveCardinalOpen);

	let highlightedKey = null;
	let hovered = false;

	function highlightKey(key) {
		hovered = true;
		highlightedKey = key;
	}

	function resetKeyHighlights() {
		hovered = false;
		highlightedKey = null;
	}
</script>

<div>
	<h1>FORMULA 1 CONSTRUCTORS</h1>

	<svg {width} {height} viewBox="0 0 {width} {height}" style:max-width="100%" style:height="auto">
		{#each stackedData as s}
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
			<path
				transition:fade={{ duration: 300 }}
				id={s.key}
				fill={teamcols[s.key]}
				opacity={hovered ? (highlightedKey === s.key ? 1 : 0.2) : 1}
				stroke="grey"
				stroke-width="0.15"
				d={area(s)}
				role="graphics-symbol"
				on:click={() => highlightKey(s.key)}
				on:mouseover={() => highlightKey(s.key)}
				on:focus={() => highlightKey(s.key)}
				on:mouseout={resetKeyHighlights}
				on:blur={resetKeyHighlights}
			/>
		{/each}
	</svg>
	<div class="labels">
		{#each keys as k, i}
			<div class="labsize">
				<!-- svelte-ignore a11y-click-events-have-key-events -->
				<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
				<p
					style="color: {highlightedKey === k
						? 
							 teamcols[k]
						: 'white'}; 
          opacity: {hovered ? (highlightedKey === k ? 1 : 0.2) : 1};"
					on:click={() => highlightKey(k)}
					on:mouseover={() => highlightKey(k)}
					on:focus={() => highlightKey(k)}
					on:mouseout={resetKeyHighlights}
					on:blur={resetKeyHighlights}
				>
					{k}
				</p>
			</div>
		{/each}
	</div>
</div>

<style>
	.labels {
		margin: 20px;
		display: flex;
		flex-wrap: wrap;
		gap: 20px;
        justify-content: center;
	}

	/* .labsize {
        width: 80px;
        text-align: center;

    } */

	h1 {
		text-align: center;
		font-weight: 400;
        margin: 40px;
	}

	p {
		cursor: pointer;
        margin: 5px;
	}
</style>
