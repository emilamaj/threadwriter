<template>
  
  <div id="app">
    <div class="details-div">
      <book-details @detailsUpdate="detailsUpdate"/>
    </div>
    <hr class="separator">
    <div class="chapters-div">
      <chapter-generator :bookTitle="bookTitle" :bookGoal="bookGoal" :bookChapters="bookChapters" @chapterUpdate="chapterUpdate"/>
    </div>
    <hr class="separator">
    <div class="sections-div">
      <section-generator :bookTitle="bookTitle" :bookGoal="bookGoal" :bookChapters="bookChapters" :bookSections="bookSections" @sectionUpdate="sectionUpdate" @sectionOverwrite="sectionOverwrite"/>
    </div>
    <hr class="separator">
    <div class="summaries-div">
      <summary-generator :bookTitle="bookTitle" :bookGoal="bookGoal" :bookChapters="bookChapters" :bookSections="bookSections" :bookSummaries="bookSummaries" @summaryUpdate="summaryUpdate" @summaryOverwrite="summaryOverwrite"/>
    </div>
    <hr class="separator">
    <div class="contents-div">
      <content-generator :bookTitle="bookTitle" :bookGoal="bookGoal" :bookChapters="bookChapters" :bookSections="bookSections" :bookSummaries="bookSummaries" :bookContents="bookContents" @contentUpdate="contentUpdate" @contentOverwrite="contentOverwrite"/>
    </div>
  </div>
</template>

<script>
import BookDetails from './components/BookDetails.vue'
import ChapterGenerator from './components/ChapterGenerator.vue'
import SectionGenerator from './components/SectionGenerator.vue'
import SummaryGenerator from './components/SummaryGenerator.vue'
import ContentGenerator from './components/ContentGenerator.vue'

export default {
  name: 'App',
  components: {
    BookDetails,
    ChapterGenerator,
    SectionGenerator,
    SummaryGenerator,
    ContentGenerator,
  },
  data() {
    return {
      bookTitle: "Mediterranean Cuisine For Beginners: A Cookbook",
      bookGoal: "allow the reader to master mediterranean cuisine, even with little cooking experience",
      bookChapters: ["Introduction", "Soups and starters", "Pastas", "Deserts"],
      bookSections: [["Words from the author"], ["Italian soups"], ["Pasta Shapes"], ["Italian deserts"]],
      // Contains the summaries of the sections of the book
      bookSummaries: [
        [
          ["Author's background","Comments by the author","Remarks about the editor","Introduction of the book"]
        ],
        [
          ["Minestrone","Pasta e fagioli","Zuppa di cozze","Zuppa di pomodoro","Zuppa di cipolle"],
        ],
        [
          ["Fusilli","Penne","Spaghetti","Macaroni","Rigatoni"],
        ],
        [
          ["Tiramisu","Baklava","Greek yogurt with honey","Greek yogurt with honey and walnuts","Greek yogurt with honey and almonds"],
          ["Crème brûlée","Cannoli","Croissants","Macarons","Choux pastry","Éclairs","Profiteroles"],
        ],
      ],
      bookContents: [ // Contains the content corresponding to the summaries of the sections of the book
        [
          ["Paragraph about the background of the author","Some comments of the author","Smth bt the editor","Book intro paragraph"]
        ],
        [
          ["a","b","c","",""],
        ],
        [
          ["","","","",""],
        ],
        [
          ["","","","",""],
          ["","","","","","",""],
        ],
      ]
    }
  },
  methods: {
    detailsUpdate(details) {
      this.bookTitle = details.title;
      this.bookGoal = details.goal;
    },
    chapterUpdate(chapters) {
      this.bookChapters = chapters;
      this.bookSections = chapters.map(() => []);
    },
    sectionUpdate(data) {
      this.bookSections[data.chapterIndex] = data.sections;
      this.bookSummaries[data.chapterIndex] = [];
    },
    sectionOverwrite(data) {
      this.bookSections = data;
      this.bookSummaries = data.map(arr => arr.map(() => []));
    },
    summaryUpdate(data) {
      this.bookSummaries[data.chapterIndex][data.sectionIndex] = data.summaries;
    },
    summaryOverwrite(data) {
      this.bookSummaries = data;
    },
    contentUpdate(data) {
      this.bookContents[data.chapterIndex][data.sectionIndex][data.contentIndex] = data.contents;
    },
    contentOverwrite(data) {
      this.bookContents = data;
    },
  },
}
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: gray;
  width: 50em;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-left: auto;
  margin-right: auto;

  padding: 10px
}

.details-div {
  /* margin-top: 10px; */
  background-color: lightgray;
  width: 100%;
}

.chapters-div {
  /* margin-top: 10px; */
  background-color: lightgray;
  width: 100%;
}

.sections-div {
  /* margin-top: 10px; */
  background-color: lightgray;
  width: 100%;
}

.summaries-div {
  /* margin-top: 10px; */
  background-color: lightgray;
  width: 100%;
}

.contents-div {
  /* margin-top: 10px; */
  background-color: lightgray;
  width: 100%;
}

.separator {
  width: 100%;
  border: 0;
  height: 10px;
  margin: 0;
}

</style>
