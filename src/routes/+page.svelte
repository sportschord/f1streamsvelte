<script>
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
	import teamcols from '../data/teamcols.json';
	import { fade } from 'svelte/transition';
	import logo from '$lib/images/SC Logo.png';

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
		});

		d3.csv('/f1streamsorted.csv').then((d) => {
			data = d;
		});
	});

	$: data;
	$: f1keys;

	var season = (d) => d.season;
	$: seasons = Array.from(new Set(data.map(season)));
	$: keys = f1keys.map((d) => d.team);

	var maxdom = 2700 / 2;

	$: xAxis = d3
		.scaleLinear()
		.domain(d3.extent(seasons))
		.range([margin.left*3, width - margin.right]);

	$: xAxisTicks = xAxis.ticks(10); // Number of ticks (adjust as needed)

	$: xScale = d3
		.scaleLinear()
		.domain(d3.extent(seasons))
		.range([margin.left, width - margin.right]);

	$: yScale = d3
		.scaleLinear()
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
    <div style="padding: 20px;">

        <svg {width} {height} viewBox="0 0 {width} {height}" style:max-width="100%" style:height="auto">
            {#each xAxisTicks as tick}
                <line
                    x1={xAxis(tick)}
                    x2={xAxis(tick)}
                    y1={margin.top}
                    y2={height - margin.bottom}
                    stroke="rgba(255, 255, 255, 0.2)"
                    stroke-width=2
                />
                <text
          x={xAxis(tick)}
          y={height - margin.bottom + 20} 
          font-size="14px"
          text-anchor="middle" 
          fill="white"
        >
          {tick}
        </text>
            {/each}
            {#each stackedData as s}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
    
                <path
                    in:fade={{ duration: 500 }}
                    id={s.key}
                    fill={teamcols[s.key]}
                    opacity={hovered ? (highlightedKey === s.key ? 1 : 0.1) : 1}
                    stroke={hovered ? (highlightedKey === s.key ? 'white' : 'none') : 'grey'}
                    stroke-width="0.2"
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
    </div>

	<div class="labels">
		{#each keys as k, i}
			<div class="labsize">
				<!-- svelte-ignore a11y-click-events-have-key-events -->
				<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
				<p
					style="color: {highlightedKey === k ? teamcols[k] : 'white'}; 
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
	<div class="footer">
		<a href="https://www.sportschord.com">Designed by SportsChord</a>

		<a class="logo" href="https://www.sportschord.com"><img src={logo} width="30" alt="sc" /></a>
	</div>
</div>

<style>
	.labels {
		margin: 20px;
		display: flex;
		flex-wrap: wrap;
		gap: 20px;
		font-size: 14px;
		justify-content: center;
	}

	.footer {
		margin: 40px;
		font-size: 10px;
		gap: 20px;
		align-items: center;
		display: flex;
		justify-content: center;
	}

	a {
		color: rgb(223, 223, 223);
		text-decoration: none;
	}

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
