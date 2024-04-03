<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    let data = [];
    let commits = [];
    let width = 1000;
    let height = 600;
    let margin = {top: 10, right: 10, bottom: 30, left: 20};

    let usableArea = {
	top: margin.top,
	right: width - margin.right,
	bottom: height - margin.bottom,
	left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;



    onMount(async () => {
        data = await d3.csv("loc.csv", row => ({
            ...row,
            line: Number(row.line), // or just +row.line
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));

        commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
                    let first = lines[0];
                    let {author, date, time, timezone, datetime} = first;
                    let ret = {
                        id: commit,
                        url: "https://github.com/vis-society/lab-7/commit/" + commit,
                        author, date, time, timezone, datetime,
                        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                        totalLines: lines.length
                    };

                    // Like ret.lines = lines
                    // but non-enumerable so it doesn’t show up in JSON.stringify
                    Object.defineProperty(ret, "lines", {
                        value: lines,
                        configurable: true,
                        writable: true,
                        enumerable: false,
                    });

                    return ret;
                });
    });

    $: xScale = d3.scaleTime()
        .domain(d3.extent(data, d => d.date))
		.range([usableArea.left, usableArea.right])
        .nice();
	
	$: yScale = d3.scaleLinear()
			.domain([0, 24])
			.range([usableArea.bottom, usableArea.top]);

    let xAxis, yAxis, yAxisGridlines;

    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
    }

    $:{
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
            .tickFormat("").tickSize(-usableArea.width));
        } 
    
    

</script>

<svelte:head>
    <title>Meta</title>
</svelte:head>

<section class="data_section">
    <h2>Summary Stats</h2>
    <dl class="stats">
        <dt>TOTAL <abbr title="Lines of code">LOC</abbr></dt>
        <dd>{data.length}</dd>
        <dt>COMMITS</dt>
        <dd>{commits.length}</dd>
        <dt>AVERAGE LINE LENGTH</dt>
        <dd>{d3.mean(data, d => d.length)}</dd>
        <dt>LONGEST LINE</dt>
        <dd>{d3.max(data, d => d.length)}</dd>
        <dt>MAX LINES</dt>
        <dd>{d3.max(data, d => d.line)}</dd>
    </dl>
    
</section>

<svg viewBox="0 0 {width} {height}">

    <g class="dots">
        {#each commits as commit, index }
            <circle
                cx={ xScale(commit.datetime) }
                cy={ yScale(commit.hourFrac) }
                r="5"
                fill="steelblue"
            />
        {/each}
    </g>
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

        
</svg>

<style>
	svg {
		overflow: visible;
	}
    .gridlines {
	stroke-opacity: .2;
    }

</style>


