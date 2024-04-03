<nav>
	{#each pages as p }
    <a href={p.url}>{p.title}</a>
	{/each}
</nav> 

<label class="color-scheme">
		Theme:
 		<select bind:value={colorScheme}>
            <option value="light">light</option>
             <option value="light dark">light dark</option>
             <option value="dark">dark</option>
 		</select>
</label>
<slot />

<script>
    let root = globalThis?.document?.documentElement;
    let localStorage = globalThis.localStorage ?? {};
    let pages = [
    { url: "./", title: "Home" },
    {
        url: "./projects",
        title: "Projects",
    },
    {
        url: "./contact",
        title: "Contact Me",
    },
    {
        url: "./resume",
        title: "My Resume",
    },
    {
        url: "./meta",
        title: "Meta",
    }
    ];
    let colorScheme = localStorage.colorScheme ?? "light dark";
    $: root?.style.setProperty("color-scheme", colorScheme);
    $: localStorage.colorScheme = colorScheme

</script>

<style>
    .color-scheme {
        position: absolute;
        top: 1rem;
        right: 1rem;
        display: block;
    }
</style>


