<template>
  <form>
    <input type="radio" name="beginner" value="beginner" v-model="pickedSize" />
    <label for="beginner">Beginner</label>
    <br />
    <input
      type="radio"
      name="intermediate"
      value="intermediate"
      v-model="pickedSize"
    />
    <label for="intermediate">Intermediate</label>
    <input type="radio" name="advanced" value="advanced" v-model="pickedSize" />
    <label for="intermediate">Advanced</label>
  </form>
</template>

<script>
export default {
  name: "TheLevelPicker",
  data() {
    return {
      pickedSize: "intermediate",
    };
  },
  emits: ["changeSize"],
  watch: {
    pickedSize() {
      this.$emit("changeSize", this.pickedSize);
    },
  },
};
</script>
