<template>
  <div v-if="step === 'join'" class="flex w-full h-screen items-center justify-center flex-col">
    <div class="card bg-gray-800 w-10/12 sm:w-2/3 md:w-2/3 lg:w-2/4 xl:w-2/7 p-4 rounded shadow">
      <h2 class="text-white text-2xl">Welcome to</h2>
      <h1 class="text-white text-4xl font-bold leading-none">Cards Against Humanity</h1>
      <div class="border-b border-gray-700 mt-3 w-full"></div>


      <div class="flex w-full flex-col items-center">
        <input v-if="showCodeField" v-model="code" type="text" class="mt-5 w-full"
               placeholder="Game Code...">

        <input v-model="username" type="text" class="mt-5 w-full"
               placeholder="Pick a username...">
        <input v-model="password" type="text" class="mt-5 w-full"
               placeholder="Set a password (Optional)">
        <small class="leading-tight mt-2 text-blue-300">The password is optional, but you might need it if you want to
          join if you refreshed the page or changed the device</small>
        <div class="flex flex-col items-center w-full -mx-2 justify-center">
          <div class="flex w-full my-2">
            <span class="text-white text-base text-red-600 font-bold"> {{errorMessage}}</span>
          </div>
          <div class="flex px-2">
            <button @click="joinGame" class="btn btn-primary mx-2">
              Join
            </button>
            <button @click="goHome" class="btn btn-secondary mx-2">
              Back
            </button>

          </div>

        </div>

      </div>


    </div>

  </div>

  <div class="w-full flex flex-col items-center" v-else-if="game">
    <div class="flex flex-col w-3/4 sm:w-2/3 md:w-2/4 lg:w-2/7 xl:w-1/5">
      <div class="flex flex-col w-full">

        <div class="flex px-2 justify-between">
          <h3 class="text-white text-xl my-2 font-bold text-sm">Players</h3>
          <a v-if="!choosingSuccessor" @click="leaveGame(null)"
             class="cursor-pointer text-indigo-500 text-xl my-2 font-bold text-sm">Leave</a>
          <a @click="cancelLeave" v-if="choosingSuccessor && isAdmin"
             class="cursor-pointer text-indigo-500 text-xl my-2 font-bold text-sm">Cancel</a>

        </div>
        <div v-if="choosingSuccessor && isAdmin" class="flex w-full items-center flex-col">
          <span class="text-white font-bold text-xl text-orange-500 w-2/3 text-center">Who should be the admin once you leave?</span>
          <span class="text-white font-bold  text-white w-2/3 text-center">Click to choose</span>
        </div>
        <div class="flex flex-wrap w-full">
          <div class="flex w-1/3 justify-center" v-for="player in game.players" :key="player.username">
            <div @click="choosingSuccessor ? leaveGame(player.username) : cancelLeave"
                 class="w-full bg-white justify-center rounded mx-2 my-2 relative shadow"
                 :class="{'bg-purple-400':game.judge === player.username,'cursor-pointer': choosingSuccessor}">
              <h2 class="text-center font-bold pt-2 flex justify-center items-center">
              <span v-if="player.username === game.admin" class="text-sm mr-1 "
                    :class="{'text-yellow-300':game.judge === player.username,'text-yellow-600':!game.judge === player.username}">★</span>
                {{player.username}}
              </h2>

              <template v-if="game.judge === player.username">
                <h4 v-if="player.username === username" class="text-sm w-full text-center pb-2">Judge (You)</h4>
                <h4 v-else class="text-sm w-full text-center pb-2 text-red-600">Judge</h4>
              </template>
              <h4 v-else-if="player.username === username"
                  class="text-sm w-full text-center pb-2 text-blue-500 font-bold">You</h4>
              <template v-else>
                <h4 class="text-sm w-full text-center pb-2 text-green-500 font-bold" v-if="player.played">Played</h4>
                <h4 class="text-sm w-full text-center pb-2 blinking-text font-bold text-orange-500" v-else>Waiting...</h4>
              </template>


              <div v-if="isAdmin && !choosingSuccessor" @click="kick(player.username)"
                   class="cursor-pointer w-3 h-3 kick-badge rounded-full bg-red-600 absolute p-3 flex items-center justify-center">
                <span class="text-white">⨉</span>
              </div>
              <div class="w-3 h-3 wins-badge rounded-full bg-red-600 absolute p-3 flex items-center justify-center">
                <span class="text-white">{{player.wins}}</span>
              </div>
            </div>
          </div>

        </div>

      </div>

      <div class="w-full flex justify-center mb-3 relative">
        <div class="w-full flex">
          <ul class="px-2 notifications-list" v-chat-scroll="{always: false, smooth: true}">
            <li class="text-gray-300 font-bold" v-for="notification in game.notifications">{{notification}}</li>
          </ul>
        </div>
      </div>
      <div class="flex w-full justify-center items-center">

        <div v-if="game.previousCard && game.previousCard.text != ''" class="flex w-3/12 pb-2 mx-3 mb-0 flex-col">
          <div class="flex flex-col w-full items-center font-bold">
            <h4 class="text-gray-300 mb-0 text-sm">Winner</h4>
            <p class="text-xl leading-zero mb-1 text-green-600 text-center">{{game.lastWinner}}</p>
          </div>
          <card type="black" :text="game.previousCard.text"></card>
          <div v-if="game.winingCards" class="flex flex-col">
            <card class="mt-2" v-for="card in game.winingCards" :key="card.text" :text="card.text"></card>
          </div>
        </div>
        <div class="flex w-4/6 px-4">
          <div class="flex flex-col w-full">
            <card v-if="game.started" type="black" :pick="game.blackCard.pick" :text="game.blackCard.text">
            </card>
            <card v-else type="black" text="">


              <div class="flex w-full items-center justify-start flex-col ">
                <div @click="copyCode" class="flex w-full flex-col">
                  <h1 class="text-center text-xl">Game Code:</h1>
                  <h1 class="text-center text-3xl mt-1">{{code}}</h1>
                  <h1 class="text-center mt-1 text-xs">Click to copy or share the link with your friends</h1>
                </div>


                <button :class="{'disabled': game.players.length < 3}" @click="startGame"
                        class="btn btn-primary text-sm  font-bold text-center mt-4 mb-3">Start Game
                </button>
                <h1 class="text-center text-xl text-orange-500 blinking-text">Waiting for players</h1>

                <h6 v-if="game.players.length < 3" class="text-sm mt-5 text-center"></h6>

              </div>


            </card>
            <a v-show="isJudge" @click="skip" class="text-sm text-white mt-2">Skip This Card?</a>

          </div>

        </div>
      </div>
      <div class="flex px-4 w-full">
        <input @keyup.enter="sendMessage" v-model="message" type="text"
               class="mt-3 py-1 bg-white w-full px-2" placeholder="Send a message...">
      </div>
    </div>
