<template>
    <div class="section-generator-div">
        <p class="label-main">Generate sections:</p>
        <p class="label-info">Click on a chapter to select. Generate and pick sections:</p>

        <div class="content-div">
            <div class="tree-viewer-div">
                <p class="description">Book Structure</p>
                <div class="tree-viewer-box">
                    <tree-viewer-2 :chapters="bookChapters" :sections="bookSections" @chapterSelected="chapterSelected"/>
                </div>
            </div>
            <list-selector v-model:left-values="selectedSections" v-model:right-values="generatedSections" @moveRight="moveRight" @moveLeft="moveLeft"/>
        </div>
        <div class="generation-control-div">
            <button class="general-button" @click="cleanSections" ref="cleanButton">Clean Sections</button>
            <button class="general-button" @click="undoCleanup" ref="undoButton" disabled="true">Undo Cleanup</button>
            <button class="general-button" @click="generateSections" ref="generateButton" disabled="true">Generate Sections</button>
            <button class="general-button" @click="autoGenSections" ref="autoButton">Auto</button>
        </div>
    </div>
</template>

<script>
import ListSelector from './ListSelector.vue'
import TreeViewer2 from './TreeViewer2.vue'

export default {
  name: 'SectionGenerator',
  components: {
    ListSelector,
    TreeViewer2
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
      default: function () {
        return []
      }
    },
    bookSections: {
      type: Array,
      default: function () {
        return []
      }
    }
  },
  data() {
    return {
        currentChapter: null,
        selectedSections: [],
        previousSelectedSections: [],
        generatedSections: [],
    }
  },
  methods: {
    chapterSelected(chapterIndex) {
        // When a chapter is selected, display its sections
        console.log("chapterSelected :", chapterIndex)
        this.currentChapter = chapterIndex
        this.selectedSections = this.bookSections[chapterIndex]
        this.$refs.generateButton.disabled = false
    },
    moveRight(selectedLeft) {
        // Move selected sections from left to right
        console.log("moveRight")

        // Keep only the non-empty values in the array toMove
        let toMove = selectedLeft.filter(value => value != "")
        console.log("moveRight", toMove)
        if (toMove.length === 0) {
            return
        }
        // Append the values to generatedChapters:
        this.generatedSections = [...this.generatedSections, ...toMove]
        // Remove the values from selectedChapters:
        this.removeValuesFromArray(this.selectedSections, toMove)
    },
    moveLeft(selectedRight) {
        // Move selected sections from right to left
        console.log("moveLeft")

        // Keep only the non-empty values in the array toMove
        let toMove = selectedRight.filter(value => value != "")
        console.log("moveLeft", toMove)
        if (toMove.length === 0) {
          return
        }
        // Append the values to selectedChapters:
        this.selectedSections = [...this.selectedSections, ...toMove]
        // Remove the values from generatedChapters:
        this.removeValuesFromArray(this.generatedSections, toMove)

    },
    removeValuesFromArray(array, values) {
      values.forEach(value => {
        const index = array.indexOf(value)
        if (index > -1) {
          array.splice(index, 1)
        }
      })
    },
    cleanSections() {
        // Clean the sections
        console.log("cleanSections")
    },
    undoCleanup() {
        // Undo the cleanup
        console.log("undoCleanup")
    },
    generateSections() {
        // This function make a call to the OpenAI Text completion API
        // It will generate a list of chapters based on the book title and goal
        // The generated chapters will be added to the generatedChapters array
        if (this.currentChapter === null) {
            return
        }
        let payload = this._sectionGenPayload(this.bookTitle, this.bookGoal, this.bookChapters, this.currentChapter, this.selectedSections)

        // Make sure the generate button text is changed to "Generating..." and the buttons are not clickable:
        this.$refs.generateButton.innerText = "Generating..."
        this.$refs.generateButton.disabled = true
        this.$refs.cleanButton.disabled = true
        this.$refs.undoButton.disabled = true

        // Make the API call
        let url = "https://api.openai.com/v1/engines/davinci/completions"
        fetch(url, payload)
        .then(response => {
            if (!response.ok) {
            throw new Error("Fetch error");
            }
            return response.json();
        })
        .then(data => {
            // Add the new chapters to the generatedSections array
            console.log("Incoming Data: ", data)
            this.generatedSections = [];
            // Split the returned text by line breaks
            data.choices[0].text.split("\n").forEach(section => {
            if (section.length > 0) {
                // Remove the "Chapter X. " part of the chapter title and trim whitespaces
                let newSec = section.replace(/Section \d+\. /, "").trim()
                // If not already there, add to generatedSections
                if (!this.generatedSections.includes(newSec)) {
                    this.generatedSections.push(newSec)
                }
            }
            })
        })
        .catch(error => {
            console.error("Error when calling the OpenAI API: ", error);
        })
        .finally(() => {
            // Change the generate button text back to "Generate Sections" and make it clickable again:
            this.$refs.generateButton.innerText = "Generate Sections"
            this.$refs.generateButton.disabled = false
            this.$refs.cleanButton.disabled = false
            this.$refs.undoButton.disabled = !!this.previousSelectedSections.length
        })
    },
    autoGenSections() {
        // Automatically generate the sections for every chapter using promises
        // set reuse to true to reuse the already generated sections
        let reuse = false
        // Make sure the buttons are not clickable:
        this.$refs.generateButton.disabled = true
        this.$refs.cleanButton.disabled = true
        this.$refs.undoButton.disabled = true

        // Make an array of promises
        let promises = []
        this.bookChapters.forEach((chapter, i) => {
            let payload = this._sectionGenPayload(this.bookTitle, this.bookGoal, this.bookChapters, i, reuse?this.bookSections[i]:null)
            // Generate new sections
            promises.push(fetch("https://api.openai.com/v1/engines/davinci/completions", payload))
        })
        // Make the API calls
        Promise.all(promises)
        .then(responses => {
            // Check if all responses are OK
            responses.forEach(response => {
                if (!response.ok) {
                    throw new Error("Fetch error");
                }
            })
            // Return the JSON data of each response
            return Promise.all(responses.map(response => response.json()))
        })
        .then(responses => {
            // Copy the 2D array bookSections to a new array
            let newSections
            if (reuse) {
              newSections = this.bookSections.map(arr => arr.slice())
            }else{
              newSections = this.bookChapters.map(() => [])
            }

            // Add the new sections to the new array
            responses.forEach((data, i) => {
                console.log("Incoming Data: ", data)
                // Add the already existing sections to the new array
                let fullText
                if (reuse) {
                  fullText = this.bookSections[i].join("\n") + "\n"
                  fullText += `\nSection ${this.bookSections[i].length+1}.`
                }else {
                  fullText = `\nSection 1.`
                }
                // Add the new sections to the new array
                fullText += data.choices[0].text
                // Split the returned text by line breaks
                fullText.split("\n").forEach(section => {
                    if (section.length > 0) {
                        // Remove the "Chapter X. " part of the chapter title and trim whitespaces
                        let newSec = section.replace(/Section \d+\. /, "").trim()
                        // If not already there, add to generatedSections
                        if (!newSections[i].includes(newSec)) {
                            newSections[i].push(newSec)
                        }
                    }
                })
            })
            // Update bookSections
            this.$emit("sectionOverwrite", newSections)
        })
        .catch(error => {
            console.error("Error when calling the OpenAI API: ", error);
        })
        .finally(() => {
            // Change the generate button text back to "Generate Sections" and make it clickable again:
            this.$refs.generateButton.innerText = "Generate Sections"
            this.$refs.generateButton.disabled = false
            this.$refs.cleanButton.disabled = false
            this.$refs.undoButton.disabled = !!this.previousSelectedSections.length
        })
        
    },
    updateSections(){
        // Called whenever the user changes the selected chapters
        this.$emit('sectionUpdate', {
            chapterIndex: this.currentChapter,
            sections: this.selectedSections
        })
    },
    _sectionGenPayload(title, goal, chapterList, targetChapterIndex, sectionList){
        // This function creates the payload for the OpenAI Text completion API

        let apiKey = process.env.VUE_APP_OPENAI_API_KEY

        // Create the prompt
        let sectionPrompt = `This book is called "${title}". The goal is to ${goal}.\n\nHere is the list of the titles of the chapters of the book:\n`
        chapterList.forEach((chapter, i) => {
            sectionPrompt += `Chapter ${i+1}. ${chapter}\n`
        })
        sectionPrompt += `\nHere is the list of the titles of the sections of the Chapter ${targetChapterIndex+1}. "${chapterList[targetChapterIndex]}":\n`
        if (sectionList != null) {
          sectionList.forEach((section, i) => {
              sectionPrompt += `Section ${i+1}. ${section}\n`
          })
          sectionPrompt += `Section ${sectionList.length+1}.`
        }else{
          sectionPrompt += `Section 1.`
        }
        console.log("Preparing prompt: ", sectionPrompt)

        // Here is the OpenAI API call
        let data = {
            "prompt": sectionPrompt,
            "max_tokens": 500,
            "temperature": 0.3,
            "frequency_penalty": 0.4,
            "presence_penalty": 0.1,
            "stop": ["This book is called", "The goal is to", "Here is"]
        }
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
    }
  },
    watch: {
        selectedSections: {
            handler: function () {
                this.updateSections()
            },
            deep: true
        },
        // Reload component when props change:
        bookTitle: function () {
            this.$forceUpdate()
        },
        bookGoal: function () {
            this.$forceUpdate()
        },
        bookChapters: {
            handler: function () {
                this.currentChapter = null
                this.$refs.generateButton.disabled = true
            },
            deep: true
        },
    }
}
</script>

<style scoped>

.label-main {
  font-size: 1.5em;
  font-weight: bold;
  padding: 0.5em;
  margin: 16px;
}

.label-info {
  font-size: 1.1em;
  font-weight: bold;
  margin: 0;
  padding: 0.1em;
}

.content-div {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  padding: 0.5em;
}

.description {
    font-size: 1.1em;
    font-weight: bold;
    margin-bottom: 5px;
}

.tree-viewer-box {
    background-color: white;
    font-size: 1em;
    width: 14em;
    height: calc(12em - 16px);
    overflow: auto;
    border: 3px grey dotted;
    padding: 5px;
}

</style>
