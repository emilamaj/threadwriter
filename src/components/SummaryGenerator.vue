<template>
    <div class="summary-generator-div">
        <p class="label-main">Generate content summary:</p>
        <p class="label-info">Click on a section to select. Generate and pick section content:</p>

        <div class="content-div">
            <div class="tree-viewer-div">
                <p class="description">Book Structure</p>
                <div class="tree-viewer-box">
                    <tree-viewer-3 :chapters="bookChapters" :sections="bookSections" :summaries="bookSummaries" @sectionSelected="sectionSelected"/>
                </div>
            </div>
            <list-selector v-model:left-values="selectedSummaries" v-model:right-values="generatedSummaries" @moveRight="moveRight" @moveLeft="moveLeft"/>
        </div>
        <div class="generation-control-div">
            <button class="general-button" @click="cleanSummaries" ref="cleanButton">Clean Elements</button>
            <button class="general-button" @click="undoCleanup" ref="undoButton" disabled="true">Undo Cleanup</button>
            <button class="general-button" @click="generateSummaries" ref="generateButton">Generate Elements</button>
            <button class="general-button" @click="autoGenSummaries" ref="autoButton">Auto</button>
        </div>
</div>
</template>

<script>
import ListSelector from './ListSelector.vue'
import TreeViewer3 from './TreeViewer3.vue'

