
<template>
<section class="pt-3">
  <div class="container is-fluid">

  <div v-if="status != 'gameCompleted'">
  <h1 class="title is-3">Join game</h1>      
        <div class="notification is-warning" v-if="error.inError">
    {{ error.message }}
  </div>

      <div v-if="status === 'initial' || status === 'evaluatingGameCode'">
      <h4 class="title is-4">Game code</h4>

      <div class="field">
      <div class="control">
        <input class="input" type="text" placeholder="Enter game code" v-model="gameCodeInput" :disabled="this.status === 'evaluatingGameCode'">
      </div>
            <p class="has-text-grey-light">Don’t worry about capitalization</p>

      <div class="control mt-3">
        <a class="button is-primary" :class="[this.status === 'evaluatingGameCode' ? 'is-loading' : '']" @click="loadGame">
          🫖 Load Game
        </a>
      </div>
      
      </div>
      </div>

      <div v-if="status === 'choosePlayer'">

        <h4 class="title is-4"><strong>{{ gameData.identifier }}</strong></h4>
          <div class="field is-horizontal" v-if="status === 'choosePlayer'">
            <div class="field-label is-normal">
              <label class="label">What player are you?</label>
            </div>
            <div class="field-body">
              <div class="field is-narrow is-grouped">
                <div class="control">
                  <div class="select is-fullwidth">
                    <select v-model="chosenPlayer">
                      <option v-for="playerData,player in gameData.players">{{ player }}</option>
                    </select>
                  </div>
                </div>

                        <p class="control">
          <a class="button is-primary" @click="selectPlayer" >
            Play!
          </a>
        </p>

              </div>
            </div>
          </div>
        </div>
  </div>

    <div v-if="status === 'gameCompleted'">
    <h4 class="title is-4"><strong>{{ gameData.identifier }}</strong></h4>
    <h2 class="title is-3">Completed game</h2>

    <div v-for="[k,v] in Object.entries(gameData.stories)" class="my-3">
      <h4 class="title is-4">Story {{parseInt(k) + 1}}</h4>
      <p v-for="line in v">{{line}}</p>
    </div>
  </div>

  </div>


 
</section>
</template>

<script lang="ts">

import Vue from "vue";
import faunadb, { query as q } from "faunadb"

var client = new faunadb.Client({ secret: 'fnAD4dsbeZACBrX3LE8aTjpoBmxbnhDMaBrkYFEm' });

  export default Vue.extend({

    components: {
    },

    props: ['initialCode'],
    data: function() {
      return {
        status: "initial",
        error: {
          inError: false,
          message: ""
        },
        gameCodeInput: this.initialCode,
        confirmedGameCode: "",
        gameData: null,
        gameRef: null,
        chosenPlayer: "",
      }
    },

    computed: {
      
    },

    methods: {

      clearError: function() {
        this.error.inError = false;
        this.error.message = "";
      },

      loadGame: function(){
        this.status = "evaluatingGameCode"
        var gci = this.gameCodeInput.toLowerCase();

        client.query(
            q.Get(
              q.Match(q.Index('games-by-identifier'), gci)
                )
        )
          .then(r => {
            this.clearError();
            this.gameData = r.data;
            this.gameRef = r.ref.value.id;
            const rounds = this.gameData.rounds;
            var t = true;
            for(const [k,v] of Object.entries(this.gameData.stories)) {
            if(v.length < rounds){ t = false }
            }

            if(t) {
            this.status = "gameCompleted";
            } else { this.status ="choosePlayer" }
            
          } )
          .catch(() => { this.status = "initial"; this.error.inError = true; this.error.message="Game code not found, please try again." } );
        
      },

      selectPlayer: function() {
        if(this.chosenPlayer === "") {
          this.error.inError = true;
          this.error.message = "Choose a player!";
        } else {
          this.clearError();
        this.$emit('startGame', this.gameRef, this.chosenPlayer)
        }
      }

    }



  });
</script>

<style scoped lang="stylus">

@import './vars.styl'

strong
  border-bottom 3px solid purplish

    
</style>
