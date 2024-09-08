<template>
  <div id="app">
    <nav>
      <router-link to="/">Landing Page</router-link> |
      <router-link to="/weekly-data">Weekly Data</router-link>
    </nav>
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App'
};
</script>

<style>
nav {
  margin: 20px;
  text-align: center;
}
nav a {
  margin: 0 10px;
  color: #42b983;
}
</style>
