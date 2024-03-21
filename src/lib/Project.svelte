<script>
    export let info = {}
    export let hLevel = 2
</script>
<article>
    <svelte:element this={"h"+hLevel}>{info.title}</svelte:element>
    <img src={info.image} alt="">
    <div class="project-info">
        <p>{info.description}</p>
        <p>{"C. "+info.year}</p>
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