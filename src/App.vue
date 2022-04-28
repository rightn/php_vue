<template>
  <div class="container">

    <nav>
      <router-link to="/">Home</router-link> |
      <router-link to="/create">Create</router-link> |          
      <router-link to="/list">List</router-link> |
      <router-link to="/about">About</router-link>
    </nav>
    
    <router-view />
  </div>
  
  
</template>

<script>
</script>

<style>
#app {
}
</style>
