<script lang="ts">
  import Header from './layouts/Header.svelte';
  import Map from './lib/Map.svelte';
  import Nav from './lib/Nav.svelte';
</script>

<!-- <Nav /> -->
<div class="container">
  <Header />
  <Map />
</div>

<style lang="scss">
  @use './styles/vars';

  .container {
    min-height: 100svh;
    margin: 0 auto;
    padding: 0.5rem;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    overflow-x: hidden;
    @media screen and (min-width: vars.$sm) {
      max-width: 75%;
    }
    @media screen and (min-width: vars.$lg) {
      max-width: 50%;
    }
  }
</style>
