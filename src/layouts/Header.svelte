<header>
  <h1>My Map App</h1>
  <p><em>by <a href="">Unibox</a></em></p>
</header>

<style lang="scss">
  @use '../styles/vars';

  header {
    width: 120%;
    box-shadow: 0 3px 3px rgba(#ff3e00, 0.2);
  }

  h1 {
    text-align: center;
    font-size: 3rem;
    font-weight: 300;
    color: #ff3e00;
    transform: skew(-20deg);
    padding: 0.5rem 1rem;
  }

  p {
    text-align: end;
    margin-right: 10%;
    color: #ff3e00;
    a {
      color: inherit;
    }
  }
</style>
