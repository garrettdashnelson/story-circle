
<template>

<section class="pt-5">
  <div class="container is-fluid" v-if="status != 'creation-successful'">

    <div class="mode-header mb-3"><button class="button is-small is-primary mr-2" @click="$emit('resetToInitial')">⤴ Back</button> Start a New Game</div>
    
    <h4 class="title is-4">Players</h4>
    <div class="field has-addons" v-for="(player,index) in playersEntered">
      <p class="control">
          <input class="input" type="text" v-model="playersEntered[index]"></input>
      </p>
      <p class="control">
        <a class="button is-primary" @click="removePlayer(index)">ⓧ</a>
      </p>
    </div>
    <div>
      <div class="my-2">
      <button class="button is-primary" @click="addPlayer">⊕ Add new player</button>
      </div>
      <div class="control my-3 select">
        <select name="randomize-players" id="randomize-players" v-model="randomize">
          <option value="randomize">Randomize Order</option>
          <option value="no-randomize">Use This Order</option>
        </select>
      </div>
    </div>

    <h4 class="title is-4 mt-5">Number of rounds</h4>

    <div class="field my-2">
        <input type="range" min="3" max="16" v-model="numberRounds" class="slider"><span class="number-rounds">{{numberRounds}}</span>
    </div>
    <div class="mt-5">
      <button class="button is-primary" :class="status === 'creating-in-progress' ? 'is-loading' : ''" :disabled="playersEntered.length === 0 || status === 'creating-in-progress' " @click="createGame">🐣 Create Game</button>
    </div>
  </div>

  <div class="container is-fluid" v-if="status === 'creation-successful'">
    <h3 class="title is-size-3 my-3">Success!</h3>
    <p class="is-size-4 my-2">Your new game code is<br><strong>{{successfulGameCode}}</strong></p>
    <p>Tell your friends to click "Join a Game" from the main screen and enter this code.</p>
    <p class="mt-3"><a class="button is-primary" @click="$emit('joinGame',successfulGameCode)">🏃🏻‍♀️ Join this game</a></p>
  </div>
</section>
</template>

<script lang="ts">

import Vue from "vue";
import faunadb, { query as q } from "faunadb";
import {hri} from "human-readable-ids";

var client = new faunadb.Client({ secret: 'fnAD4dsbeZACBrX3LE8aTjpoBmxbnhDMaBrkYFEm' });

  export default Vue.extend({

    components: {
    },

    data: function() {
      return {
        status: 'initial',
        playersEntered: ['Player 1','Player 2'],
        numberRounds: 8,
        successfulGameCode: '',
        randomize: 'randomize'
      }
    },

    computed: {
    },

    methods: {
      addPlayer: function() {
        this.playersEntered.push('New Player')
      },

      removePlayer: function(i) {
        this.playersEntered.splice(i,1);
      },

      createGame: function() {
        this.status = 'creating-in-progress';

        var playersObj = {}

        var pe = this.playersEntered;
        if(this.randomize === 'randomize'){
          console.log('randomizing')
              for (let i = pe.length - 1; i > 0; i--) {
              const j = Math.floor(Math.random() * (i + 1));
              [pe[i], pe[j]] = [pe[j], pe[i]];
              }
          console.log(pe);
        }

        pe.forEach((p,i) => {playersObj[p] = {lastPlayed: 0, ordinal: i}});

        var storiesObj = {}
        Array.from(Array(this.playersEntered.length)).forEach((_,i)=>{ storiesObj[i.toString()] = [] });


        var gameData = {
          players: playersObj,
          rounds: +this.numberRounds,
          stories: storiesObj
        };

        const writeNewGame = (c) => { 
          var gd = gameData;
          gd.identifier = c;


          client.query(
            q.Create(q.Collection('games'), {data: gd})
          )
          .then(r => { this.successfulGameCode = c; this.status = 'creation-successful'; })
          .catch(() => this.status = 'error');

        }

        const tryNewGameCode = () => {
          var gameCode = hri.random();
          client.query(
                q.Paginate(q.Match(q.Index('games-by-identifier'), gameCode))
          )
          .then(r => {
            if(r.data.length === 0) { writeNewGame(gameCode); } else { tryNewGameCode(); }
          })
          .catch(() => this.status = 'error');
        }

        tryNewGameCode();


      }
    }
  });
</script>

<style lang="stylus">

@import './vars.styl'
p strong
  border-bottom: 2px solid purplish

.number-rounds
  background-color: #444
  font-weight bold
  color white
  padding 3px 5px
  border-radius 2px
  margin-left 5px
  position relative
  bottom 4px
</style>
