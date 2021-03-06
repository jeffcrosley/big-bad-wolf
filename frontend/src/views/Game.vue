<template>
  <div id="game" v-if="playersInGame">
    <div class="game__info">
      <p>{{currentGame.name}}</p>
      <p>Your Role: {{currentPlayerRole}}</p>
      <p>Players: {{currentGame.numberOfPlayers}}</p>
      <p>Sheep Policies: {{currentGame.sheepPolicies}}</p>
      <p>Wolf Policies: {{currentGame.wolfPolicies}}</p>
      <p>President: {{currentGame.presidentName}}</p>
      <p>Chancellor: {{currentGame.chancellorName}}</p>
      <p>Chancellor Nominee: {{currentGame.nomineeName}}</p>
    </div>
    <nomination-list v-if="currentUser.id == currentGame.presidentId && currentGame.nomineeName == null" :currentUser="currentUser" @nomination="gameSetup"></nomination-list>
    <ballot v-if="currentGame.nomineeName" :currentUser="currentUser"></ballot>
  </div>
</template>

<script>
import NominationList from '@/components/NominationList'
import Ballot from '@/components/Ballot'
import auth from '../auth'
import api from '../api'

export default {
    name: 'game',
    components: {
      NominationList,
      Ballot
    },
    props: ['currentUser'],
    data() {
      return {
        currentGame: null,
        playersInGame: null,
        currentPlayerRole: ''
      }
    },
    methods: { 
      gameSetup() {
        const authToken = auth.getToken();
        const fetchConfigGet = api.fetchConfigGet(authToken);
        const gameId = this.$route.params.gameId;
        const userId = this.$props.currentUser.id;
        let presidentId = null;
        let chancellorId = null;
        let nomineeId = null;

        fetch(`${process.env.VUE_APP_REMOTE_API}/userRole/${gameId}/${userId}`, fetchConfigGet)
        .then((response) => {
          return response.text();
        })
        .then((userRole) => {
          this.currentPlayerRole = userRole;
          return fetch(`${process.env.VUE_APP_REMOTE_API}/game/${gameId}`, fetchConfigGet)
        })
        .then((response) => {
          return response.json();
        })
        .then((game) => {
          this.currentGame = game;
          presidentId = this.currentGame.presidentId;
          return fetch(`${process.env.VUE_APP_REMOTE_API}/userId/${presidentId}`, fetchConfigGet)
        })
        .then((response) => {
          return response.json();
        })
        .then((president) => {
          this.currentGame.presidentName = president.username;
          chancellorId = this.currentGame.chancellorId;
          return fetch(`${process.env.VUE_APP_REMOTE_API}/userId/${chancellorId}`, fetchConfigGet)
        })
        .then((response) => {
          return response.json();
        })
        .then((chancellor) => {
          this.currentGame.chancellorName = chancellor.username;
          nomineeId = this.currentGame.nomineeId;
          return fetch(`${process.env.VUE_APP_REMOTE_API}/userId/${nomineeId}`, fetchConfigGet)
        })
        .then((response) => {
          return response.json();
        })
        .then((nominee) => {
          this.currentGame.nomineeName = nominee.username;
          return fetch(`${process.env.VUE_APP_REMOTE_API}/usersInGame/${gameId}`, fetchConfigGet)
        })
        .then((response) => {
          return response.json();
        })
        .then((players) => {
          this.playersInGame = players;
        });
      }
    },
    created() {
      this.gameSetup();
    }
}
</script>

<style>
#game > * {
  margin: 2rem;
}
</style>