<template>
  <div id="app">
    <header>
      <img alt="Vue logo" class="logo" src="@/assets/logo.png" width="125" height="125" />
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
        <RouterLink to="/life">Life</RouterLink>
        <RouterLink to="/draw">Draw</RouterLink>
      </nav>
    </header>
    <router-view/>
  </div>
</template>

<script setup>
import { RouterLink, RouterView} from 'vue-router'
</script>


<style scoped>
header {
  display: grid;
  grid-template-columns: 1fr 8fr;
  line-height: 1.5;
  border: 1px dashed red;
}

.logo {
  display: inline;
  margin: 0 auto 2rem;
  border: 1px solid purple;
}

nav {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: grid;
    grid-template-columns: 1fr;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: grid;
    grid-template-columns: 1fr;
    border: 2px solid blue;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;
    padding: 1rem 0;
    margin-top: 1rem;
  }
}

</style>