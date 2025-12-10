<template>
    <div class="card w-100">
        <div class="card-header">
            <h4 class="w-100">Segmentation of Phrase Structure</h4> <button href="#"
                class="btn-save-mei btn btn-primary ml-1" @click="saveToMEI" title="Apply Information To MEI File"
                data-bs-customClass="custom-tooltip" data-bs-toggle="tooltip" data-bs-placement="bottom"
                data-bs-html="true">Apply To MEI</button>
        </div>
        <div class="card-body container drag-container">
            <div id="form" class="mt-1 mb-3 pt-0 pb-0 p-5">
                <div class="w-100" @click="cleanPaintSVG">
                    <button class="mb-3 ml-3" @click="addPhrase" title="Add a New Phrase"
                        data-bs-customClass="custom-tooltip" data-bs-toggle="tooltip" data-bs-placement="bottom"
                        data-bs-html="true">Add</button>
                    <button class="mb-3 btn-automatic" @click="getAutomaticPhrases" title="Add a New Phrase"
                        data-bs-customClass="custom-tooltip" data-bs-toggle="tooltip" data-bs-placement="bottom"
                        data-bs-html="true">Automatic Segmentation</button>
                </div>
                <draggable :list="phraseSegmentData[0].value" class="drag-area" group="n">
                    <div v-for="item in phraseSegmentData[0].value" class="row phrase-sel drag-item" :key="item.n">
                        <div class="col col-xs-1" title="Number of the Phrase (min: 1)"
                            data-bs-customClass="custom-tooltip" data-bs-toggle="tooltip" data-bs-placement="bottom"
                            data-bs-html="true">N</div>
                        <input :id="'note-' + item.n" class="col col-xs-1" type="number" :value="item.n"
                            @change="chooseN"></input>
                        <div class="col col-xs-2" title="ID of Phrase's Starting Note"
                            data-bs-customClass="custom-tooltip" data-bs-toggle="tooltip" data-bs-placement="bottom"
                            data-bs-html="true">Start ID</div>
                        <input :id="'start-id-' + item.n" class="col col-xs-2 start-id-sel" type="search"
                            list="noteIDS-datalist" :value="item.startid" @click="paintNoteSVG" @change="chooseNote"
                            @focus="setActiveInput"></input>
                        <div class="col col-xs-1" title="ID of Phrase's Ending Note"
                            data-bs-customClass="custom-tooltip" data-bs-toggle="tooltip" data-bs-placement="bottom"
                            data-bs-html="true">End ID</div>
                        <input :id="'end-id-' + item.n" class="col col-xs-2 end-id-sel" type="search"
                            list="noteIDS-datalist" :value="item.endid" @click="paintNoteSVG" @change="chooseNote"
                            @focus="setActiveInput"></input>
                        <div class="col col-xs-1" title="Type of Phrase (A/B, etc)" data-bs-customClass="custom-tooltip"
                            data-bs-toggle="tooltip" data-bs-placement="bottom" data-bs-html="true">Type</div>
                        <input :id="'type-' + item.n" class="col col-xs-2" type="search" list="typePhrases"
                            :value="item.type" @change="chooseType"></input>
                        <div class="phrase-controls col col-xs-2">
                            <!-- Visibility toggle -->
                            <button class="visibility-btn"
                                @click="item.visible = !item.visible; togglePhraseVisibility(item.n, item.visible)">
                                <svg-icon v-if="item.visible" :path="closedEyeIcon" size="20" viewbox="0 0 32 32"
                                    :style="'color: ' + item.color + '; stroke-width: 2%; fill: null;'"></svg-icon>
                                <svg-icon v-else :path="openEyeIcon" size="20" viewbox="0 0 15 15"
                                    :style="'color: ' + item.color + '; stroke-width: 2%; fill: null;'"></svg-icon>
                            </button>
                            <!-- Color picker -->
                            <input type="color" v-model="item.color" class="color-picker"
                                @input="togglePhraseVisibility(item.n, item.visible)" />
                        </div>
                        <a @click="deletePhrase(item.n)" class="col col-xs-1 btn btn-danger btn-sm del-btn"
                            type="button" title="Delete Phrase" data-bs-customClass="custom-tooltip"
                            data-bs-toggle="tooltip" data-bs-placement="bottom" data-bs-html="true">
                            <svg-icon :path="thrashIcon" size="20" viewbox="0 0 28 28" style="color: white"></svg-icon>
                        </a>
                        <datalist id="noteIDS-datalist">
                            <option v-for="itemX in noteIDS" :value="itemX"></option>
                        </datalist>
                        <datalist id="typePhrases">
                            <option v-for="itemX in ['A', 'A1', 'A2', 'B', 'B1', 'B2', 'C', 'CODA', 'INTRO']"
                                :value="itemX"></option>
                        </datalist>
                    </div>
                </draggable>
            </div>
        </div>
        <MusicalScore id="phraseForm" :vT="vT" :meiTree="MEIData" showIds="true" @rendered="addNoteClickHandlers" />
        <Teleport to="body">
            <modal :show="showModal" @close="showModal = false">
                <template #header>
                    <h3>Saved Phrase Structure to MEI File</h3>
                </template>
                <template #body>
                    <pre class="w-100" id="MEI-Modal-Segmentation">{{ SegmentationOntMEI }}</pre>
                </template>
            </modal>
        </Teleport>
        <Teleport to="body">
            <modal :show="showPhrasesModal" @close="showPhrasesModal = false">
                <template #header>
                    <h3>Proposed Phrase Structure</h3>
                </template>
                <template #body>
                    <pre class="w-100" id="MEI-Modal-Segmentation-Proposal">
                        <div v-for="item in automaticSegmentationData" class="row phrase-sel disabled-phrase-sel">
                            <div class="col col-xxs-1" title="Number of the Phrase (min: 1)" data-bs-customClass="custom-tooltip"
                                data-bs-toggle="tooltip" data-bs-placement="bottom" data-bs-html="true">N</div>
                            <input :id="'pnote-' + item.n" class="col col-xxs-1" type="number" :value="item.n"
                                @change="chooseN" disabled></input>
                            <div class="col col-xs-2" title="ID of Phrase's Starting Note" data-bs-customClass="custom-tooltip"
                                data-bs-toggle="tooltip" data-bs-placement="bottom" data-bs-html="true">Start ID</div>
                            <input :id="'p-start-id-' + item.n" class="col col-xs-2" type="search" list="p-noteIDS-datalist"
                                :value="item.startid" @click="paintNoteSVG" @change="chooseNote" disabled></input>
                            <div class="col col-xs-1" title="ID of Phrase's Ending Note" data-bs-customClass="custom-tooltip"
                                data-bs-toggle="tooltip" data-bs-placement="bottom" data-bs-html="true">End ID</div>
                            <input :id="'p-end-id-' + item.n" class="col col-xs-2" type="search" list="p-noteIDS-datalist"
                                :value="item.endid" @click="paintNoteSVG" @change="chooseNote" disabled></input>
                            <div class="col col-xxs-1" title="Type of Phrase (A/B, etc)" data-bs-customClass="custom-tooltip"
                                data-bs-toggle="tooltip" data-bs-placement="bottom" data-bs-html="true">Type</div>
                            <input :id="'p-type-' + item.n" class="col col-xxs-1" type="search" list="typePhrases"
                                :value="item.type" @change="chooseType" disabled></input>
                            <datalist id="p-noteIDS-datalist">
                                <option v-for="itemX in noteIDS" :value="itemX"></option>
                            </datalist>
                            <datalist id="typePhrases">
                                <option v-for="itemX in ['A', 'A1', 'A2', 'B', 'B1', 'B2', 'C', 'CODA', 'INTRO']"
                                    :value="itemX"></option>
                            </datalist>
                        </div>
                    </pre>
                </template>
                <template #footer>
                    <button class="modal-default-button btn btn-success"
                        @click="phraseSegmentData[0].value = automaticSegmentationData; phraseSegmentData[0].automatic = true; showPhrasesModal = false">
                        <svg-icon :path="saveIcon" size="20" viewbox="0 0 38 38" style="color: white"></svg-icon>
                    </button>
                    <button class="modal-default-button btn btn-warning" @click="showPhrasesModal = false">
                        <svg-icon :path="cancelIcon" size="20" viewbox="0 0 38 38" style="color: white"></svg-icon>
                    </button>
                </template>
            </modal>
        </Teleport>
    </div>