<div class="w-full lg:w-8/12">
  <template v-if="game.started">
    <div class="flex flex-col w-full mt-4 ">
      <div class="flex px-3">
        <transition name="fade" mode="out-in">
          <h3 key="your-cards" v-if="shouldShowCards" class="text-white text-xl my-1 font-bold">Your Cards</h3>
          <h3 key="submissions" v-else class="text-white text-xl my-1 font-bold">Submissions</h3>
        </transition>
      </div>
      <div class="deck w-full nowrap px-3 relative overflow-scroll">
        <transition name="list" mode="out-in">
          <div key="cards" v-if="shouldShowCards" class="w-full flex-wrap justify-center">
            <div v-for="(card,index) in cards" @click="activeIndex = index"
                 class="game-card w-1/3 sm:w-2/12 md:w-3/8 lg:w-2/8 xl:w-1/7 px-1 inline-block pb-2" :key="card.text">
              <card @play="playCard(card.text)" :active="index === activeIndex" type="white"
                    :text="card.text"></card>

            </div>
          </div>
          <div key="judgingCards" class="w-full flex-wrap justify-center" v-else-if="game.judgingCards.length > 0">

            <div v-for="(player,index) in game.judgingCards"
                 :key="`player-${index}`"
                 @click="activePlayerIndex = index"
                 class="game-card w-1/3 sm:w-2/12 md:w-3/8 lg:w-2/8 xl:w-1/7 px-1 inline-block pb-2 relative">

              <card v-for="card in player" :key="card" :judging="true" @pick="pick(index)"
                    type="white"
                    class="mb-2"
                    :text="card"></card>
              <div class="flex justify-center w-full">
                <button v-if="game.judging && isJudge" @click="pick(index)" class="btn btn-primary btn-sm">Pick
                </button>
              </div>
            </div>

          </div>
          <div key="waiting" class="w-full flex-wrap justify-center" v-else>
            <div v-for="index in game.players.length - 1" class="game-card w-1/3 sm:w-2/12 md:w-3/8 lg:w-2/8 xl:w-1/7 px-1 inline-block pb-2"
                 :key="`${index}-waiting`">
              <card type="white" :text="''">
                <h3 class="text-center">Waiting for players to play</h3>
              </card>
            </div>

          </div>

        </transition>
      </div>

    </div>

  </template>
  <template v-else>
    <div class="flex flex-col w-full mt-4">
      <div class="flex px-3">
        <h3 class="text-white text-xl my-1 font-bold">Your Cards</h3>

      </div>
      <div class="deck w-full nowrap overflow-scroll px-3">
        <div class="flex justify-center">
          <h1 class="text-white text-xl text-gray-600 mt-4">The game has not started yet</h1>
        </div>
      </div>

    </div>

  </template>
