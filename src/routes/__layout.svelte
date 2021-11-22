<script lang='ts'>
  import '../app.css';
  import GithubCorner from '$lib/GithubCorner.svelte';
  import Info from '$lib/data/info.json';

  const {
    name
  } = Info;
</script>

<GithubCorner />
<h1>
  {name}
</h1>
<hr>

<main>
  <slot />
</main>


<style>
</style>
