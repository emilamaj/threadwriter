<template>
  <div class="list-selector-div">
    <div class="pane">
        <p class="description">To keep</p>
        <select multiple="true" class="text-select" v-model="selectedLeft">
          <option v-for="item in leftValues" :key="item" :value="item">
              {{ item }}
          </option>
        </select>
    </div>
    <div class="arrows">
      <button class="arrow-right" @click="moveRight">&gt;</button>
      <button class="arrow-left" @click="moveLeft">&lt;</button>
    </div>
    <div class="pane">
        <p class="description">Proposition</p>
        <select multiple="true" class="text-select" v-model="selectedRight">
          <option v-for="item in rightValues" :key="item" :value="item">
              {{ item }}
          </option>
        </select>
    </div>
  </div>
</template>

<script>
export default {
  name: "ListSelector",
  props: {
    leftValues: {
      type: Array,
      default: () => []
    },
    rightValues: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
        selectedLeft: [""],
        selectedRight: [""]
    };
  },
  methods: {
    moveRight() {
        // Emits a "moveRight" event with "this.selectedLeft" as paramter to the parent component to signal that the value has changed
        this.$emit("moveRight", this.selectedLeft);
        this.selectedLeft = [""];
    },
    moveLeft() {
        // Emits a "moveLeft" event with "this.selectedRight" as paramter to the parent component to signal that the value has changed
        this.$emit("moveLeft", this.selectedRight);
        this.selectedRight = [""];
    },
  }
};
</script>

<style scoped>
.list-selector-div {
  display: flex;
  flex-direction: row;
  padding: 20px;
}

.text-select {
  font-size: 1em;
  width: 13em;
  height: 12em;
  overflow: auto;
  border: 3px grey solid;
  padding: 5px;
}

.arrows {
  margin: 0 10px;
  padding-top: 1em;
  height: auto;
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
}

button {
  /* font-weight: bold; */
  font-size: 1.5em;
  color: rgb(70, 70, 70);
  border: 1px solid grey;
  border-radius: 5px;
  background-color: white;
  cursor: pointer;
}

.pane .description {
  font-size: 1.1em;
  font-weight: bold;
  margin-bottom: 5px;
}


</style>