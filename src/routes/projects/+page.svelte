<script>
  import projects from '$lib/projects.json';
  import Project from '../../lib/Project.svelte';
  import Pie from '../../lib/Pie.svelte';
  import * as d3 from 'd3';

  let rolledData = d3.rollups(projects, v => v.length, d => d.year);
  let pieData = rolledData.map(([year, count]) => {
	return { value: count, label: year };
});

</script>
<svelte:head>
	<title>Projects</title>
</svelte:head>
<h1>{projects.length} Projects</h1>
<Pie data={pieData}/>
<div class="projects">
{#each projects as p}
<Project info={p} hLevel=3/>
{/each}
</div>