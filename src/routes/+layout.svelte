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
{#await fetch("https://api.github.com/users/usonia09") }
	<p>Loading...</p>
{:then response}
	{#await response.json()}
		<p>Decoding...</p>
	{:then data}
        <section>
            <h2>My GitHub Stats</h2>
            <dl>
                <dt>Number of Public Repos: </dt>
                <dd>{data.public_repos}</dd>

                <dt>Followers:</dt>
                <dd>{data.followers}</dd>

                <dt>Following:</dt>
                <dd>{data.following}</dd>
            </dl>
        </section>
	{:catch error}
		<p class="error">
			Something went wrong: {error.message}
		</p>
	{/await}
{:catch error}
	<p class="error">
		Something went wrong: {error.message}
	</p>
{/await}

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
    dl {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
    }
    dt {
        grid-row: 1;
    }
    dd {
        grid-row: 2;
    }
</style>


