<script>
    import * as d3 from "d3";

    export let lines =[];
    let files = [];
    $: {
        files = d3.groups(lines, d => d.file).map(([name, lines]) => {
            return {name, lines};
        });
    }

    console.log("files:", files)

</script>

<dl class="files">
	{#each files as file (file.name) }
		<div>
			<dt>
				<code>{file.name}</code>
			</dt>
			<dd>{file.lines.length} lines</dd>
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


</style>