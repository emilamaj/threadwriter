<template>
  <div class="chapter-generator-div">
    <h1>Generate and select chapters:</h1>
    <list-selector v-model:left-values="selectedChapters" v-model:right-values="generatedChapters" @moveRight="moveRight" @moveLeft="moveLeft"/>
    <div class="generation-control-div">
      <button class="general-button" @click="cleanChapters" ref="cleanButton">Clean Chapters</button>
      <button class="general-button" @click="undoCleanup" ref="undoButton" disabled="true">Undo Cleanup</button>
      <button class="general-button" @click="generateChapters" ref="generateButton">Generate Chapters</button>
    </div>
  </div>
</template>

<script>
import ListSelector from './ListSelector.vue'

export default {
  name: 'ChapterGenerator',
  components: {
    ListSelector
  },
  props: {
    bookTitle: {
      type: String,
      default: "Mediterranean Cuisine For Beginners"
    },
    bookGoal: {
      type: String,
      default: "allow the reader to master mediterranean cuisine, even with little cooking experience"
    },
    bookChapters: {
      type: Array,
      default: () => ["Introduction","Soups and starters","Pastas","Deserts"]
    }
  },
  data() {
    return {
      selectedChapters: [],
      previousSelectedChapters: [],
      generatedChapters: ["Another chapter", "A chapter about something", "Last chapter"]
    }
  },
  methods: {
    moveRight(selectedLeft) {
        // Keep only the non-empty values in the array toMove
        let toMove = selectedLeft.filter(value => value != "" && !this.generatedChapters.includes(value))
        console.log("moveRight", toMove)
        if (toMove.length === 0) {
            return
        }
        // Append the values to generatedChapters:
        this.generatedChapters = [...this.generatedChapters, ...toMove]
        // Remove the values from selectedChapters:
        this.removeValuesFromArray(this.selectedChapters, toMove)
    },
    moveLeft(selectedRight) {
        // Keep only the non-empty values in the array toMove
        let toMove = selectedRight.filter(value => value != "" && !this.selectedChapters.includes(value))
        console.log("moveLeft", toMove)
        if (toMove.length === 0) {
          return
        }
        // Append the values to selectedChapters:
        this.selectedChapters = [...this.selectedChapters, ...toMove]
        // Remove the values from generatedChapters:
        this.removeValuesFromArray(this.generatedChapters, toMove)
    },
    removeValuesFromArray(array, values) {
      values.forEach(value => {
        const index = array.indexOf(value)
        if (index > -1) {
          array.splice(index, 1)
        }
      })
    },
    cleanChapters() {
      // Use the OpenAI API to keep only the text part of the selected chapters and clean out the rest
      // The cleaned titles will be added to the selectedChapters array
      // A backup of the titles is made in case GPT messes up the cleaning

      if (this.selectedChapters.length === 0) {
        return
      }

      let url = "https://api.openai.com/v1/engines/davinci/completions"
      let apiKey = process.env.VUE_APP_OPENAI_API_KEY

      // Create the prompt
      let chapterPrompt = `Here is a list of chapters:
Chapter 1: Introduction
Chapter 2. Explanations from the author
Warnings and disclaimers
Chapter 4. Matters of interest
Chapter 2 - Last Chapter

We need to process the list so that we only get the title of the chapter on each line, without the numbering or the prefix.
After processing it, we get:
Introduction
Explanations from the author
Warnings and disclaimers
Matters of interest
Last Chapter

We perform the same cleanup operation for the following list (we make sure that we didn't forget anything):\n`

      this.selectedChapters.forEach(chapter => {
        chapterPrompt += chapter + "\n"
      })
      chapterPrompt += `\nWe get:\n`

      // Here is the OpenAI API call
      let data = {
        "prompt": chapterPrompt,
        "max_tokens": 500,
        "temperature": 0.0,
        "frequency_penalty": 0.4,
        "presence_penalty": 0.1,
        "stop": ["Here is a", "We need to", "We perform", "We get"]
      }
      let headers = {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${apiKey}`,
      }

      // Make sure the clean button text is changed to "Cleaning..." and the buttons are not clickable:
      this.$refs.cleanButton.innerText = "Cleaning..."
      this.$refs.generateButton.disabled = true
      this.$refs.cleanButton.disabled = true
      this.$refs.undoButton.disabled = true

      // Make the API call
      console.log("Sending prompt:", chapterPrompt)
      fetch(url, {
        method: "POST",
        headers: headers,
        body: JSON.stringify(data)
      })
      .then(response => {
        if (!response.ok) {
          throw new Error("Fetch error");
        }
        return response.json();
      })
      .then(data => {
        // Add the new chapters to the selectedChapters array
        console.log("Incoming Data:", data)
        this.previousSelectedChapters = [...this.selectedChapters]
        // this.$refs.undoButton.disabled = false // This is not needed because the button is enabled in the finally block (it seems)
        this.selectedChapters = [];
        // Split the returned text by line breaks
        data.choices[0].text.split("\n").forEach(chapter => {
          if (chapter.length > 0) {
            this.selectedChapters.push(chapter)
          }
        })
      })
      .catch(error => {
        console.error("Error when calling the OpenAI API", error);
      })
      .finally(() => {
        // Change the clean button text back to "Clean Chapters" and make it clickable again:
        this.$refs.cleanButton.innerText = "Clean Chapters"
        this.$refs.generateButton.disabled = false
        this.$refs.cleanButton.disabled = false
        this.$refs.undoButton.disabled = !this.previousSelectedChapters.length
      })
    },
    undoCleanup() {
      // Restore the selectedChapters array to its previous state
      this.selectedChapters = [...this.previousSelectedChapters]
    },
    _chapterGenPayload(title, goal, chapterList) {
      // This function creates the payload for the OpenAI API call
      // It takes the book title, the book goal and the list of chapters as input
      // It returns the payload object to be given to fetch(url, payload)

      let apiKey = process.env.VUE_APP_OPENAI_API_KEY

      // Create the prompt
      let chapterPrompt = `This book is called "${title}". The goal is to ${goal}.\n\nHere is the list of the titles of the chapters of the book:\n`
      chapterList.forEach((chapter, i) => {
        chapterPrompt += `Chapter ${i+1}. ${chapter}\n`
      })
      chapterPrompt += `Chapter ${chapterList.length+1}.`
      console.log("Sending chapter prompt:", chapterPrompt)
      // Here the data obejct
      let data = {
        "prompt": chapterPrompt,
        "max_tokens": 500,
        "temperature": 0.0,
        "frequency_penalty": 0.4,
        "presence_penalty": 0.1,
        "stop": ["This book is called", "The goal is to", "Here is the list", "Chapter 1."]
      }

      // Header object
      let headers = {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${apiKey}`,
      }

      let payload = {
        method: "POST",
        headers: headers,
        body: JSON.stringify(data)
      }

      return payload
    },
    generateChapters() {
      // This function make a call to the OpenAI Text completion API
      // It will generate a list of chapters based on the book title and goal
      // The generated chapters will be added to the generatedChapters array

      let url = "https://api.openai.com/v1/engines/davinci/completions"
      let payload = this._chapterGenPayload(this.bookTitle, this.bookGoal, this.selectedChapters)

      // Make sure the generate button text is changed to "Generating..." and the buttons are not clickable:
      this.$refs.generateButton.innerText = "Generating..."
      this.$refs.generateButton.disabled = true
      this.$refs.cleanButton.disabled = true
      this.$refs.undoButton.disabled = true

      // Make the API call
      fetch(url, payload)
      .then(response => {
        if (!response.ok) {
          throw new Error("Fetch error");
        }
        return response.json();
      })
      .then(data => {
        // Add the new chapters to the generatedChapters array
        console.log("Incoming Data:", data)
        this.generatedChapters = [];
        let fullText = `\nChapter ${this.selectedChapters.length+1}.`
        fullText += data.choices[0].text
        // Split the returned text by line breaks
        fullText.split("\n").forEach(chapter => {
          if (chapter != "") {
            // Remove the "Chapter X. " part of the chapter title
            let newChap = chapter.replace(/Chapter \d+\. /, "")
            if (!this.generatedChapters.includes(newChap)) {
              this.generatedChapters.push(newChap)
            }
          }
        })
      })
      .catch(error => {
        console.error("Error when calling the OpenAI API", error);
      })
      .finally(() => {
        // Change the generate button text back to "Generate Chapters" and make it clickable again:
        this.$refs.generateButton.innerText = "Generate Chapters"
        this.$refs.generateButton.disabled = false
        this.$refs.cleanButton.disabled = false
        this.$refs.undoButton.disabled = !!this.previousSelectedChapters.length
      })
    },
    updateChapters(){
        // Called whenever selectedChapters is updated, to $emit the new value to the parent component
        this.$emit('chapterUpdate', this.selectedChapters) 
    }
  },
  watch: {
      selectedChapters: {
      handler: function() {
          this.updateChapters()
      },
      deep: true
      }
  },
  mounted() {
    // When the component is mounted, we update the chapters
    this.selectedChapters = this.bookChapters
  }
}
</script>

<style>

h1 {
  font-size: 1.5em;
  font-weight: bold;
  padding: 0.5em;
}

.chapter-generator-div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: auto;
}

.generation-control-div {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
}

.general-button {
  margin: 10px;
  padding: 10px;
  font-size: 1.1em;
  border: 3px grey solid;
  border-radius: 5px;
  background-color: white;
  
  /* When hovering */
  transition: 0.3s;
  cursor: pointer;
}

</style>
