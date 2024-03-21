<script>
    export let info = {}
    export let hLevel = 2
    let p = info
</script>
<article>
    <svelte:element this={"h"+hLevel}>{p.title}</svelte:element>
    <img src={p.image} alt="">
    <div class="project-info">
        <p>{p.description}</p>
        <p>{"C. "+p.year}</p>
    </div>
</article>

<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
        margin: 0;
    }
    div {
        font-family: Baskerville;
        font-variant-numeric: oldstyle-nums;
    }
</style>