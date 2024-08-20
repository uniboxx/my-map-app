<script>
  const menuItems = ['Home', 'About', 'Contact', 'More'];
  let current = $state(menuItems.at(0));
</script>

<nav>
  {#each menuItems as item (item)}
    <button class:active={current === item} onclick={() => (current = item)}
      >{item}</button>
  {/each}
</nav>

<style lang="scss">
  nav {
    display: flex;
    gap: 2rem;
  }
  button {
    padding: 0.5rem 1rem;
    border-radius: 10px;
    cursor: pointer;
  }
  .active {
    background-color: #ff3e00;
    color: #fff;
  }
</style>
