<template>
    <div class='tree-viewer'>
        <!-- Display the chapters, clicking them toggles the display of the sections -->
        <div class='chapter-div' v-for="(chapter,i) in chapters" :key="chapter">
            <div class="chapter-title">
                <button class="general-button" @click="toggleChapter(i)">{{ displaySections[i]?"-":"+" }}</button>
                <p class="chapter-label" :style="currentChapter===i?selectedStyle:unselectedStyle"> {{ chapter }} </p>
            </div>
            
            <template v-if='displaySections[i]'>
                <div class="section-div" v-for="(section,j) in sections[i]" :key="section">
                    <div class="section-title" >
                        <button class="general-button" @click="toggleSection(i,j)">{{ displaySummaries[i][j]?"-":"+" }}</button>
                        <p class="section-label" @click="selectSection(i,j)" :style="(currentChapter==i && currentSection==j) ? selectedStyle:unselectedStyle"> {{ section }} </p>
                    </div>

                    <template v-if='displaySummaries[i][j]'>
                        <div class="summary-div" v-for="summary in summaries[i][j]" :key="summary">
                            <p class="summary-label"> {{ summary }} </p>
                        </div>
                    </template>
                </div>
            </template>
        </div>
    </div>
</template>

<script>

export default {
    name: 'TreeViewer3',
    data() {
        return { 
            displaySections: [],
            displaySummaries: [],
            currentChapter: null,
            currentSection: null,
            selectedStyle: { "background-color": "skyblue", "color": "white"},
            unselectedStyle: { "background-color": "", "color": "black"}
        }
    },
    props: {
        chapters: {
            type: Array,
            default: function () {
                return []
            }
        },
        sections: {
            type: Array,
            default: function () {
                return []
            }
        },
        summaries: {
            type: Array,
            default: function () {
                return []
            }
        },
    },
    methods: {
        selectSection(chapterIndex, sectionIndex) {
            // Update the current chapter and section
            this.currentChapter = chapterIndex;
            this.currentSection = sectionIndex;
            this.$emit('sectionSelected', {"chapterIndex":chapterIndex, "sectionIndex":sectionIndex});
        },
        toggleChapter(chapterIndex) {
            // Toggle the display of the sections of a given chapter
            this.displaySections[chapterIndex] = !this.displaySections[chapterIndex];
        },
        toggleSection(chapterIndex, sectionIndex) {
            // Toggle the display of the summaries of a given section
            this.displaySummaries[chapterIndex][sectionIndex] = !this.displaySummaries[chapterIndex][sectionIndex];
        },
    },
    watch: {
        // When chapters changes, reset displaySections
        chapters: {
            handler: function() {
                this.displaySections = this.chapters.map(() => true);
            }
        },
        // When sections changes, reset displaySummaries
        sections: {
            handler: function() {
                this.displaySummaries = this.sections.map((section) => section.map(() => true));
            }
        }
        
    },
    mounted() {
        // Initialize displaySections and displaySummaries
        this.displaySections = this.chapters.map(() => true);
        this.displaySummaries = this.sections.map((section) => section.map(() => false));
    }
}
</script>

<style scoped>

.tree-viewer {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
}

.general-button {
    font-size: 0.8em;
    /* padding: 0 0.3em; */
    padding: 0px;
    margin: 0em;
    border: grey solid 1px;
    border-radius: 0.1em;
    width: 1.2em;
}

.chapter-div {
    padding: 0;
}

.chapter-title {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin: 0.2em;
}

.chapter-label {
    font-size: 0.9em;
    font-weight: bold;
    padding: 0 0.3em;
    margin: 0;
    white-space: nowrap;
    cursor: pointer;
}

.section-div {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    transform: translate(2em, 0);
}

.section-title {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin: 0.2em;
}

.section-label {
    font-size: 0.8em;
    padding: 0 0.3em;
    margin: 0;
    white-space: nowrap;
    cursor: pointer;
}

.summary-div {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    transform: translate(2em, 0);
}

.summary-label {
    font-size: 0.75em;
    padding: 0 0.3em;
    margin: 0;
    white-space: nowrap;
    cursor: pointer;
}

</style>