export default {
  name: 'SectionGenerator',
  components: {
    ListSelector,
    TreeViewer3
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
    },
    bookSummaries: {
      type: Array,
      default: function () {
        return []
      }
    }
  },
  data() {
    return {
        currentChapter: null,
        currentSection: null,
        selectedSummaries: [],
        previousSelectedSummaries: [],
        generatedSummaries: [],
    }
  },
  methods: {
    sectionSelected(selectionData) {
        // When a chapter is selected, display its sections
        console.log("chapterSelected :", selectionData.chapterIndex)
        console.log("sectionSelected :", selectionData.sectionIndex)
        this.currentChapter = selectionData.chapterIndex
        this.currentSection = selectionData.sectionIndex
        this.selectedSummaries = this.bookSummaries[selectionData.chapterIndex][selectionData.sectionIndex]
        this.generatedSummaries = []
        
    },
    moveRight(selectedLeft) {
        // Move selected sections from left to right
        console.log("moveRight")

        // Keep only the non-empty values in the array toMove
        let toMove = selectedLeft.filter(value => value != "" && !this.generatedSummaries.includes(value))
        console.log("moveRight", toMove)
        if (toMove.length === 0) {
            return
        }
        // Append the values to generatedSummaries:
        this.generatedSummaries = [...this.generatedSummaries, ...toMove]
        // Remove the values from selectedSummaries:
        this.removeValuesFromArray(this.selectedSummaries, selectedLeft)
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
        // Append the values to selectedSummaries:
        this.selectedSummaries = [...this.selectedSummaries, ...toMove]
        // Remove the values from generatedSummaries:
        this.removeValuesFromArray(this.generatedSummaries, selectedRight)

    },
    removeValuesFromArray(array, values) {
      values.forEach(value => {
        const index = array.indexOf(value)
        if (index > -1) {
          array.splice(index, 1)
        }
      })
    },
    cleanSummaries() {
        // Clean the Summaries
        // This function make a call to the OpenAI Text completion API
        // It takes as input a list of titles with random mistakes at the beginning like numberings or extra spaces and returns a list of titles with the mistakes removed
        // For example, the input [" 1. Introduction", " 2- Pasta Shapes", " 3-1 Pasta Sauces"] will return ["Introduction", "Pasta Shapes", "Pasta Sauces"]

        // Load API KEY from process.env file
        let apiKey = process.env.VUE_APP_OPENAI_API_KEY

        let cleanPrompt = "We are going to remove the numberings before the titles like in the following:\n"
        cleanPrompt += "1. Introduction\n"
        cleanPrompt += "2-1 Another title\n"
        cleanPrompt += "3-1. Something something\n"
        cleanPrompt += "4. Another line\n"

        cleanPrompt += "\nSo we get:\n"
        cleanPrompt += "Introduction\n"
        cleanPrompt += "Another title\n"
        cleanPrompt += "Something something\n"
        cleanPrompt += "Another line\n"

        cleanPrompt += "\nWe are also going to remove the numberings before the text in the following titles:\n"
        this.selectedSummaries.forEach(summary => {
            cleanPrompt += summary + "\n"
        })

        cleanPrompt += "\nSo we get:\n"

        console.log("cleanPrompt:", cleanPrompt)
        let data = {
            "prompt": cleanPrompt,
            "max_tokens": 500,
            "temperature": 0.1,
            "frequency_penalty": 0,
            "presence_penalty": 0,
            "stop": ["We are going to remove the numberings", "So we get:","We are also going to remove the numberings","So that the clean titles"]
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
        // Make sure the buttons are not clickable:
        this.$refs.generateButton.disabled = true
        this.$refs.cleanButton.disabled = true
        this.$refs.undoButton.disabled = true
        this.$refs.autoButton.disabled = true
        
        // Send the request, catch errors, and update UI elements
        fetch("https://api.openai.com/v1/engines/davinci/completions", payload)
        .then(response => {
            if (!response.ok) {
            throw new Error("Fetch error");
            }
            return response.json();
        })
        .then(data => {
            console.log("incoming data:", data)

            // Split the data into lines and remove empty lines
            let cleanSummaries = data.choices[0].text.split("\n")
            cleanSummaries = cleanSummaries.filter(summary => summary != "")
            // Only push unique elements
            this.selectedSummaries = []
            cleanSummaries.forEach(summary => {
                summary = summary.trim()
                this.selectedSummaries.push(summary)
            })
            this.previousSelectedSummaries.push(this.selectedSummaries.slice())
        })
        .catch(error => {
            console.log("error", error)
        })
        .finally(() => {
            // Update the UI elements
            this.$refs.generateButton.disabled = false
            this.$refs.cleanButton.disabled = false
            this.$refs.undoButton.disabled = false
            this.$refs.autoButton.disabled = false
        })
    },
    undoCleanup() {
        // Undo the cleanup
        // This function removes the last element of the previousSelectedSummaries array and sets it as the selectedSummaries
        if (this.previousSelectedSummaries.length > 0) {
            this.selectedSummaries = this.previousSelectedSummaries.pop()
        }
    },
    generateSummaries() {
        // This function make a call to the OpenAI Text completion API
        // Its goal is to generate the summaries for the selected section
        // A summary is a short text that summarizes the content of an element of a section
        // For example, in a chapter about Pastas, the section about the shapes of pasta addresses elements of content best summarized as "Fusilli, Penne, Spaghetti, etc."

        let payload = this._summaryGenPayload(this.bookTitle, this.bookGoal, this.bookChapters, this.bookSections, this.currentChapter, this.currentSection, this.selectedSummaries)

        // Make sure the generate button text is changed to "Generating..." and the buttons are not clickable:
        this.$refs.generateButton.innerText = "Generating..."
        this.$refs.generateButton.disabled = true
        this.$refs.cleanButton.disabled = true
        this.$refs.undoButton.disabled = true
        this.$refs.autoButton.disabled = true

        // Make the API call
        let url = "https://api.openai.com/v1/engines/davinci/completions"
        fetch(url, payload)
        .then(response => {
            if (!response.ok) {
                throw new Error("Network response was not ok.")
            }
            return response.json()
        })
        .then(data => {
            console.log("Incoming Data: ", data)
            this.generatedSummaries = [];
            data.choices[0].text.split("\n").forEach(summary => {
                if (summary != "") {
                    let newSum = summary.replace(/\d+- /, "").trim()
                    if (!this.generatedSummaries.includes(newSum)) {
                        this.generatedSummaries.push(newSum)
                    }
                }
            })
        })
        .catch(error => {
            console.error("Error when calling the OpenAI API: ", error)
        })
        .finally(() => {
            // Make sure the generate button text is changed back to "Generate" and the buttons are clickable again:
            this.$refs.generateButton.innerText = "Generate Elements"
            this.$refs.generateButton.disabled = false
            this.$refs.cleanButton.disabled = false
            this.$refs.undoButton.disabled = false
            this.$refs.autoButton.disabled = false
        })
    },
    autoGenSummaries() {
        // This function generates the summaries for every section of every chapter of the book
        // It is used to generate the summaries for the book in one go

        let reuse = false

        // Make sure the buttons are not clickable:
        this.$refs.generateButton.disabled = true
        this.$refs.cleanButton.disabled = true
        this.$refs.undoButton.disabled = true
        this.$refs.autoButton.disabled = true

        let chapterSectionFlatIndex = []
        let summaryPromises = []
        this.bookChapters.forEach((chapter, chapterIndex) => {
            this.bookSections[chapterIndex].forEach((section, sectionIndex) => {
                // Create the payload
                let payload = this._summaryGenPayload(this.bookTitle, this.bookGoal, this.bookChapters, this.bookSections, chapterIndex, sectionIndex, reuse?this.bookSummaries[chapterIndex][sectionIndex]:null)
                
                // Make the API call
                let url = "https://api.openai.com/v1/engines/davinci/completions"
                summaryPromises.push(fetch(url, payload))
                chapterSectionFlatIndex.push([chapterIndex, sectionIndex])
            })
        })
        Promise.all(summaryPromises)
        .then(responses => {
            
            responses.forEach(response => {
                if (!response.ok) {
                    throw new Error("Network response was not ok.")
                }
            })
            return Promise.all(responses.map(response => response.json()))
        })
        .then(responses => {
            // Copy the shape of bookSections
            let newBookSummaries = this.bookSections.map(chapter => chapter.map(() => []))

            responses.forEach((data, index) => {
                console.log("Incoming Data: ", data)
                console.log("newBookSummaries: ", newBookSummaries)
                console.log("this.bookSummaries: ", this.bookSummaries)
                // Retrieve the indices pair
                let [chapterIndex, sectionIndex] = chapterSectionFlatIndex[index]
                
                // Craft the list of summaries depending on wether we reused the already generated ones
                let fullText
                if (reuse) {
                    fullText = this.bookSummaries[chapterIndex][sectionIndex].join("\n")
                    fullText += `\n${this.bookSummaries.length + 1}-`
                }else{
                    fullText = "1-"
                }
                fullText += data.choices[0].text
                
                fullText.split("\n").forEach(summary => {
                    if (summary != "") {
                        let newSum = summary.replace(/\d+- /, "").trim()
                        if (!newBookSummaries[chapterIndex][sectionIndex].includes(newSum)) {
                            newBookSummaries[chapterIndex][sectionIndex].push(newSum)
                        }
                    }
                })
            })
            this.$emit('summaryOverwrite', newBookSummaries)
        })
        .catch(error => {
            console.error("Error when calling the OpenAI API: ", error)
        })
        .finally(() => {
            // Make sure the buttons are clickable again:
            this.$refs.generateButton.disabled = false
            this.$refs.cleanButton.disabled = false
            this.$refs.undoButton.disabled = false
            this.$refs.autoButton.disabled = false
        })
    },
    updateSummaries(){
        // Called whenever the user changes the selected summary
        this.$emit('summaryUpdate', {
            chapterIndex: this.currentChapter,
            sectionIndex: this.currentSection,
            summaries: this.selectedSummaries
        })
    },
    _summaryGenPayload(title, goal, chapterList, sectionList, targetChapterIndex, targetSectionIndex, summaryList){
        // This function creates the payload for the OpenAI Text completion API

        let apiKey = process.env.VUE_APP_OPENAI_API_KEY

        // Create the prompt
        let summaryPrompt = `This book is called ${title}.\n`
        summaryPrompt += `The goal is to ${goal}.\n\n`
        summaryPrompt += `Here is a list of the chapter names:\n`
        chapterList.forEach((chapter,index) => {
            summaryPrompt += `Chapter ${index + 1}. ${chapter}\n`
        })
        summaryPrompt += `\nHere is a list of all the sections in Chapter ${targetChapterIndex + 1}. ${chapterList[targetChapterIndex]} :\n`
        sectionList[targetChapterIndex].forEach((section,index) => {
            summaryPrompt += `Section ${index + 1}. ${section}\n`
        })
        summaryPrompt += `\nHere is a list of all the points addressed in Section ${targetSectionIndex+1}. ${sectionList[targetChapterIndex][targetSectionIndex]} :\n`
        if (summaryList != null) {
            summaryList.forEach((summary,index) => {
                summaryPrompt += `${index + 1}- ${summary}\n`
            })
            summaryPrompt += `${summaryList.length + 1}-`
        }else{
            summaryPrompt += "1-"
        }
        
        console.log("Preparing prompt: ", summaryPrompt)
        
        // Here is the OpenAI API call
        let data = {
            "prompt": summaryPrompt,
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
        selectedSummaries: {
            handler: function () {
                this.updateSummaries()
            },
            deep: true
        }
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
