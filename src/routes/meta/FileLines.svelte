<script>
    import * as d3 from "d3";
  import { scale } from "svelte/transition";

    export let lines =[];
    let colors = d3.scaleOrdinal(d3.schemeTableau10);

    let files = [];
    $: {
        files = d3.groups(lines, d => d.file).map(([name, lines]) => {
            return {name, lines};
        });
        files = d3.sort(files, d => -d.lines.length);

    }

</script>

<dl class="files">
	{#each files as file (file.name) }
		<div>
			<dt>
				<code>{file.name}, {file.lines.length}</code>
			</dt>
			{#each file.lines as line (line.line) }
                <div class="line" style="--color: { colors(line.type) }" transition:scale={ 2}>
                </div>
            {/each}

		</div>
	{/each}
</dl>


<style>
    dl {
        display: grid;
        grid-template-columns: 1fr 1fr;
            & > div {
            grid-column: 1 / -1;
            display: grid;
            grid-template-columns: subgrid;
        }
    }

    dd {

        grid-column: 2;
        display: flex;
        flex-wrap: wrap;
        align-items: start;
        align-content: start;
        gap: .15em;
        padding-top: .6em;
        margin-left: 0;
    }

    .line {
        display: flex;
        width: .5em;
        aspect-ratio: 1;
        background: var(--color);
        border-radius: 50%;
    }



</style>