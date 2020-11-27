
<template>

<section>
  <div class="container is-fluid pt-5">

    <div class="mode-header mb-3"><button class="button is-small is-primary mr-2" @click="$emit('resetToInitial')">⤴ Back</button><strong>{{$parent.gameData.identifier}}</strong> → <strong>{{$parent.chosenPlayer}}</strong></div>


 <div v-if="!roundsCompleted" class="my-4">
        <h4 class="title is-5">
          Round {{ thisPlayerRoundNumber }} of {{ $parent.gameData.rounds }} <span v-if="thisPlayerRoundNumber+1===$parent.gameData.rounds" class="tag is-warning">Second to last round!</span><span v-if="thisPlayerRoundNumber===$parent.gameData.rounds" class="tag is-warning">Last round!</span>
        </h4>
    <div class="notification is-info is-light" v-if="waitForLastPlayer"><div class="level"><div class="level-item">🤨 Waiting for the player before you in the circle to submit.</div><div class="level-item"><a class="button is-secondary" @click="triggerCheckInterval">Checking automatically in {{checker.countdownTimer}}</a></div></div></div>
        <div class="title is-3 my-3">{{ lastPlayersText }}</div>
          <progress class="progress is-primary" :value="enteredText.length" max="80"></progress>

        <div class="field">
  <div class="control">
    <textarea class="textarea is-primary is-large" autocomplete="off" autocorrect="off" autocapitalize="off" :maxlength="maxLineLength" rows="2" v-model="enteredText" :disabled="waitForLastPlayer"></textarea>
  </div>
  <div class="field mt-3">
    <div class="control">
    <button class="button is-primary is-large" @click="submitEntry" :disabled="waitForLastPlayer || freezeSubmit">📯 Submit</button>
    </div>
  </div>
</div>
<div>
</div>
      </div>


 <div v-if="roundsCompleted" class="my-4">

        <h3 class="title is-3">Here’s your beautiful story 🎭</h3>
        <p v-for="line in completedStory">{{ line }}</p>

  </div>



</div>


</section>

</template>

<script lang="ts">

import Vue from "vue";
import faunadb, { query as q } from "faunadb";


var client = new faunadb.Client({ secret: 'fnAD4dsbeZACBrX3LE8aTjpoBmxbnhDMaBrkYFEm' });

  export default Vue.extend({

    components: {
    },

    data: function() {
      return {
        maxLineLength: 80,
        freezeSubmit: false,
        enteredText: "",
        checker: {interval: null, countdownTimer: 10}
      }
    },

    computed: {
      thisPlayerOrdinal: function() {
        return this.$parent.gameData.players[this.$parent.chosenPlayer].ordinal;
      },

      thisPlayerRoundNumber: function(){
        return this.$parent.gameData.players[this.$parent.chosenPlayer].lastPlayed + 1;
      },

      thisStoryOrdinal: function() { return ((this.thisPlayerOrdinal + this.$parent.gameData.players[this.$parent.chosenPlayer].lastPlayed) % Object.keys(this.$parent.gameData.players).length).toString(); },

      waitForLastPlayer: function() {
        var round = this.$parent.gameData.players[this.$parent.chosenPlayer].lastPlayed;

        if(round === 0) {
          return false;
        }
        else if( this.$parent.gameData.stories[this.thisStoryOrdinal].length >= round ) {
          return false;
        }
        else { return true; }
      },

      lastPlayersText: function() {
        if(this.waitForLastPlayer || this.$parent.gameData.players[this.$parent.chosenPlayer].lastPlayed === 0) {
          return "";
        } else {
          return this.$parent.gameData.stories[this.thisStoryOrdinal].slice(-1)[0];
        }
      },

      roundsCompleted: function() {
        if(this.thisPlayerRoundNumber > this.$parent.gameData.rounds) {
          return true;
        } else { return false; }
      },

      completedStory: function() {
        var ord = ((this.thisPlayerOrdinal + this.$parent.gameData.players.[this.$parent.chosenPlayer].lastPlayed-1) % Object.keys(this.$parent.gameData.players).length).toString()
        return this.$parent.gameData.stories[ord];

      }

    },

    methods: {

      triggerCheckInterval: function() {
          clearInterval(this.checker.interval);
          this.checker.countdownTimer = 10;

          this.checker.interval = setInterval(()=>{ 
            if(this.waitForLastPlayer){ 
              this.checker.countdownTimer--;

              if(this.checker.countdownTimer == 0) {
                console.log('checking!')
                this.$parent.refreshData();
                this.checker.countdownTimer = 10;
              } 

            } else {
              clearInterval(this.checker.interval)
              this.checker.countdownTimer = 10;
            }

          }, 1000);
      },
     
     submitEntry: function() {
          this.freezeSubmit = true;
          this.$parent.gameData.stories[this.thisStoryOrdinal].push(this.enteredText);

          var queryData = {
            stories: { [this.thisStoryOrdinal]: this.$parent.gameData.stories[this.thisStoryOrdinal] },
            players: { [this.$parent.chosenPlayer]: { lastPlayed: this.thisPlayerRoundNumber } } 
          }
  
          client.query(
            q.Update(
            q.Ref(q.Collection('games'), this.$parent.gameRef),
            { data: queryData },
              )
            )
            .then((r) => {
              this.enteredText = "";
              this.freezeSubmit = false;
              this.$parent.gameData = r.data;

            })
            .catch((error) => {});
        this.triggerCheckInterval();

      }



    },

  });
</script>

<style lang="stylus">

@import './vars.styl'
strong
  border-bottom: 2px solid purplish
</style>
