<template>
  <h1>peek-a-vue</h1>
  <section class="game-board">
    <card
      v-for="(card, index) in cardList"
      :key="`card-${index}`"
      :value="card.value"
      :visible="card.visible"
      :position="card.position"
      :matched="card.matched"
      @select-card="flipcard"
    />
  </section>
  <h2>{{ status }}</h2>
  <button @click="restartGame">Restart Game</button>
</template>

<script>
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

    const cardItems = [1, 2, 3, 4, 5, 6, 7, 8];

    cardItems.forEach((item) => {
      cardList.value.push({
        matched: false,
        value: item,
        position: null,
        visible: true,
      });
      cardList.value.push({
        matched: false,
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
        userSelection.value[1] = payload;
      } else {
        userSelection.value[0] = payload;
      }
    };

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
            cardList.value[card1.position].visible = false;
            cardList.value[card2.position].visible = false;
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
#app {
  font-family: Arial, Helvetica, sans-serif;
  /* -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale; */
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.card {
  border: 5px solid #ccc;
}

.game-board {
  display: grid;
  grid-template-columns: 100px 100px 100px 100px;
  grid-template-rows: 100px 100px 100px 100px;
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}
</style>
