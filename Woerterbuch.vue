<template>
    <div class="woerterbuch">
        <div v-if="suchart==='georges'" class="woerterbuch__title">
            <p class="woerterbuch__title--head text-3xl bold">Ausführliches lateinisch-deutsches Handwörterbuch</p>
            <p class="woerterbuch__title--sub text-base italic">ausgearbeitet von</p>
            <p class="woerterbuch__title--sub text-lg italic bold">Karl Ernst Georges</p>
            <p class="woerterbuch__title--sub text-base italic">Hannover und Leipzig 1940</p>

        </div>
        <div class="woerterbuch__container">
            <label v-if="suchart==='hermeneus'" class="woerterbuch__suche--label">Suche lateinisches Wort oder Bedeutung </label>
            <div class="woerterbuch__suche--container-both">
                <div class="woerterbuch__suche--suchart-selector">
                    <div class="woerterbuch__suche--suchart-selector__hermeneus"
                         @click="setActiveSuchart('hermeneus')"
                         v-bind:class="{'woerterbuch__suche--suchart-active': isActiveSuchart('hermeneus')}">Standardsuche
                    </div>
                    <div class="woerterbuch__suche--suchart-selector__georges"
                         @click="setActiveSuchart('georges')"
                         v-bind:class="{'woerterbuch__suche--suchart-active': isActiveSuchart('georges')}">Georges (1913)
                    </div>
                </div>
                <div class="woerterbuch__suche--container">
                    <!--SUCHFELD-->
                    <input class="woerterbuch__suche--input"
                           name="woerterbuch__suche--input"
                           placeholder="Lateinisches Wort oder Bedeutung eingeben..."
                           @keydown.prevent.enter="searchGeorges()"
                           v-model="search">
                    <button v-if="this.suchart==='georges'" class="woerterbuch__suche--button" @click="this.searchGeorges">
                        <loading-dots v-if="this.isSearchingGeorges" :color="'white'"></loading-dots>
                        <div v-else class="woerterbuch__suche--button__description">
                            <div class="woerterbuch__suche--button__description--icon"></div>
                            Suchen
                        </div>

                    </button>
                </div>
            </div>
            <div class="woerterbuch__suche--options" v-if="this.suchart==='georges'">
                <input type="checkbox" id="woerterbuch__suche--options__allExpanded" v-model="allExpanded">
                <label for="woerterbuch__suche--options__allExpanded">Komplette Einträge anzeigen</label>
            </div>
        </div>


        <!--ERGEBNISSE-->
        <!--ERGEBNISSE: GEORGES-->

        <div
                v-if="this.suchart==='georges' && this.results_georges"
                class="woerterbuch__ergebnisse__georges_result-container">
            <div>{{this.results_georges.length}} Ergebnisse:</div>
            <br><br>
            <div v-for="result in this.results_georges"
                 class="woerterbuch__ergebnisse__georges_entry-container georges--global"
                 :class="{'woerterbuch__ergebnisse__georges_entry__is-expanded': allExpanded,
                          'woerterbuch__ergebnisse__georges_entry__is-collapsed': allCollapsed}">
                <div class="woerterbuch__ergebnisse__georges_header" @click.prevent="toggleShow($event.currentTarget.parentNode)">
                    <div v-html="result.headword" class="woerterbuch__ergebnisse__georges_header--headword"></div>
                    <div v-html="highlight(result.senses)" class="woerterbuch__ergebnisse__georges_header--senses"></div>
                    <button class="woerterbuch__ergebnisse__georges_header--button-expand">

                        <span class="woerterbuch__ergebnisse__georges_header--button-expand__icon"></span>

                    </button>
                </div>
                <div v-show="result.errors" class="woerterbuch__ergebnisse__error-note">Dieser Eintrag enthält möglicherweise formale Fehler.</div>
                <div v-html="highlight(result.entry)" class="woerterbuch__ergebnisse__georges_entry" @click.prevent="toggleShowEntryClick($event.currentTarget.parentNode)"></div>
                <div v-if="!result.errors" class="woerterbuch__ergebnisse__georges_entry--bottom-buttons">
                    <span>Eintrag als fehlerhaft markieren</span>
                    <button @click="markError(result.id, $event.currentTarget)">🔧</button>
                </div>
                <div class="woerterbuch__ergebnisse__georges_entry__shade" @click.prevent="toggleShowEntryClick($event.currentTarget.parentNode)"></div>
            </div>
            <div v-if="this.results_georges.length === 0">
                Leider wurden keine Ergebnisse zum Stichwort '{{this.search}}' gefunden!
            </div>
            <div v-if="!this.results_georges">
                Keine Datenbankverbindung! Suche fehlgeschlagen
            </div>
        </div>

        <!--ERGEBNISSE: HERMENEUS-->
        <div v-if="this.suchart==='hermeneus'">
            <div v-if="hasResults()" class="woerterbuch__ergebnisse_container">
                <div class="woerterbuch__ergebnisse__header">
                    <label>Ergebnisse: </label>

                    <div class="woerterbuch__ergebnisse__header--column-titles">
                        <span class="woerterbuch__ergebnisse__header--column-titles__lemma">Lateinisches Wort</span>
                        <span class="woerterbuch__ergebnisse__header--column-titles__wortart">Wortart</span>
                        <span class="woerterbuch__ergebnisse__header--column-titles__bedeutung">Bedeutung</span>
                    </div>
                </div>
                <div class="woerterbuch__ergebnisse__body--container">
                    <div class="woerterbuch__ergebnisse__body--container-sub">
                        <a
                                class="woerterbuch__ergebnisse__body--result-btn js-opens-modal"
                                v-for="wort in results"
                                href="#"
                                @click.prevent="viewVocab(wort)">

                            <div class="woerterbuch__ergebnisse__body--result-row">
                                <span class="woerterbuch__ergebnisse__body--result-lemma lemma" v-text="wort.lemma"></span>
                                <span class="woerterbuch__ergebnisse__body--result-wortart" v-text="wort.wortart.charAt(0).toUpperCase() + wort.wortart.substr(1)"></span>
                                <span class="woerterbuch__ergebnisse__body--result-bedeutung" v-text="wort.bedeutung"></span>
                            </div>


                        </a>
                    </div>
                </div>

            </div>
        </div>
        <!--LOADER-->
        <loading-dots v-if="isSearching()"></loading-dots>

    </div>
