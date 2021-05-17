<template>
  <div id="board">
    {{ wonGame ? "CONGRATS! You won the game!" : "" }}
    Mines: {{ mines - markedMines }}
    <div
      className="row"
      v-for="rowNumber in boardArray.length"
      :key="rowNumber"
    >
      <square
        v-for="number in boardArray[rowNumber - 1]"
        :key="number"
        :squareObj="number"
        @clicked="handleClick"
        @marked="handleMark"
      ></square>
    </div>
  </div>
</template>

<script>
import Square from "./Square.vue";

export default {
  name: "Board",
  props: ["boardArray", "mines", "markedMines", "wonGame"],
  components: { Square },
  emits: ["clicked", "marked"],
  methods: {
    handleMark(row, column) {
      this.$emit("marked", row, column);
    },
    handleClick(row, column) {
      this.$emit("clicked", row, column);
    },
  },
};
</script>

<style scoped>
.row {
  display: flex;
  align-items: stretch;
}

#board {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 1rem;
}
</style>
