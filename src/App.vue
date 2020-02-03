<template>
  <div id="app">
    <div class="navbar">
      <app-menu min-width="300" position-x="center" has-arrow open-on-hover close-on-click>
        <template #activator>
          <button>Click Me!</button>
        </template>

        <ul>
          <li v-for="i in Array(10).fill(0).map((_, b) => b + 1)" :key="i">
            <a href="#">Item {{ i }}</a>
          </li>
        </ul>
      </app-menu>

      <app-menu min-width="300" position="right" has-arrow>
        <template #activator>
          <button>Click Me!</button>
        </template>

        <ul>
          <li v-for="i in Array(10).fill(0).map((_, b) => b + 1)" :key="i">
            <a href="#">Item {{ i }}</a>
          </li>
        </ul>
      </app-menu>
    </div>
  </div>
</template>

<script>
import AppMenu from './components/AppMenu.vue';

export default {
    components: {
        AppMenu,
    },
};
</script>

<style lang="scss">
#app {
    height: 100vh;
    max-width: 600px;
    margin: 0 auto;
}

.navbar {
    display: flex;
    width: 100%;
    justify-content: space-between;
}

button {
    border: 0;
    color: white;
    background-color: #38b2ac;
    min-height: 40px;
    padding: 0 20px;
    border-radius: 4px;
    cursor: pointer;
}

ul {
    list-style-type: none;
    padding-left: 1rem;
}

li {
    padding: 0.5rem 0;
}

p {
    font-size: 2rem;
}
</style>