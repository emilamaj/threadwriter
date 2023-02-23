<template>
    <div class="content-generator-div">
        <p class="label-main">Generate piece of content:</p>
        <p class="label-info">Click on a summary to edit/generate content</p>

        <div class="content-div">
            <div class="tree-viewer-div">
                <p class="description">Book Structure</p>
                <div class="tree-viewer-box">
                    <tree-viewer-3 :chapters="bookChapters" :sections="bookSections" :summaries="bookSummaries" :summarySelectable="true" @summarySelected="summarySelected"/>
                </div>
            </div>
            <paragraph-viewer class="paragraph-viewer-box" :paragraph="selectedContent" @paragraphChange="paragraphChange"/>
        </div>
        <div class="generation-control-div">
            <button class="general-button" @click="undoGen" ref="undoButton" disabled="true">Undo Last Gen.</button>
            <button class="general-button" @click="generateContent" ref="generateButton">Generate Elements</button>
            <button class="general-button" @click="autoGenContent" ref="autoButton">Auto</button>
        </div>
</div>
</template>

<script>
import ParagraphViewer from './ParagraphViewer.vue'
import TreeViewer3 from './TreeViewer3.vue'

export default {
  name: 'ContentGenerator',
  components: {
    TreeViewer3,
    ParagraphViewer,
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
    },
    bookContents: {
      type: Array,
      default: function () {
        return []
      }
    },
  },
  data() {
    return {
        currentChapter: null,
        currentSection: null,
        currentSummary: null,
        selectedContent: "",
        previousContents: [],
    }
  },
  methods: {
    summarySelected(selectionData) {
        // When a chapter is selected, display its sections
        console.log(selectionData)
        console.log("chapterSelected :", selectionData.chapterIndex)
        console.log("sectionSelected :", selectionData.sectionIndex)
        console.log("summarySelected :", selectionData.summaryIndex)
        this.currentChapter = selectionData.chapterIndex
        this.currentSection = selectionData.sectionIndex
        this.currentSummary = selectionData.summaryIndex

        this.selectedContent = this.bookContents[selectionData.chapterIndex][selectionData.sectionIndex][selectionData.summaryIndex]
    },
    paragraphChange(paragraph) {
        // Event triggered in the paragraph viewer when the paragraph is changed by the user
        console.log("Paragraph change:", paragraph);
        this.selectedContent = paragraph
    },
    undoGen() {
        // Undo the last generation
    },
    generateContent() {
        // This function make a call to the OpenAI Text completion API
        // Its goal is to generate the summaries for the selected section
        // A summary is a short text that summarizes the content of an element of a section
        // For example, in a chapter about Pastas, the section about the shapes of pasta addresses elements of content best summarized as "Fusilli, Penne, Spaghetti, etc."

        let payload = this._contentGenPayload(this.bookTitle, this.bookGoal, this.bookChapters, this.bookSections, this.bookSummaries, this.currentChapter, this.currentSection, this.currentSummary)

        // Make sure the generate button text is changed to "Generating..." and the buttons are not clickable:
        this.$refs.generateButton.innerText = "Generating..."
        this.$refs.generateButton.disabled = true
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
            let generatedContent = data.choices[0].text
            this.selectedContent = generatedContent
        })
        .catch(error => {
            console.error("Error when calling the OpenAI API: ", error)
        })
        .finally(() => {
            // Make sure the generate button text is changed back to "Generate" and the buttons are clickable again:
            this.$refs.generateButton.innerText = "Generate Elements"
            this.$refs.generateButton.disabled = false
            this.$refs.undoButton.disabled = false
            this.$refs.autoButton.disabled = false
        })
    },
    autoGenContent() {
        // This function automatically generates the content for the entire book
        // It separates each call by 3 seconds to avoid hitting the API limit
        // The progress is displayed in the text of the "auto" button

        // Update UI
        this.$refs.generateButton.disabled = true
        this.$refs.undoButton.disabled = true
        this.$refs.autoButton.disabled = true
        this.$refs.autoButton.innerText = "Auto (0%)"

        // Go through each chapter, section and summary
        let payloads = []
        for (let i=0; i<this.bookChapters.length; i++) {
            for (let j=0; j<this.bookSections[i].length; j++) {
                for (let k=0; k<this.bookSummaries[i][j].length; k++) {
                    // Generate the payload for the current call
                    let payload = this._contentGenPayload(this.bookTitle, this.bookGoal, this.bookChapters, this.bookSections, this.bookSummaries, i, j, k)
                    payloads.push({data:payload, chapter:i, section:j, summary:k})
                }
            }
        }

        // Make the calls, separated by 3 seconds. Cancel all the call if any fails
        let url = "https://api.openai.com/v1/engines/davinci/completions"
        let counter = 0
        let interval = setInterval(() => {
            // Make the API call
            fetch(url, payloads[counter].data)
            .then(response => {
                if (!response.ok) {
                    throw new Error("Network response was not ok.")
                }
                return response.json()
            })
            .then(data => {
                console.log("Incoming Data: ", data)
                let generatedContent = data.choices[0].text

                // Update the book content
                this.$emit('contentUpdate', {
                    chapterIndex: payloads[counter].chapter,
                    sectionIndex: payloads[counter].section,
                    summaryIndex: payloads[counter].summary,
                    contents: generatedContent
                })

                // Update UI
                counter += 1
                this.$refs.autoButton.innerText = `Auto (${Math.floor(counter/payloads.length*100)}%)`
            })
            .catch(error => {
                console.error("Error when calling the OpenAI API: ", error)
                counter = payloads.length // Fulfill the final condition to stop the interval
            })
            .finally(() => {
                if (counter === payloads.length) {
                    clearInterval(interval)
                    // Update UI
                    this.$refs.generateButton.disabled = false
                    this.$refs.undoButton.disabled = false
                    this.$refs.autoButton.disabled = false
                    this.$refs.autoButton.innerText = "Auto"
                }
            })
        }, 3000)

    },
    updateContent(){
        // Called whenever the user changes the selected summary
        this.$emit('contentUpdate', {
            chapterIndex: this.currentChapter,
            sectionIndex: this.currentSection,
            contentIndex: this.currentSummary,
            contents: this.selectedContent
        })
    },
    _contentGenPayload(title, goal, chapterList, sectionList, summaryList, targetChapterIndex, targetSectionIndex, targetSummaryIndex){
        // This function creates the payload for the OpenAI Text completion API

        let apiKey = process.env.VUE_APP_OPENAI_API_KEY

        // Create the prompt
        // Start with the information about the book
        let summaryPrompt = `This book is called ${title}.\n`
        summaryPrompt += `The goal is to ${goal}.\n\n`
        // Add the list of chapters
        summaryPrompt += `Chapter List:\n`
        chapterList.forEach((chapter,index) => {
            summaryPrompt += `Chapter ${index + 1}. ${chapter}\n`
        })
        // Add the list of sections
        summaryPrompt += `\nSections in Chapter ${targetChapterIndex + 1}. ${chapterList[targetChapterIndex]} :\n`
        sectionList[targetChapterIndex].forEach((section,index) => {
            summaryPrompt += `Section ${index + 1}. ${section}\n`
        })
        // Add the list of summaries of the target section
        summaryPrompt += `\nPoints addressed in Section ${targetSectionIndex+1}. ${sectionList[targetChapterIndex][targetSectionIndex]} :\n`
        summaryList[targetChapterIndex][targetSectionIndex].forEach((summary,index) => {
            summaryPrompt += `${index + 1}- ${summary}\n`
        })
            
        // Add the prompt to make it generate the content 
        summaryPrompt += "\nThe following is taken right from the book and given for free in its entierty\n"
        summaryPrompt += `Title: ${targetSummaryIndex+1}. ${summaryList[targetChapterIndex][targetSectionIndex][targetSummaryIndex]}\n`
        summaryPrompt += `The summary corresponds to the following 5 paragraphs:\n`

        
        console.log("Preparing prompt: ", summaryPrompt)
        
        // Here is the OpenAI API call
        let data = {
            "prompt": summaryPrompt,
            "max_tokens": 1500,
            "temperature": 0.3,
            "frequency_penalty": 0.4,
            "presence_penalty": 0.1,
            "stop": ["This book is called", "The goal is to"]
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
        selectedContent: {
            handler: function () {
                this.updateContent()
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
  align-items: flex-end;
  padding: 0.5em;
}

.description {
    font-size: 1.1em;
    font-weight: bold;
    margin-bottom: 5px;
}

.tree-viewer-div {
    padding: 20px;
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

.paragraph-viewer-box {
    /* background-color: white; */
    /* font-size: 1em; */
    /* overflow: auto; */
    /* border: 3px grey dotted; */
    /* padding: 5px; */
}

</style>
