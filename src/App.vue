
<script>
import { ref, watch } from "vue";
import { launchConfetti } from "./utilities/confetti";
import Card from "./components/Card.vue";
import createDeck from "../src/features/createDeck";
import deckCards from "./data/deckCards.json";
import handleGameSetting from "./features/handleGameSetting";

export default {
  name: "App",
  components: {
    Card,
  },
  setup() {
    const picked = ref("");
    const showCards = ref(false);
    const attempts = ref(0);
    const { cardList } = createDeck(deckCards);
    const { newGame, startGame, restartGame, remainingPairs, status } =
      handleGameSetting(cardList, attempts);
    const userSelection = ref([]);

    const flipCard = (payload) => {
      cardList.value[payload.position].visible = true;
      if (userSelection.value[0]) {
        if (
          userSelection.value[0].position === payload.position &&
          userSelection.value[0].faceValue === payload.faceValue
        ) {
          return;
        } else {
          userSelection.value[1] = payload;
        }
      } else {
        userSelection.value[0] = payload;
      }
    };

    const chooseBoard = async () => {
      // if (picked.value === "1") {
        
      //   cardList.value = await createDeck(deckCards);
      // }
      if (picked.value === "1.3") {
        cardList.value = cardList.value.slice(0, 12);
        console.log("cardList", cardList.value);
      }
      showCards.value = true;
    };

    watch(remainingPairs, (currentValue) => {
      if (currentValue === 0) {
        launchConfetti();
      }
    });

    watch(
      userSelection,
      (currentValue) => {
        if (currentValue.length === 2) {
          const cardOne = currentValue[0];
          const cardTwo = currentValue[1];
          attempts.value += 1;
          if (cardOne.faceValue === cardTwo.faceValue) {
            cardList.value[cardOne.position].matched = true;
            cardList.value[cardTwo.position].matched = true;
          } else {
            setTimeout(() => {
              cardList.value[cardOne.position].visible = false;
              cardList.value[cardTwo.position].visible = false;
            }, 1500);
          }

          userSelection.value.length = 0;
        }
      },
      { deep: true }
    );

    return {
      chooseBoard,
      cardList,
      flipCard,
      userSelection,
      status,
      restartGame,
      startGame,
      newGame,
      picked,
      showCards,
      attempts,
    };
  },
};
</script>

<template>
  <h1>Memory Game Vue.js</h1>
  <h2 v-if="!showCards" >choose game board</h2>
  <div v-if="!showCards" id="v-model-radiobutton" class="radioButton">
    <input
      type="radio"
      id="one"
      value="1"
      v-model="picked"
      style="width: 2rem; height: 2rem"
    />
    <label style="font-size: 1.6rem; margin-right: 10px" for="one">4x4</label>
    <br />
    <input
      type="radio"
      id="two"
      value="1.3"
      v-model="picked"
      style="width: 2rem; height: 2rem"
    />
    <label style="font-size: 1.6rem; margin-right: 10px" for="two">4x3</label>
    <br />
    <!-- <span>Picked: {{ picked }}</span> -->
    <button style="background-color:#e23c3c;margin-left: 20px;" @click="chooseBoard">
      <h3>Choose Board</h3>
    </button>
  </div>
  <transition-group
    v-else
    tag="section"
    class="game-board"
    name="shuffle-card"
  >
    <Card
      v-for="card in cardList"
      :key="`${card.value}-${card.variant}`"
      :matched="card.matched"
      :value="card.value"
      :visible="card.visible"
      :position="card.position"
      @select-card="flipCard"
    />
  </transition-group>

  <h2 v-if="showCards">Number Of Attempts: {{ attempts }}</h2>
  <h2 v-if="showCards">{{ status }}</h2>
  <button
    v-if="showCards && newGame"
    class="button"
    type="button"
    @click="startGame"
  >
    <img src="../public/images/play.png" width="30" height="28" />
    Start Game
  </button>
  <div v-else />
  <button v-if="!newGame" class="button" type="button" @click="restartGame">
    <img src="../public/images/Refresh.png" width="30" height="28" />
    Restart Game
  </button>
</template>

<style>
html,
body {
  margin: 0;
  padding: 0;
}

h1 {
  margin-top: 0;
  padding-bottom: 30px;
}
a {
  color: white;
  text-decoration: none;
}
a:hover {
  text-decoration: underline;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-image: url("../public/images/backgroundApp.png");
  height: 100vh;
  color: #fff;
  padding-top: 60px;
}
.game-board {
  display: grid;
  grid-template-columns: repeat(4, 130px);
  grid-template-rows: repeat(4, 130px);
  grid-column-gap: 12px;
  grid-row-gap: 12px;
  justify-content: center;
}

.button {
  background-color: #e23c3c;
  color: #fff;
  padding: 0.85em;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 auto;
  font-weight: bold;
}

.button img {
  margin-right: 10px;
}

.shuffle-card-move {
  transition: transform 0.8s ease-in;
}

.radioButton {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
</style>