</div>


  </div>
</template>

<script lang="ts">
    import Vue from 'vue'
    // @ts-ignore;
    import Card from "~/components/Card.vue";
    import VueSocketIOExt, {Socket} from 'vue-socket.io-extended';
    import io from 'socket.io-client';


    import {Component} from 'vue-property-decorator'
    import axios, {AxiosInstance} from 'axios';

    // @ts-ignore;
    import VueChatScroll from 'vue-chat-scroll';

    Vue.use(VueChatScroll)
    const socket = io(window.location.host);
    Vue.use(VueSocketIOExt, socket);

    interface Player {
        username: string;
        played: boolean;
        wins: number;
    }

    export interface GameCard {
        pick: number;
        text: string;
    }

    class GameData {
        judge: string = "";
        blackCard: GameCard = {text: "", pick: 0};
        players: Player[] = [];
        started: boolean = false;
        judging: boolean = false;
        judgingCards: GameCard[] = [];
        playedCards: number = 0;
        notifications: string[] = [];
        previousCard: GameCard | null = null;
        winingCards: GameCard[] | null = null;
        lastWinner: string | null = null;
        admin: string = "";
    }

    @Component({
        components: {
            Card
        }
    })


    export default class GameView extends Vue {
        game: GameData = new GameData;
        username: string = "";
        message: string = "";
        password: string = "";

        code: string = "";

        step: string = "join";

        cards: string[] = [];

        activeIndex: number | null = null;

        activePlayerIndex: number = 0;

        client: AxiosInstance = axios.create({});

        showCodeField: boolean = false;

        errorMessage: string | null = null;

        choosingSuccessor: boolean = false;

        @Socket("connect")
        connect() {
            console.log('socket connected')
        }

        copyCode() {
            let dummy = document.createElement("textarea");
            document.body.appendChild(dummy);
            dummy.value = this.code;
            dummy.select();
            document.execCommand("copy");
            document.body.removeChild(dummy);
            alert("Code copied to the clipboard")
        }

        joinGame() {
            if (!this.code) {
                this.errorMessage = 'Please enter the game code';
                return;
            }
            this.errorMessage = null;
            this.client.post(`/api/games/${this.code}`).then(response => {
                this.cards = response.data.cards;
                this.step = "game";

                this.setupSocket();
                this.sync(response.data.game);
            }).catch(error => {
                if (error.response && error.response.data)
                    this.errorMessage = error.response.data.message;
                else
                    console.log(error)
            })
        }

        goHome() {
            this.$router.push({
                name: 'index'
            })
        }

        sync(game: GameData) {
            this.game = game;
            this.getCards();
        }

        sendMessage() {
            if (this.message.trim() === '') {
                return;
            }
            this.client.post(`/api/games/${this.code}/message`, {message: this.message});
            this.message = "";

        }

        startGame() {
            if (this.game.players.length < 3)
                return;

            this.client.post(`/api/games/${this.code}/start`);
        }

        playSound(sound: string) {
            let audio = new Audio(`/sounds/${sound}.m4a`);
            audio.play();
        }


        playCard(text: string) {
            this.client.post(`/api/games/${this.code}/play`,
                {
                    card: text
                }).then(response => {
                if (response.data.draw) {
                    // @ts-ignore
                    // this.cards.splice(this.activeIndex, 1, response.data.draw);
                    // @ts-ignore
                    this.activeIndex = null;
                }
            });
        }

        kick(username: string) {
            this.client.post(`/api/games/${this.code}/kick`, {
                kick: username
            });
        }

        leaveGame(successor: string | undefined) {

            if (this.isAdmin && !successor) {
                this.choosingSuccessor = true;
                return;
            }
            this.client.post(`/api/games/${this.code}/leave`, {
                successor: successor
            }).then(response => {

            })
        };

        pick(index: number) {
            this.client.post(`/api/games/${this.code}/pick`, {
                cards: this.game.judgingCards[index],
            }).then(response => {

            })
        }

        setupSocket() {
            if (this.code)
                this.$socket.$subscribe(`update.${this.code}`, (payload: { data: GameData, message: string, sound?: string }) => {
                    this.sync(payload.data);
                    if (payload.sound) {
                        this.playSound(payload.sound);
                    }
                })

        }

        cancelLeave() {
            this.choosingSuccessor = false
        }

        skip() {
            this.client.post(`/api/games/${this.code}/skip`);
        }

        getCards() {
            this.client.get(`/api/games/${this.code}/cards`).then(response => {
                this.cards = response.data.cards;
            });
        }

        get isJudge(): boolean {
            return this.game.judge === this.username;
        }


        get canPlay() {
            let player = this.game.players.find(player => player.username === this.username);
            if (player)
                return !player.played;

            return false;

        }

        get isAdmin(): boolean {
            return this.game.admin === this.username;
        }

        get played(): boolean {
            if (this.game.blackCard)
                return this.game.playedCards === this.game.blackCard.pick;
            else
                return false;
        }

        get shouldShowCards(): boolean {
            if (this.isJudge) {
                return false;
            }
            return this.canPlay;
        }


        mounted(): void {
            this.code = this.$route.params.code;
            if (!this.code)
                this.showCodeField = true;

            this.client.interceptors.request.use((config) => {
                config.headers = {
                    'x-username': this.username,
                    'x-password': this.password,
                }
                console.log(config);
                return config;
            }, function (error) {
                // Do something with request error
                return Promise.reject(error);
            });
            if (this.$route.params.username) {
                this.step = "game";
                this.username = this.$route.params.username;
                this.password = this.$route.params.password;

                this.joinGame();
            }


        }


    }

</script>

<style>

  .notifications-list:before {
    content: '';
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    background: rgba(255,255,255,0.1);


  }

  .notifications-list {
    width: 100%;
    overflow: scroll;
    height: 80px;
    @apply px-4;
  }

  .link {
    color: orangered !important;
  }

  /* Sample `apply` at-rules with Tailwind CSS
  .container {
    @apply min-h-screen flex justify-center items-center text-center mx-auto;
  }
  */
</style>
