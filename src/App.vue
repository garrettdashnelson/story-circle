
<template>
<div id="contains-all" :class="mode === 'initial' ? 'dark-bg' : 'light-bg'">
  <header v-if="headerOn">
    <button class="button is-small is-primary mr-2" @click="resetToInitial">⤴ Back</button> <div class="header-title">Storycircle</div>  
  </header>

  <div id="main-area">

  <mode-controller @changeMode="changeMode" v-if="mode === 'initial'"></mode-controller>

  <new-game @joinGame="joinGame" v-if="mode==='create-new'"></new-game>

  <enter-game @startGame="startGame" v-if="mode==='enter-existing'" :initialCode="gameCode"></enter-game>

  <gameplay v-if="mode==='playing'" ></gameplay>

  <section v-if="mode==='instructions'" class="py-4 instructions">
    <div class="container is-fluid has-text-centered">
          
      <p><span class="storycircle">Storycircle</span> is a pretty simple game. It works best the more deranged your mind is. The goal is to create an absurd story where each player only knows what happened in the last line—sort of similar to the game Telephone.</p>
      <p>One player should set up a new game by clicking "Start a New Game" from the home screen. Enter each player's name, and how many rounds (lines) you want your story to be. You'll get a random game code which you should share with everyone.</p>
      <p>Have everyone "Join a Game" using the code you created. On each round, you write a line of a story. You can enter text up until the bar runs out. Don't worry if you haven't finished a sentence: the point is to get cut off mid-thought!</p>
      <p>When you submit, the story will get passed on to the next player, and you'll receive the last line of a story from the person on the other side of the circle. If they haven't submitted yet, you'll have to wait until their next line is ready.</p>
      <p>Once you've finished all the rounds, you'll have a bizarre story to read. Good luck narrating with a straight face!</p>
    <p class="is-size-7">Storycircle was made during the 2020 quarantine by <a href="http://people.matinic.us/garrett">Garrett Dash Nelson</a> (<a href="https://twitter.com/en_dash">@en_dash</a>) as a fun little project for family. If you enjoy it, please let me know!</p>  </div>
  </section>

  </div>
</div>
</template>

<script lang="ts">
import Vue from "vue";
import ModeController from './ModeController.vue'
import NewGame from './NewGame.vue'
import EnterGame from './EnterGame.vue'
import Gameplay from './Gameplay.vue'

import faunadb, { query as q } from "faunadb";

var client = new faunadb.Client({ secret: 'fnAD4dsbeZACBrX3LE8aTjpoBmxbnhDMaBrkYFEm' });

export default Vue.extend({

    components: {
      modeController: ModeController,
      newGame: NewGame,
      enterGame: EnterGame,
      gameplay: Gameplay
    },

    data: function() {
      return {
        mode: 'initial',
        gameRef: null,
        chosenPlayer: "",
        gameCode: "",
        gameData: null
      }
    },

    computed: {
      headerOn: function() { 
        return this.mode === 'initial' ? false : true;
       }
    },

    methods: {

      changeMode: function(m) {
        this.mode = m;
      },

      resetToInitial: function() {
        this.mode = 'initial';
      },

      joinGame: function(c) {
        this.gameCode = c;
        this.changeMode('enter-existing');
      },

      startGame: function(r,p) { 
        this.gameRef = r;
        this.chosenPlayer = p;
        this.refreshData();
             
      },
      
      refreshData: function(){
        client.query(
            q.Get(
            q.Ref(q.Collection('games'), this.gameRef),
            ))
            .then((r) => {
              this.gameData = r.data;
              this.mode = 'playing';
            })
            .catch((error) => {});
      }

    },

  });
</script>


<style lang="stylus">

@import 'vars.styl'

html
  height: 100%;

body
  height: 100%;


#contains-all
  height: 100%;
  display: flex;
  flex-direction: column
  
  &.dark-bg
    background: rgb(131,58,180);
    background: linear-gradient(144deg, rgba(131,58,180,1) 0%, rgba(253,29,29,1) 50%, rgba(252,176,69,1) 100%);

  &.light-bg
    background: rgb(238,174,202);
    background: linear-gradient(215deg, rgba(238,174,202,0.335171568627451) 0%, rgba(148,233,210,0.4051995798319328) 100%);

  header
    padding: 10px 24px;

    .header-title
      display: inline-block
      font-family: 'Sirin Stencil'
      color: bluish
      font-size: 1.4rem
      text-transform: uppercase
      letter-spacing: 2px
      position: relative
      bottom: 3px



  #main-area
    flex-grow: 1
    overflow: scroll


.instructions
  .storycircle
    font-family: 'Sirin Stencil'
    letter-spacing: 1px
    text-transform: uppercase
  p
    margin-bottom: 10px

</style>
