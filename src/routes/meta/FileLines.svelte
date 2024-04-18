<script>
    import * as d3 from "d3";
  import { scale } from "svelte/transition";
  import { flip as originalFlip } from "svelte/animate";



    export let lines =[];
    export let colors = [];

    let files = [];
    $: {
        files = d3.groups(lines, d => d.file).map(([name, lines]) => {
            return {name, lines};
        });
        files = d3.sort(files, d => -d.lines.length);

    }
    function getFlip () {
        return originalFlip;
    }
    $: flip = getFlip(files);


</script>

<dl class="files">
	{#each files as file (file.name) }
		<div>
			<dt>
				<code>{file.name}, {file.lines.length}</code>
			</dt>
            <dd>
                {#each file.lines as line (line.line) }
                    <div class="line" style="--color: { colors(line.type) }" transition:scale={ 2} animate:flip>
                    </div>
                {/each}
            </dd>

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
            /* background: hsl(0 0% 100% / 90%); */
	        /* box-shadow: 0 0 .2em .2em hsl(0 0% 100% / 90%); */
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