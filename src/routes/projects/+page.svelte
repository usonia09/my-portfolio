<script>
  import projects from '$lib/projects.json';
  import Project from '../../lib/Project.svelte';
  import Pie from '../../lib/Pie.svelte';
  import * as d3 from 'd3';


  let query = "";

  let filteredProjects;
  $: filteredProjects = projects.filter(project => {
	let values = Object.values(project).join("\n").toLowerCase();
	return values.includes(query.toLowerCase());
  });

  // Make sure the variable definition is *outside* the block
  let pieData;

  $: {
    // Initialize to an empty object every time this runs
    pieData = {};

    // Calculate rolledData and pieData based on filteredProjects here
    let rolledData = d3.rollups(filteredProjects, v => v.length, d => d.year);
    pieData = rolledData.map(([year, count]) => {
    return { value: count, label: year };
    });
  }
</script>

<svelte:head>
	<title>Projects</title>
</svelte:head>
<h1>{projects.length} Projects</h1>
<Pie data={pieData}/>
<input type="search" bind:value={query}
       aria-label="Search projects" placeholder="🔍 Search projects…" />
<div class="projects">
  {#each filteredProjects as p}
  <Project info={p} hLevel=3/>
  {/each}
</div>