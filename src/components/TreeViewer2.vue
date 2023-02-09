<template>
    <div class='tree-viewer'>
        <!-- Display the chapters, clicking them toggles the display of the sections -->
        <div class='chapter-div' v-for="(chapter,i) in chapters" :key="chapter">
            <div class="chapter-title">
                <button class="general-button" @click="toggleDisplay(i)" :ref="'tgl_' + i">{{ displayChildren[i]?"-":"+" }}</button>
                <p class="chapter-label" @click="selectChapter(i)" :style="currentChapter===i?selectedStyle:unselectedStyle"> {{ chapter }} </p>
            </div>
            
            <div class="section-div" v-if='displayChildren[i]'>
                <p class="section-label" v-for="section in sections[i]" :key="section"> {{ section }} </p>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'TreeViewer2',
    data() {
        return { 
            displayChildren: [],
            currentChapter: null,
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
    },
    methods: {
        selectChapter(index) {
            this.currentChapter = index;
            this.$emit('chapterSelected', index);

        },
        toggleDisplay(index) {
            this.displayChildren[index] = !this.displayChildren[index];
        },
    },
    watch: {
        // When chapters changes, reset displayChildren
        chapters: {
            handler: function() {
                this.displayChildren = this.chapters.map(() => true);
            }
        }
    },
    mounted() {
        this.displayChildren = this.chapters.map(() => true);
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

.chapter-div {
    padding: 0;
}

.chapter-title {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin: 0.2em;
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

.section-label {
    font-size: 0.9em;
    font-weight: normal;
    padding: 0;
    margin: 0;
    white-space: nowrap;
}

</style>