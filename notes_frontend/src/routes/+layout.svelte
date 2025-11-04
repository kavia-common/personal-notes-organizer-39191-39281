<script lang="ts">
  import '../app.css';
  let { children } = $props();
</script>

<main>
  {@render children()}
</main>

<style>
  main {
    min-height: 100vh;
  }
</style>
