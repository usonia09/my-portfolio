<script>
    import * as d3 from 'd3';

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    export let data = [];
    let colors = d3.scaleOrdinal(d3.schemeTableau10);
    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData;
    let arcs;
    let pieData;
    $: {
        pieData = data.map(d => ({...d}));
    };

 
    $: {
        arcData = sliceGenerator(pieData);
        arcs = arcData.map(d => arcGenerator(d));
        pieData = pieData.map((d, i) => ({...d, ...arcData[i], arc: arcs[i]}));
    };

    
    export let selectedIndex = -1;


</script>

<div class="container">
    <svg viewBox="-50 -50 100 100">
        {#each pieData as d, index}
        <path d={d.arc} fill={ colors(d.label) }
              class:selected={selectedIndex === index}
              on:click={e => selectedIndex = selectedIndex === index ? -1 : index} />
        {/each}
    
    </svg>
    <ul class="legend">
        {#each pieData as d, index}
            <li style="--color: { colors(index) }" class:selected={selectedIndex === index}>
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>    
</div>



<style>
.selected {
	--color: oklch(50% 0.1 145) !important;

	&:is(path) {
		fill: var(--color);
	}
}

svg {
	max-width: 20em;
	margin-block: 2em;
	/* Do not clip shapes outside the viewBox */
	overflow: visible;
} 

svg:has(path:hover) {
	path:not(:hover) {
		opacity: 50%;
	}
}

ul {
    border: 1px solid rgb(233, 229, 229);
    padding: 10px 5px;
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
    flex: 1;
}

.container {
    display: flex;
    align-items: center;
    gap: 2em;
}

.legend li {
    display: flex;
    align-items: center;
    gap: 10px;
    
}

.legend li span {
    width: 15px;
    height: 15px;
    border-radius: 50%;
    background-color: var(--color);

}
</style>