</template>

<script>

    import LoadingDots from "../loader/LoadingDots";

    export default {
        name: "woerterbuch",
        data: function () {
            return {
                search: '',
                suchart: this.suchart_preconfig || 'hermeneus',
                results_georges: false,
                isSearchingGeorges: false,
                allExpanded: false,
            };
        },
        components: {
            'loading-dots': LoadingDots
        },
        props: ['vocab', 'suchart_preconfig'],
        computed: {
            /**
             *
             */
            results () {
                return this.vocab.filter(wort => {
                    let string_match = new RegExp(this.search, 'gi');
                    return wort.lemma.match(string_match) !== null || wort.bedeutung.match(string_match) !== null;
                });
            },
            allCollapsed: function () {
                return !this.allExpanded
            },


        },
        methods: {
            highlight (content) {

                if (!this.search) {
                    return content;
                }
                try {
                    return content.replace(new RegExp("(?<!\"|')" + this.search, "gi"), match => {
                        return '<span class="highlighted_search">' + match + '</span>';
                    });
                } catch (e) {
                    return content;
                }
            },
            markError (id, buttonElement) {

                axios.get('/georges/markerror/' + id, {})
                     .then((response) => {
                         buttonElement.parentNode.innerHTML = "Dankeschön!";
                     })
                     .catch((error) => {
                     });
            },
            searchGeorges () {
                this.results_georges = false;
                if (this.suchart === "georges") {
                    this.isSearchingGeorges = true;
                    axios.get('/georges/search-scout/' + this.search, {})
                         .then((response) => {
                             this.isSearchingGeorges = false;
                             this.results_georges = response.data;
                         })
                         .catch((error) => {
                             this.isSearchingGeorges = false;
                             this.results_georges = false;
                         });

                }
            },

            isActiveSuchart (suchart) {
                return suchart === this.suchart;
            },

            /*            toggleShow (event) {
                            event.currentTarget.parentNode.parentNode.classList.toggle('woerterbuch__ergebnisse__georges_entry__is-collapsed');
                            event.currentTarget.parentNode.parentNode.classList.toggle('woerterbuch__ergebnisse__georges_entry__is-expanded');

                        },*/
            toggleShow (Element) {
                Element.classList.toggle('woerterbuch__ergebnisse__georges_entry__is-collapsed');
                Element.classList.toggle('woerterbuch__ergebnisse__georges_entry__is-expanded');

            },
            toggleShowEntryClick (Element) {
                if (Element.classList.contains('woerterbuch__ergebnisse__georges_entry__is-collapsed')) {
                    this.toggleShow(Element);
                }

            },
            setActiveSuchart (suchart) {
                this.suchart = suchart;
            },
            /**
             * Condition for loading animation
             */
            isSearching () {
            },
            /**
             *
             * @returns {boolean}
             */
            hasResults () {
                return this.search !== '' && this.results.length > 0;
            },
            /**
             *
             * @param wort
             */
            viewVocab (wort) {
                this.openModal(wort);
            },
            /**
             *
             * @param wort
             */
            openModal (wort) {
                let ModalData = {
                    title: 'Vokabelansicht: ' + wort.lemma,
                    type: 'success',
                    size: 'fullscreen',
                    component: 'vocab-viewer',
                    uri: '/glossarium/' + wort.route_name + '/' + wort.id,
                };
                EventBus.$emit('OpenModalEvent', JSON.stringify(ModalData));
            }
        },

    }
