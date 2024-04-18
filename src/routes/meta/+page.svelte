<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";
    import Pie from "$lib/Pie.svelte";
    import FileLines from "./FileLines.svelte";
    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';



    let data = [];
    let commits = [];
    let colors = d3.scaleOrdinal(d3.schemeTableau10);

    let selectedCommits = [];
    let width = 1000;
    let height = 600;
    let margin = {top: 10, right: 10, bottom: 30, left: 20};
    let commitTooltip;
    let tooltipPosition = {x: 0, y: 0};
    let commitProgress = 100;

    $: timeScale = d3.scaleTime()
        .domain(d3.extent(data, d => d.date))
        .range([0, 100])
        .nice();

    $: rScale = d3.scaleSqrt()
			.domain(d3.extent(data, d => d.length))
			.range([2, 30])
            .nice();

    $: commitMaxTime = timeScale.invert(commitProgress);

    $: filteredCommits = commits.filter( (commit) => commit.datetime < commitMaxTime)
    $: filteredLines = data.filter((data) => data.datetime < commitMaxTime)

    let usableArea = {
	top: margin.top,
	right: width - margin.right,
	bottom: height - margin.bottom,
	left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;


    let hoveredIndex = -1;
    $: hoveredCommit = filteredCommits[hoveredIndex] ?? {};


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

                    // https://github.com/61040-fa23/portfolio-usonia09

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
        .domain(d3.extent(filteredLines, d => d.date))
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


    $: {
	    d3.select(svg).call(d3.brush().on("start brush end", brushed));
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    async function dotInteraction (index, evt) {
	// code will go here
        let hoveredDot = evt.target;

        if (evt.type === "mouseenter" || evt.type === "focus") {
            // dot hovered
            hoveredIndex = index;
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed", // because we use position: fixed
                middleware: [
                    offset(5), // spacing from tooltip to dot
                    autoPlacement() // see https://floating-ui.com/docs/autoplacement
                ],
            });

        }
        else if (evt.type === "mouseleave" || evt.type === "blur") {
            // dot unhovered
            hoveredIndex = -1
        } else if (evt.type === "click" || evt.type === "keyup" && evt.key === "Enter") {
            // dot clicked
            selectedCommits = [commits[index]]
        } 

    }


    function brushed (evt) {
        let brushSelection = evt.selection;
        selectedCommits = !brushSelection ? [] : filteredCommits.filter(commit => {
            let min = {x: brushSelection[0][0], y: brushSelection[0][1]};
            let max = {x: brushSelection[1][0], y: brushSelection[1][1]};
            let x = xScale(commit.date);
            let y = yScale(commit.hourFrac);

            return x >= min.x && x <= max.x && y >= min.y && y <= max.y;
        });
    }


    function isCommitSelected (commit) {
	    return selectedCommits.includes(commit);
    }


    $: hasSelection = selectedCommits.length > 0;
    $: selectedLines = (hasSelection ? selectedCommits : commits).flatMap(d => d.lines);

    $: languageBreakdown = d3.rollup(selectedLines, (L) => L.length, (l) => l.type)



</script>

<svelte:head>
    <title>Meta</title>
</svelte:head>

<section class="data_section">
    <div class="slider">
        <label for="commit_time">Show commits until:</label>
        <input type="range" id="slider" bind:value={commitProgress} min="0" max="100">

    </div>
    <time datetime="2018-07-07T20:00:00">{commitMaxTime.toLocaleString()}</time>

    <dl class="stats">
        <dt>TOTAL <abbr title="Lines of code">LOC</abbr></dt>
        <dd>{filteredLines.length}</dd>
        <dt>COMMITS</dt>
        <dd>{filteredCommits.length}</dd>
        <dt>AVERAGE LINE LENGTH</dt>
        <dd>{d3.mean(filteredLines, d => d.length)}</dd>
        <dt>LONGEST LINE</dt>
        <dd>{d3.max(filteredLines, d => d.length)}</dd>
        <dt>MAX LINES</dt>
        <dd>{d3.max(filteredLines, d => d.line)}</dd>
    </dl>
    
</section>

    <svg viewBox="0 0 {width} {height}" bind:this={svg}>

        <g class="dots" style="width: {100}">
            {#each filteredCommits as commit, index (commit.id) }
                <circle
                    cx={ xScale(commit.datetime) }
                    cy={ yScale(commit.hourFrac) }
                    r= {rScale(commit.totalLines)}
                    fill="steelblue"
                    on:mouseenter={evt => dotInteraction(index, evt)}
                    on:mouseleave={evt => dotInteraction(index, evt)}
                    tabindex="0"
                    aria-describedby="commit-tooltip"
                    aria-haspopup="true"
                    on:click={evt => dotInteraction(index, evt)}
                    fill-opacity="50%"
                    style="--r: {rScale(commit.totalLines)}; transition: all 200ms, r calc(var(--r)*100ms)"
                />
            {/each}

            {#each selectedCommits as commit}
                <circle
                cx={ xScale(commit.datetime) }
                cy={ yScale(commit.hourFrac) }
                r="5"
                fill="red"/>

            {/each}
        </g>
        <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
        <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
        <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    </svg>
    <dl id="commit-tooltip" class="info tooltip" hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"  bind:this={commitTooltip} role="tooltip">

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
    <p>{hasSelection ? selectedCommits.length : "No"} commits selected</p>

    <!-- <dl class="stats">

        {#each languageBreakdown as [language, lines] }
                <dt>{language}</dt>
                <dd>{lines} lines ({d3.format(".0%")(lines/selectedLines.length)})</dd>

            
        {/each}
    </dl> -->
    <Pie data={Array.from(languageBreakdown).map(([language, lines]) => ({label: language, value: lines}))} colors={colors} />
    <h2>Codebase evaluation</h2>
    <FileLines lines={filteredLines} colors={colors}/>




<style>

	svg {
		overflow: visible;
        margin: 10px;
    
	}
    .dots {
        width: 100%;
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


    .slider {
        display: grid;
        grid-template-columns: 1fr 4fr;

    }

    time {
        display: flex;
        justify-content: flex-end;
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
        &:selection {
            color: red;
            fill: red;
            background-color: aquamarine;
        }
        @starting-style {
            r: 0;
        }
   }

</style>

