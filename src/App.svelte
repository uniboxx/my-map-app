<script lang="ts">
  import Map from './lib/Map.svelte';
  import Nav from './lib/Nav.svelte';
</script>

<!-- <Nav /> -->
<div class="container">
  <h1>My Map App</h1>
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

  h1 {
    text-align: center;
    font-size: 3rem;
    font-weight: 300;
    color: #ff3e00;
    transform: skew(-20deg);
    box-shadow: 0 3px 3px rgba(#ff3e00, 0.2);
    padding: 0.5rem 1rem;
    width: 120%;
  }
</style>
