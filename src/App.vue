
<template>
<div id="contains-all" :class="mode === 'initial' ? 'dark-bg' : 'light-bg'">

  <mode-controller @changeMode="changeMode" v-if="mode === 'initial'"></mode-controller>

  <new-game @resetToInitial="resetToInitial" @joinGame="joinGame" v-if="mode==='create-new'"></new-game>

  <enter-game @resetToInitial="resetToInitial" @startGame="startGame" v-if="mode==='enter-existing'" :initialCode="gameCode"></enter-game>

  <gameplay @resetToInitial="resetToInitial" v-if="mode==='playing'" ></gameplay>

  <section v-if="mode==='instructions'" class="py-4 instructions">
    <button class="button is-small is-primary mr-2" @click="resetToInitial">⤴ Back</button>
    <div class="container is-fluid has-text-centered">
          

      <h1>Storycircle</h1>
      <p>is a pretty simple game. It works best the more deranged your mind is.</p>
      <p>The goal is to create an absurd story where each player only knows what happened in the last line.</p>
      <p>One player should set up a new game by clicking "Start a New Game" from the home screen. Enter each player's name, and how many rounds (lines) you want your story to be. You'll get a random game code which you should share with everyone.</p>
      <p>Have everyone "Join a Game" using the code you created. You can enter text up until the bar runs out. Don't worry if you haven't finished a sentence: the point is to get cut off mid-thought!</p>
      <p>When you submit, the story will get passed on to the next player, and you'll receive a player from the person on the other side of the circle. If they haven't submitted yet, you'll have to wait until their next line is ready.</p>
      <p>Once you've finished all the rounds, you'll have a bizarre story to read. Good luck narrating with a straight face!</p>
    </div>
  </section>
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
  height: 100%


#contains-all
  min-height: 100%
  
  &.dark-bg
    background: rgb(131,58,180);
    background: linear-gradient(144deg, rgba(131,58,180,1) 0%, rgba(253,29,29,1) 50%, rgba(252,176,69,1) 100%);

  &.light-bg
    background: rgb(238,174,202);
    background: linear-gradient(215deg, rgba(238,174,202,0.335171568627451) 0%, rgba(148,233,210,0.4051995798319328) 100%);

.mode-header
  font-size 1.2em
  font-weight 700
  padding-bottom 5px
  border-bottom 2px solid bluish

.instructions
  h1
    font-family: 'Sirin Stencil'
    color: bluish
    font-size: 1.9em
    letter-spacing: 2px
    text-transform: uppercase
  p
    margin-bottom: 10px
  button
    position: fixed
    top: 5px
    left: 5px
</style>
