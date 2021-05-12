<template>
  <div
    class="square"
    :class="handleClasses"
    @click="handleClick"
    @click.right.prevent="handleMark"
  >
    {{ squareObj.revealed ? showContents : "" }}
    {{ squareObj.marked ? "🚩" : "" }}
  </div>
</template>

<script>
//squareObj: {row: Number, column: Number, mine: boolean, number: Number, marked: boolean}
export default {
  name: "Square",
  props: ["squareObj"],
  emits: ["clicked", "marked"],
  data() {
    return {
      show: false,
    };
  },
  methods: {
    handleMark() {
      console.log("called");
      this.$emit("marked", this.squareObj.row, this.squareObj.column);
    },
    handleClick() {
      this.show = true;
      this.$emit("clicked", this.squareObj.row, this.squareObj.column);
    },
  },
  computed: {
    showContents() {
      if (this.squareObj.mine) {
        return "💥";
      } else {
        return this.squareObj.number == 0 ? "" : this.squareObj.number;
      }
    },
    handleClasses() {
      return {
        revealed: this.squareObj.revealed,
      };
    },
  },
};
</script>

<style scoped>
.square {
  width: 20px;
  height: 20px;
  background-color: lightgray;
  border-top: 3px solid rgb(245, 243, 243);
  border-left: 3px solid rgb(245, 243, 243);
  border-right: 3px solid darkgrey;
  border-bottom: 3px solid darkgrey;
}

.clickedMine {
  background-color: red;
}

.revealed {
  background-color: lightgray;
  padding: 2px;
  border: 1px solid darkgrey;
}
</style>
