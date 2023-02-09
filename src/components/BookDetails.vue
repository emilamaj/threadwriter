<template>
    <div class="book-details-div">
      <h1>Enter details about the book:</h1>
      <div class="details-div">
        <p class="input-label">Title</p>
        <input type="text" class="input-field-single" placeholder="Mediterranean Cuisine For Beginners" v-model="bookTitle">
      </div>
      <div class="details-div">
        <p class="input-label">The goal is to...</p>
        <textarea type="text" class="input-field-multi" v-model="bookGoal" placeholder="allow the reader to master mediterranean cuisine, even with little cooking experience"/>
      </div>
    </div>
</template>

<script>

export default {
  name: 'BookDetails',
  components: {},
  props: {},
  data() {
    return {
        bookTitle: "Mediterranean Cuisine For Beginners",
        bookGoal: "allow the reader to master mediterranean cuisine, even with little cooking experience",
    }
  },
  methods: {
    updateChapters(){
        // Called whenever selectedChapters is updated, to $emit the new value to the parent component
        this.$emit('detailsUpdate', {"title": this.bookTitle, "goal": this.bookGoal}) 
    }
  },
    watch: {
        bookTitle: {
        handler: function() {
            this.updateChapters()
        }},
        bookGoal: {
        handler: function() {
            this.updateChapters()
        }},
    }
}
</script>

<style>

.book-details-div {
  /* padding: 20px; */
}

h1 {
  font-size: 1.5em;
  font-weight: bold;
  padding: 0.5em;
}

.details-div {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px 0;
}

.input-label {
  padding-right: 1em;
  display: inline-block;
  font-weight: bold;
}

.input-field-single {
  width: 20em;
  height: 3em;
  font-size: 1.1em;
  color: #2c3e50;
  border: 3px grey solid;
  padding: 0 0.5em;
}

.input-field-multi {
  width: 25em;
  height: 5em;
  font-size: 1.1em;
  color: #2c3e50;
  border: 3px grey solid;
  resize: vertical;
  padding: 0.5em;
}

</style>
