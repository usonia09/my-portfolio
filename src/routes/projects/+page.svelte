<script>
  import projects from '$lib/projects.json';
  import Project from '../../lib/Project.svelte';
  import Pie from '../../lib/Pie.svelte';
</script>
<svelte:head>
	<title>Projects</title>
</svelte:head>
<h1>{projects.length} Projects</h1>
<Pie/>
<div class="projects">
{#each projects as p}
<Project info={p} hLevel=3/>
{/each}
</div>