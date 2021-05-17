<template>
  <div
    class="square"
    :class="handleClasses"
    @click="handleClick"
    @click.right.prevent="handleMark"
  >
    {{ squareObj.revealed ? showContents : "" }}
    {{ squareObj.marked && !squareObj.revealed ? "🚩" : "" }}
  </div>
</template>

<script>
export default {
  name: "Square",
  props: ["squareObj"],
  emits: ["clicked", "marked"],
  methods: {
    handleMark() {
      this.$emit("marked", this.squareObj.row, this.squareObj.column);
    },
    handleClick() {
      this.$emit("clicked", this.squareObj.row, this.squareObj.column);
    },
  },
  computed: {
    //show contents: mine, number if not zero  中身を見せる：地雷、またはゼロじゃなければnumber
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
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  background-color: lightgray;
  border-top: 3px solid rgb(245, 243, 243);
  border-left: 3px solid rgb(245, 243, 243);
  border-right: 3px solid darkgrey;
  border-bottom: 3px solid darkgrey;
  cursor: default;
}

.revealed {
  padding: 2px;
  border: 1px solid darkgrey;
}
</style>
