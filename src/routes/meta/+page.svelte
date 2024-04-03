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


    let hoveredIndex = -1;
    $: hoveredCommit = commits[hoveredIndex] ?? {};


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

    let xAxis, yAxis, yAxisGridlines, svg;

    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
    }

    $:{
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
            .tickFormat("").tickSize(-usableArea.width));
        } 
    
    let cursor = {x: 0, y: 0};

    $: {
	    d3.select(svg).call(d3.brush());
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
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

    <svg viewBox="0 0 {width} {height}" bind:this={svg}>

        <g class="dots">
            {#each commits as commit, index }
                <circle
                    cx={ xScale(commit.datetime) }
                    cy={ yScale(commit.hourFrac) }
                    r="5"
                    fill="steelblue"
                    on:mouseenter={evt => {hoveredIndex = index;
                    cursor = {x: evt.x, y: evt.y};}
                    }
                    on:mouseleave={evt => hoveredIndex = -1}
                />
            {/each}
        </g>
        <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
        <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
        <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
        
    </svg>
    <dl id="commit-tooltip" class="info tooltip" hidden={hoveredIndex === -1} style="top: {cursor.y}px; left: {cursor.x}px">

        <dt>Commit</dt>
        <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>
    
        <dt>Date</dt>
        <dd>{ hoveredCommit.date?.toLocaleString("en", {datetime: "%B %d, %Y"}) }</dd>
    
        <dt>Time</dt>
        <dd>{ hoveredCommit.time }</dd>
    
        <dt>Author</dt>
        <dd>{ hoveredCommit.author}</dd>

        <dt>Lines</dt>
        <dd>{ hoveredCommit.totalLines}</dd>
       
    </dl>




<style>

	svg {
		overflow: visible;
	}
    .gridlines {
	stroke-opacity: .2;
    }

    dl.info {
        display: grid;
        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }

    .info dt {
        font-weight: bold;
    }

    .tooltip {
        position: fixed;
        top: 1em;
        left: 1em;
        background-color: rgb(182, 181, 181);
        box-shadow: 2px 2px 2px rgb(87, 86, 86);
        padding: 1em;
        border-radius: 10px;
    }

    circle {

        &:hover {
            transform: scale(1.5);
            transform-origin: center;
            transform-box: fill-box;

        }
   }

</style>


