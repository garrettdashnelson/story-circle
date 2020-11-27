
<template>

<section class="pt-6">
  <div class="container has-text-centered">

    <h1 class="title is-1"><strong>Storycircle</strong></h1>
    <button class="button is-primary my-3" @click="$emit('changeMode','create-new')">🌞 Start a New Game</button><br>
    <button class="button is-primary my-3" @click="$emit('changeMode','enter-existing')">👪 Join a Game</button><br>
    <button class="button is-primary my-3" @click="$emit('changeMode','instructions')">🧐 How to Play</button>
  </div>
</section>
</template>

<script lang="ts">

import Vue from "vue";

  export default Vue.extend({

    components: {
    },

    data: function() {
      return {
      }
    },

    computed: {
    },

    methods: {
    }
  });
</script>

<style scoped lang="stylus">

button
  font-size: 1.5rem
  font-weight: bold
  box-shadow: 4px 4px 0px yellow;

h1
  strong
    font-family: 'Sirin Stencil'
    border: 0;
    font-size: 3.1rem
    color: white;
    text-transform: uppercase
    letter-spacing: 2px;
    font-weight: normal

      
</style>
