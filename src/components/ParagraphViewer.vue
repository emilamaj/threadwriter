<template>
  <div class="paragraph-viewer-div">
    <textarea class="paragraph" v-model="currentParagraph" @input="updateParagraph()"/>
  </div>
</template>

<script>
export default {
    name: "ParagraphViewer",
    props: {
        paragraph: {
            type: String,
            default: function () {
                return "";
            },
        },
    },
    data() {
        return {
            currentParagraph: "",
        };
    },
    methods: {
        updateParagraph() {
            console.log("Paragraph change:", this.currentParagraph);
            this.$emit("paragraphChange", this.currentParagraph);
        },
    },
    watch: {
        paragraph: {
            handler: function () {
                this.currentParagraph = this.paragraph;
            },
        },
    },
};
</script>

<style scoped>
/* .paragraph-viewer-div {
  display: flex;
  flex-direction: row;
  padding: 20px;
} */

.paragraph {
  font-size: 1em;
  width: 26em;
  height: 13em;
  overflow: auto;
  border: 3px grey solid;
  padding: 5px;
  margin: 20px;
}



</style>