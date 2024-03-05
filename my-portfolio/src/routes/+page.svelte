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
