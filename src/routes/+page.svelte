<script>
  import projects from '$lib/projects.json';
  import Project from '../lib/Project.svelte';
</script>
<h1>Sonia Uwase</h1>
<img
  src="./images/piece_of_art.jpg"
  alt="An artistic picture of a lady"
  width="300px"
  height="auto"
/>

<p>
  Hi, I'm Sonia Uwase from Rwanda. I am super excited for learning more
  about data visualization!
</p>

<h2>Latest projects</h2>
<div class="projects">
  {#each projects.slice(0,3) as p}
  <Project info={p} hLevel={3}/>
  {/each}
</div>

{#await fetch("https://api.github.com/users/usonia09") }
	<p>Loading...</p>
{:then response}
	{#await response.json()}
		<p>Decoding...</p>
	{:then data}
        <section class="data_section">
            <h2>My GitHub Stats</h2>
            <dl class="stats">
                <dt>Number of Public Repos</dt>
                <dd>{data.public_repos}</dd>

                <dt>Followers</dt>
                <dd>{data.followers}</dd>

                <dt>Following</dt>
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

<!-- <style>
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
</style> -->