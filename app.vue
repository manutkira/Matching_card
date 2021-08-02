<template>
  <img
    src="./public/images/peek-a-vue-title.png"
    alt="peak-a-vue"
    class="title"
  />
  <transition-group tag="section" class="game-board" name="shuffle-card">
    <card
      v-for="card in cardList"
      :key="`${card.value}-${card.variant}`"
      :value="card.value"
      :visible="card.visible"
      :position="card.position"
      :matched="card.matched"
      @select-card="flipcard"
    />
  </transition-group>
  <h2>{{ status }}</h2>
  <div class="button">
    <button @click="restartGame" class="restart">
      <img src="./public/images/restart.svg" alt="Restart Game" />Restart Game
    </button>
    <button @click="restartGame" class="restart">Start</button>
  </div>
</template>

<script>
import { launchConfetti } from "./utilities/confetti.js";
import { _ } from "lodash";
import { ref, watch, computed } from "vue";
import card from "./card.vue";
export default {
  components: { card },
  name: "App",
  setup() {
    const cardList = ref([]);
    const userSelection = ref([]);
    const status = computed(() => {
      if (remainingPairs.value === 0) {
        return `You Win!`;
      } else {
        return `Remaining Pairs: ${remainingPairs.value}`;
      }
    });
    const remainingPairs = computed(() => {
      const remainingCards = cardList.value.filter(
        (card) => card.matched === false
      ).length;

      return remainingCards / 2;
    });
    const shuffleCards = () => {
      cardList.value = _.shuffle(cardList.value);
    };

    const restartGame = () => {
      shuffleCards();

      cardList.value = cardList.value.map((card, index) => {
        return {
          ...card,
          matched: false,
          position: index,
          visible: false,
        };
      });
    };

    const cardItems = [
      "bat",
      "candy",
      "cauldron",
      "cupcake",
      "ghost",
      "pumpkin",
      "witch-hat",
      "moon",
    ];

    cardItems.forEach((item) => {
      cardList.value.push({
        matched: false,
        variant: 1,
        value: item,
        position: null,
        visible: true,
      });
      cardList.value.push({
        matched: false,
        variant: 2,
        value: item,
        position: null,
        visible: true,
      });
    });

    cardList.value = cardList.value.map((card, index) => {
      return {
        ...card,
        position: index,
      };
    });

    const flipcard = (payload) => {
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

    watch(remainingPairs, (currentValue) => {
      if (currentValue === 0) {
        launchConfetti();
      }
    });

    watch(
      userSelection,
      (currentValue) => {
        if (currentValue.length === 2) {
          const card1 = currentValue[0];
          const card2 = currentValue[1];

          if (card1.faceValue === card2.faceValue) {
            cardList.value[card1.position].matched = true;
            cardList.value[card2.position].matched = true;
          } else {
            setTimeout(() => {
              cardList.value[card1.position].visible = false;
              cardList.value[card2.position].visible = false;
            }, 700);
          }

          userSelection.value.length = 0;
        }
      },
      { deep: true }
    );

    return {
      flipcard,
      cardList,
      userSelection,
      status,
      remainingPairs,
      shuffleCards,
      restartGame,
      cardItems,
    };
  },
};
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}
#app {
  font-family: Arial, Helvetica, sans-serif;
  /* -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale; */
  text-align: center;
  color: #f5f5f5;
  background-image: url("./public/images/page-bg.png");
  background-color: #00070c;
  height: 100vh;
  padding-top: 60px;
}

.title {
  padding-bottom: 30px;
}

.game-board {
  display: grid;
  grid-template-columns: 120px 120px 120px 120px;
  grid-template-rows: 120px 120px 120px 120px;
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}

.restart {
  display: flex;
  background-color: orange;
  color: white;
  padding: 0.75rem 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: auto;
  font-weight: bold;
}

.button {
  display: flex;
}

.shuffle-card-move {
  transition: transform 0.8s ease-in;
}
</style>