</template>

<script type="module">

import { Tooltip } from 'bootstrap';
import { VueDraggableNext as draggable } from 'vue-draggable-next';

import Modal from '../Modal.vue';
import MusicalScore from '../MusicalScore.vue';

export default {
    inject: ['getXpathNode', 'prettifyXml', 'createNodesMethods', 'updateNodesMethods', 'getAutomaticSegmentation'],
    components: {
        MusicalScore,
        Tooltip,
        Modal,
        draggable,
    },
    props: ['MEIData', 'vT', 'export'],
    emits: ["saveFinished"],
    data() {
        return {
            phraseSegmentData: [
                { name: 'phrases', tag: './/mei:music//mei:section//mei:supplied[@type="phrases"]', value: [], on_display: 'Phrases', automatic: false },
            ],
            automaticSegmentationData: [],
            noteIDS: [],
            activeInput: null,
            showModal: false,
            showPhrasesModal: false,
            SegmentationOntMEI: '',
            thrashIcon: "M 10 2 L 9 3 L 5 3 C 4.4 3 4 3.4 4 4 C 4 4.6 4.4 5 5 5 L 7 5 L 17 5 L 19 5 C 19.6 5 20 4.6 20 4 C 20 3.4 19.6 3 19 3 L 15 3 L 14 2 L 10 2 z M 5 7 L 5 20 C 5 21.1 5.9 22 7 22 L 17 22 C 18.1 22 19 21.1 19 20 L 19 7 L 5 7 z M 9 9 C 9.6 9 10 9.4 10 10 L 10 19 C 10 19.6 9.6 20 9 20 C 8.4 20 8 19.6 8 19 L 8 10 C 8 9.4 8.4 9 9 9 z M 15 9 C 15.6 9 16 9.4 16 10 L 16 19 C 16 19.6 15.6 20 15 20 C 14.4 20 14 19.6 14 19 L 14 10 C 14 9.4 14.4 9 15 9 z",
            saveIcon: "M9.467 3.106h3.030c0.266 0 0.482 0.278 0.482 0.621v4.374c0 0.343-0.216 0.621-0.482 0.621h-3.030c-0.266 0-0.482-0.278-0.482-0.621v-4.374c0-0.343 0.216-0.621 0.482-0.621zM21.964 14.524l7.987 7.987h-3.993v7.987h-7.987v-7.987h-3.993l7.987-7.987zM17.422 24.768h-10.309c-0.276 0-0.499-0.223-0.499-0.499v-11.481c0-0.276 0.223-0.499 0.499-0.499h17.721c0.276 0 0.499 0.223 0.499 0.499v4.402l3.12 3.12v-13.262c0-0.276-4.216-4.493-4.493-4.493h-4.741c0.169 0.069 0.287 0.223 0.287 0.404v6.181c0 0.244-0.216 0.442-0.482 0.442h-10.594c-0.266 0-0.482-0.198-0.482-0.442v-6.181c0-0.18 0.118-0.335 0.287-0.404h-4.242c-0.277 0-0.499 0.223-0.499 0.499v23.961c0 0.277 0.223 0.499 0.499 0.499h13.429v-2.746zM26.506 22.903v4.611h1.449c0.277 0 0.499-0.223 0.499-0.499v-4.111l-1.948 0z",
            cancelIcon: "M10.771 8.518c-1.144 0.215-2.83 2.171-2.086 2.915l4.573 4.571-4.573 4.571c-0.915 0.915 1.829 3.656 2.744 2.742l4.573-4.571 4.573 4.571c0.915 0.915 3.658-1.829 2.744-2.742l-4.573-4.571 4.573-4.571c0.915-0.915-1.829-3.656-2.744-2.742l-4.573 4.571-4.573-4.571c-0.173-0.171-0.394-0.223-0.657-0.173v0zM16 1c-8.285 0-15 6.716-15 15s6.715 15 15 15 15-6.716 15-15-6.715-15-15-15zM16 4.75c6.213 0 11.25 5.037 11.25 11.25s-5.037 11.25-11.25 11.25-11.25-5.037-11.25-11.25c0.001-6.213 5.037-11.25 11.25-11.25z",
            openEyeIcon: "M7.5 11C4.80285 11 2.52952 9.62184 1.09622 7.50001C2.52952 5.37816 4.80285 4 7.5 4C10.1971 4 12.4705 5.37816 13.9038 7.50001C12.4705 9.62183 10.1971 11 7.5 11ZM7.5 3C4.30786 3 1.65639 4.70638 0.0760002 7.23501C-0.0253338 7.39715 -0.0253334 7.60288 0.0760014 7.76501C1.65639 10.2936 4.30786 12 7.5 12C10.6921 12 13.3436 10.2936 14.924 7.76501C15.0253 7.60288 15.0253 7.39715 14.924 7.23501C13.3436 4.70638 10.6921 3 7.5 3ZM7.5 9.5C8.60457 9.5 9.5 8.60457 9.5 7.5C9.5 6.39543 8.60457 5.5 7.5 5.5C6.39543 5.5 5.5 6.39543 5.5 7.5C5.5 8.60457 6.39543 9.5 7.5 9.5Z",
            closedEyeIcon: "M22 16a1 1 0 10-2 0h2zm-6 4a1 1 0 100 2v-2zm-6-4a1 1 0 102 0h-2zm6-4a1 1 0 100-2v2zm-2.776 11.68a1 1 0 00-.448 1.95l.448-1.95zm-7.9-2.007a1 1 0 001.351-1.475l-1.35 1.475zM19.242 8.436a1 1 0 00.518-1.932l-.518 1.932zm7.358 1.822a1 1 0 10-1.34 1.484l1.34-1.484zM28 16c0 .464-.243 1.203-.853 2.116-.593.888-1.471 1.845-2.578 2.727C22.351 22.611 19.314 24 16 24v2c3.866 0 7.329-1.611 9.816-3.593 1.246-.993 2.271-2.099 2.994-3.18C29.515 18.172 30 17.037 30 16h-2zM4 16c0-.464.243-1.203.853-2.116.593-.888 1.471-1.845 2.578-2.727C9.649 9.389 12.686 8 16 8V6c-3.866 0-7.329 1.611-9.816 3.593-1.246.993-2.271 2.098-2.994 3.18C2.485 13.828 2 14.963 2 16h2zm16 0a4 4 0 01-4 4v2a6 6 0 006-6h-2zm-8 0a4 4 0 014-4v-2a6 6 0 00-6 6h2zm4 8c-.952 0-1.881-.114-2.776-.32l-.448 1.95c1.031.236 2.111.37 3.224.37v-2zm0-16c1.118 0 2.205.158 3.24.436l.52-1.932A14.489 14.489 0 0016 6v2zm9.258 3.742c.899.812 1.6 1.655 2.071 2.427.482.79.671 1.423.671 1.831h2c0-.928-.389-1.93-.963-2.872-.586-.962-1.42-1.95-2.438-2.87l-1.34 1.484zM6.675 20.198c-.878-.804-1.563-1.636-2.021-2.395C4.184 17.024 4 16.403 4 16H2c0 .917.38 1.906.941 2.836.573.95 1.389 1.926 2.384 2.837l1.35-1.475z",
        }
    },
    mounted() {
        this.getInfoFromMEI();
        const tooltipTriggerList = Array.from(document.querySelectorAll('[data-bs-toggle="tooltip"]'));
        tooltipTriggerList.forEach(tooltipTriggerEl => {
            new Tooltip(tooltipTriggerEl, {
                customClass: 'custom-tooltip',
                animated: 'fade',
                placement: 'bottom',
                trigger: 'hover'
            });
        });
        this.addNoteClickHandlers();
    },
    watch: {
        export: function (newVal, oldVal) {
            if (newVal != oldVal) {
                this.saveToMEI(false);
            }
        }
    },
    methods: {
        saveToMEI(openModal = true) {
            let phraseNode = this.getXpathNode(this.MEIData, './/mei:music//mei:section//mei:supplied[@type="phrases"]');
            if (!phraseNode) {
                this.createNodesMethods(this.MEIData, 'segmentation');
            }
            this.updateNodesMethods(this.MEIData, this.phraseSegmentData, 'segmentation');

            this.SegmentationOntMEI = this.prettifyXml(new XMLSerializer().serializeToString(this.getXpathNode(this.MEIData, './/mei:music//mei:section//mei:supplied[@type="phrases"]')));

            if (openModal) {
                this.showModal = !this.showModal;
            } else {
                this.$emit("saveFinished", "SegmentationForm");
            }
        },
        getAutomaticPhrases() {
            const segments = this.getAutomaticSegmentation(this.vT, this.MEIData);
            this.automaticSegmentationData = segments.map((item, i) => ({ 'n': i + 1, 'startid': item.startNoteID, 'endid': item.endNoteID, 'type': item.label, 'visible': false, 'rectElement': [], 'color': '#e66465' }));
            this.showPhrasesModal = true;
        },
        getInfoFromMEI() {
            this.noteIDS = this.vT.getDescriptiveFeatures()['pitchesIds'].flat().flat();
            this.phraseSegmentData.forEach(item => {
                let node = this.getXpathNode(this.MEIData, item.tag);
                if (node) {
                    for (let i in node.children) {
                        let phraseN = node.children[i];
                        if (phraseN.tagName == 'phrase') {
                            item.value.push({ 'n': phraseN.getAttribute('n'), 'startid': phraseN.getAttribute('startid').replace('#', ''), 'endid': phraseN.getAttribute('endid').replace('#', ''), 'type': phraseN.getAttribute('type'), 'visible': false, 'rectElement': [], 'color': '#e66465' })
                        }
                    };
                } else {
                    this.addPhrase();
                }
            });
        },
        deletePhrase(n) {
            this.phraseSegmentData[0].value = this.phraseSegmentData[0].value.filter(function (el) { return el.n != n; });
        },
        addPhrase() {
            let lastPhrase = this.phraseSegmentData[0].value[this.phraseSegmentData[0].value.length - 1];
            if (lastPhrase == null) {
                lastPhrase = { 'n': 0 };
            }
            this.phraseSegmentData[0].value.push({ 'n': parseInt(lastPhrase['n']) + 1, 'startid': this.noteIDS[0], 'endid': this.noteIDS[this.noteIDS.length - 1], 'type': 'A', 'visible': false, 'rectElement': [], 'color': '#e66465' });
        },
        setActiveInput(event) {
            this.activeInput = event.target.id; // e.g. "start-id-3"

            // 1️⃣ Remove previous color highlights
            const allInputs = document.querySelectorAll(".start-id-sel, .end-id-sel");
            allInputs.forEach(input => {
                input.classList.remove("active-start");
                input.classList.remove("active-end");
            });

            // 2️⃣ Apply color to the newly selected input
            if (event.target.id.startsWith("start")) {
                event.target.classList.add("active-start");
            } else if (event.target.id.startsWith("end")) {
                event.target.classList.add("active-end");
            }
        },
        addNoteClickHandlers() {
            this.$nextTick(() => {
                const notes = document.querySelectorAll('.note'); // Verovio note class
                notes.forEach(n => {
                    n.addEventListener('click', () => {
                        this.onNoteClick(n.id);
                    });
                });
            });
        },
        onNoteClick(noteId) {
            if (!this.activeInput) return;

            const input = document.getElementById(this.activeInput);
            if (!input) return;

            input.value = noteId;

            // Update internal data model
            if (this.activeInput.includes('start-id-')) {
                const n = this.activeInput.replace('start-id-', '');
                this.phraseSegmentData[0].value.find(el => el.n == n).startid = noteId;
            } else if (this.activeInput.includes('end-id-')) {
                const n = this.activeInput.replace('end-id-', '');
                this.phraseSegmentData[0].value.find(el => el.n == n).endid = noteId;
            }

            this.paintNoteSVG({ target: input });
        },
        paintNoteSVG(event) {
            let svgClass = event.target.id.includes('start') ? "select-start" : "select-end";
            let notesOnSVG = document.getElementsByClassName('note');
            Array.from(notesOnSVG).forEach((e, _) => {
                if (e.id === event.target.value && !e.classList.contains('bounding-box')) {
                    e.classList.add(svgClass);
                } else {
                    e.classList.remove(svgClass);
                }
            });
        },
        cleanPaintSVG() {
            this.activeInput = null;
            // 1️⃣ Remove previous color highlights
            const allInputs = document.querySelectorAll(".start-id-sel, .end-id-sel");
            allInputs.forEach(input => {
                input.classList.remove("active-start");
                input.classList.remove("active-end");
            });

            let notesOnSVG = document.getElementsByClassName('note');
            Array.from(notesOnSVG).forEach((e, _) => {
                e.classList.remove("select-start");
                e.classList.remove("select-end");
            });
        },
        drawPhraseRect(phrase) {
            if (!phrase.startid || !phrase.endid) return;
            if (!phrase.visible) return;

            if (phrase.rectElement) {
                phrase.rectElement.forEach((el) => el.remove());
                phrase.rectElement = null;
            }

            const svg = document.querySelector("#phraseForm-score-div-img svg svg");
            if (!svg) return;

            const notesPhrase = [...this.noteIDS].slice([...this.noteIDS].findIndex((element) => element == phrase.startid), [...this.noteIDS].findIndex((element) => element == phrase.endid) + 1);

            function hexToRgb(hex) {
                // Expand shorthand form (e.g. "03F") to full form (e.g. "0033FF")
                var shorthandRegex = /^#?([a-f\d])([a-f\d])([a-f\d])$/i;
                hex = hex.replace(shorthandRegex, function (m, r, g, b) {
                    return r + r + g + g + b + b;
                });

                var result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
                return result ? {
                    r: parseInt(result[1], 16),
                    g: parseInt(result[2], 16),
                    b: parseInt(result[3], 16)
                } : null;
            };
            let phraseC = hexToRgb(phrase.color);

            phrase.rectElement = [];
            notesPhrase.forEach((n) => {
                const el = svg.querySelector(`#${n}`);
                const box = el.getBBox();

                const rect = document.createElementNS("http://www.w3.org/2000/svg", "rect");
                rect.setAttribute("x", box.x + 500);
                rect.setAttribute("y", box.y - 10);
                rect.setAttribute("width", box.width);
                rect.setAttribute("height", box.height);
                rect.setAttribute("fill", "rgba("+phraseC.r+","+phraseC.g+","+phraseC.b+",0.2)");
                rect.setAttribute("stroke", phrase.color);
                rect.setAttribute("stroke-width", "2");
                rect.classList.add("phrase-box");

                svg.appendChild(rect);
                phrase.rectElement.push(rect);
            })
        },
        togglePhraseVisibility(phraseId, checked) {
            const phrase = this.phraseSegmentData[0].value.find(function (el) { return el.n == phraseId });
            if (!phrase) return;

            phrase.visible = checked;
            if (checked) {
                this.drawPhraseRect(phrase);
            } else {
                if (phrase.rectElement) {
                    phrase.rectElement.forEach((el) => el.remove());
                    phrase.rectElement = null;
                }
            }
        },
        chooseNote(event) {
            if (event.target.id.includes('start')) {
                let phrase = this.phraseSegmentData[0].value.filter(function (el) { return el.n == event.target.id.replace('start-id-', ''); })[0];
                phrase.startid = event.target.value;
                if (phrase.visible) {
                    this.drawPhraseRect(phrase);
                }
            } else {
                let phrase = this.phraseSegmentData[0].value.filter(function (el) { return el.n == event.target.id.replace('end-id-', ''); })[0];
                phrase.endid = event.target.value;
                if (phrase.visible) {
                    this.drawPhraseRect(phrase);
                }
            };

            this.paintNoteSVG(event);
        },
        chooseN(event) {
            this.phraseSegmentData[0].value.filter(function (el) { return el.n == event.target.id.replace('note-', ''); })[0].n = parseInt(event.target.value);
        },
        chooseType(event) {
            this.phraseSegmentData[0].value.filter(function (el) { return el.n == event.target.id.replace('type-', ''); })[0].type = event.target.value;
        }
    },
};
</script>

<style scoped>
.phrase-sel {
    align-items: center;
    font-size: small;
    margin-bottom: .1em;
    border: 1px solid grey;
}

.del-btn {
    margin: .1em .5em .1em 1.5em;
    padding: .2em;
}

.btn-automatic {
    margin-left: 1em;
    background-color: lightblue;
}

.col-xxs-1 {
    max-width: 5%;
}

.disabled-phrase-sel {
    max-width: 99%;
}

option:hover {
    background-color: red;
}

#MEI-Modal-Segmentation-Proposal {
    display: flex !important;
    flex-wrap: wrap;
    justify-content: center;
}

#MEI-Modal-Segmentation {
    max-height: 80% !important;
}

.active-start {
    color: red;
    font-weight: bold;
}

.active-end {
    color: green;
    font-weight: bold;
}

.phrase-controls {
    display: flex;
    align-items: center;
    gap: 8px;
}

.visibility-btn {
    background: none;
    border: none;
    cursor: pointer;
}

.visibility-btn:hover {
    color: #000;
}

.color-picker {
    width: 28px;
    height: 28px;
    padding: 0;
    border: none;
    background: none;
}
</style>
