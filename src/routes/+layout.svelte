<nav>
	{#each pages as p }
    <a href={p.url}>{p.title}</a>
	{/each}
</nav> <slot />
<script>
  import { linear } from "svelte/easing";

    let pages = [
    { url: "./", title: "Home" },
    {
        url: "./projects",
        title: "Projects",
    },
    {
        url: "./Assignments",
        title: "Assignments",
    },
    {
        url: "./contact",
        title: "Contact Me",
    },
    {
        url: "./resume",
        title: "My Resume",
    },
    ];

</script>