</script>

<style>


    .woerterbuch_standalone {
        /*    border: 10px solid red;*/
        background-color: #fff;
        width: 90%;
        margin: auto;
    }

    .highlighted_search {
        background: yellow;
    }

    .woerterbuch__title {
        font-family: 'Merriweather';
        width: 70%;
        text-align: center;
        color: #aaa;
        height: auto;
        margin: auto auto 3rem auto;
    }

    .woerterbuch__title p {
        margin-top: 0.5rem;
    }
    .woerterbuch__title--head {
        line-height: 2.5rem;
    }
    .woerterbuch {

        margin: 3rem auto auto auto;
    }

    .woerterbuch__ergebnisse__error-note {
        text-align: center;
        color: #B22E2F;
        font-size: 0.8rem;
    }

    .woerterbuch__ergebnisse__georges_entry orth {
        font-family: 'Merriweather';
        font-weight: bold;

    }

    .woerterbuch__ergebnisse__georges_header {
        display: flex;
        justify-content: space-between;
    }

    .woerterbuch__ergebnisse__georges_header--headword {
        /*font-family: 'Noto Serif';*/
        font-weight: bold;
        padding-left: 0.5rem;
        display: inline-block;
        margin-bottom: 0.5rem;
        font-size: 1rem;
        width: 20%;
    }

    .woerterbuch__ergebnisse__georges_header--senses {
        width: 70%;
        font-size: 0.8rem;
    }

    .woerterbuch__ergebnisse__georges_header--button-expand {
        width: 10%;
        background: none;
        color: #B22E2F;
        font-size: 1.5rem;
        text-align: right;
    }

    .woerterbuch__ergebnisse__georges_result-container {
        width: 90%;
        margin: auto;
        padding: 1rem;

    }

    .woerterbuch__ergebnisse__georges_entry-container {
        padding-top: 1rem;
        margin-top: 2rem;
        border-top: 1px solid #ccc;
        position: relative;
    }

    .woerterbuch__ergebnisse__georges_entry {
        padding: 1rem 0 1rem 0.5rem;
    }

    .woerterbuch__ergebnisse__georges_entry__is-expanded .woerterbuch__ergebnisse__georges_entry__shade {
        -webkit-box-shadow: none;
        -moz-box-shadow: none;
        box-shadow: none;
    }

    .woerterbuch__ergebnisse__georges_entry__is-collapsed .woerterbuch__ergebnisse__georges_entry__shade {
        position: absolute;
        bottom: 0;
        width: 100%;
        height: 3rem;
        -webkit-box-shadow: inset 0px -40px 40px -10px rgba(255, 255, 255, 1);
        -moz-box-shadow: inset 0px -40px 40px -10px rgba(255, 255, 255, 1);
        box-shadow: inset 0px -40px 40px -10px rgba(255, 255, 255, 1);
    }

    .woerterbuch__ergebnisse__georges_entry__is-collapsed orth {
        display: none;
    }

    .woerterbuch__ergebnisse__georges_entry__is-collapsed {
        height: 12rem;
        overflow: hidden;
        cursor: pointer;
    }

    .woerterbuch__ergebnisse__georges_entry__is-collapsed .woerterbuch__ergebnisse__georges_header--button-expand__icon:before {
        Content: '►';
        color: #999;
    }

    .woerterbuch__ergebnisse__georges_entry__is-expanded .woerterbuch__ergebnisse__georges_header--button-expand__icon:before {
        Content: '▼';
    }

    .woerterbuch__ergebnisse__georges_entry__is-collapsed.woerterbuch__ergebnisse__georges_entry-container:hover {
        background: #eee;
    }

    .woerterbuch__ergebnisse__georges_entry__is-collapsed .woerterbuch__ergebnisse__georges_entry {
        opacity: 0.4;
    }

    .woerterbuch__ergebnisse__georges_entry--bottom-buttons {
        display: flex;
        justify-content: flex-end;
    }

    .woerterbuch__ergebnisse__georges_entry--bottom-buttons button {
        background: none;
        margin-left: 1rem;
        font-size: 1.5rem;
    }

    .woerterbuch__ergebnisse__georges_entry--bottom-buttons button:hover {
        text-shadow: -1px -1px 5px rgba(150, 150, 150, 1);
    }

    .woerterbuch__ergebnisse__georges_entry--bottom-buttons:hover span {
        display: inline-block;
    }

    .woerterbuch__ergebnisse__georges_entry--bottom-buttons span {
        padding-top: 0.25rem;
        display: none;
        font-size: 1rem;
        font-style: italic;
        color: #666;
    }

    /**
     *  SUCHKOMPONENTEN
     */
    .woerterbuch {
    }

    .woerterbuch {
        padding: 0.75rem;
    }

    .woerterbuch .woerterbuch__container {
        padding: 0.5rem;
        min-height: 10rem;
        text-align: center;
    }

    .woerterbuch__container {
        width: 90%;
        margin: auto;
    }

    .woerterbuch__suche--label {
        display: block;
        margin-bottom: 1rem;
        font-size: 1.1rem;
        text-align: center;
    }

    .woerterbuch__suche--container-both {
        box-shadow: 0px 1px 3px #ddd;
        background: #fff;

    }

    .woerterbuch__suche--container {
        border: 1px solid #ddd;
        margin: auto auto 1rem auto;
        display: flex;
    }

    .woerterbuch__suche--suchart-selector {
        display: flex;
        margin: 0;
        padding: 0;
        font-size: 1.15rem;
        width: 100%;

        height: 2.5rem;
    }

    .woerterbuch__suche--suchart-selector__hermeneus, .woerterbuch__suche--suchart-selector__georges {
        padding: 0.5rem;
        border-top: 1px solid #ddd;
        border-left: 1px solid #ddd;
        border-right: 1px solid #ddd;
        width: 50%;
        cursor: pointer;
        color: #ccc;
    }

    .woerterbuch__suche--suchart-selector__hermeneus:hover, .woerterbuch__suche--suchart-selector__georges:hover {
        background: #efefef;
        color: #666;

    }

    .woerterbuch__suche--suchart-active {
        color: #666;
        background: #efefef;
        border-left: 1px solid #ddd;
        border-right: 1px solid #ddd;
        border-top: 3px solid #B22E2F;
    }

    .woerterbuch__suche--input {
        /*max-width: 75%;*/
        min-width: 15rem;
        height: 3rem;
        background: #eee;
        border: none;
        padding: 0.5rem;
        width: 100%;
    }

    .woerterbuch__suche--button {
        width: 20%;
        height: 3rem;
        padding: 0.5rem
    }

    .woerterbuch__suche--button:hover {
        background: #721E1E;
    }

    .woerterbuch__suche--button__description {
        display: flex;
        font-size: 1.5rem;
        font-weight: lighter;
        font-style: italic;
        justify-content: center;
    }

    .woerterbuch__suche--button__description--icon {
        margin: 0.25rem 0.5rem 0.25rem 0.25rem;
        width: 1.5rem;
        height: 1.5rem;
        background-image: url("../../hermeneus/public/img/svg/hermeneus-icon-glossarium.svg");
        background-repeat: no-repeat;
        background-size: contain;
        filter: brightness(400%) saturate(0%);
    }

    .woerterbuch__suche--radio-container input:checked + input:before {
        -webkit-transform: translateX(calc(100% - 1rem));
        -ms-transform: translateX(calc(100% - 1rem));
        transform: translateX(calc(100% - 1rem));
    }

    .woerterbuch__ergebnisse_container {
        padding: 0.5rem;
    }

    .woerterbuch__ergebnisse__header {
        text-align: left;
    }

    .woerterbuch__ergebnisse__header--column-titles {
        color: #B22E2F;
        padding: 0.5rem;
    }

    .woerterbuch__ergebnisse__header--column-titles span {
        text-align: center;
    }

    .woerterbuch__ergebnisse_container .woerterbuch__ergebnisse__header {
    }

    .woerterbuch__ergebnisse_container a {
        text-decoration: none;
    }

    .woerterbuch__ergebnisse__body--container {
        max-height: 25rem;
        overflow-y: scroll;
        overflow-x: hidden;
        animation-name: fadeIn;
        animation-duration: 0.4s;
    }

    .woerterbuch__ergebnisse_container a:hover {
        color: #B22E2F;
    }

    .woerterbuch__ergebnisse__body--result-wortart {
        font-style: italic;
    }

    .woerterbuch__ergebnisse__header--column-titles, .woerterbuch__ergebnisse__body--result-row {
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;
        overflow: hidden;
        text-align: left;
        padding: 0.25rem;
    }

    .woerterbuch__ergebnisse__body--result-lemma, .woerterbuch__ergebnisse__header--column-titles__lemma {
        width: 30%;
    }

    .woerterbuch__ergebnisse__body--container-sub {
        line-height: 1rem;
        padding: 0.25rem;
        margin: 0 auto 0 auto;
    }

    .woerterbuch__ergebnisse__body--container {
        border: 1px solid #eee;
    }

    .woerterbuch__ergebnisse__body--container a {
        display: block;
    }

    .woerterbuch__ergebnisse__body--container a:nth-child(odd) {
        background: #fff;
    }

    .woerterbuch__ergebnisse__body--container a:nth-child(even) {
        background: #eee;
    }

    .woerterbuch__ergebnisse__body--result-row:hover span {
        color: #B22E2F;
    }

    .woerterbuch__ergebnisse__body--result-wortart {
        color: #888;
        font-size: 0.8rem;
    }

    .woerterbuch__ergebnisse__body--result-bedeutung {
        display: inline-block;
        font-size: 0.8rem;
        line-height: 1rem;
        color: #000;
    }

    .woerterbuch__ergebnisse__body--container {
        color: #fff;
    }

    @media only screen and (min-width: 640px) {
        .woerterbuch {
            padding: 4rem;
        }

        .woerterbuch__container {
            padding: 2rem;
        }

        .woerterbuch__ergebnisse__header--column-titles, .woerterbuch__ergebnisse__body--container-sub {
            text-align: left;
            padding: 0.5rem;
        }

        .woerterbuch__ergebnisse__body--container-sub a span {
            line-height: 1.25rem;
            font-size: 1.10rem;
        }

        .woerterbuch__ergebnisse__header--column-titles__wortart, .woerterbuch__ergebnisse__body--result-wortart {
            width: 10%;
            display: inline-block;
        }

        .woerterbuch__ergebnisse__header--column-titles__bedeutung,
        .woerterbuch__ergebnisse__body--result-bedeutung {
            width: 40%;
        }
    }

    @media only screen and (min-width: 1024px) {
        .woerterbuch__suche--input {
            width: 100%;
        }
    }


</style>