<template>
    <div class="woerterbuch">
        <div class="woerterbuch__container">
            <label class="woerterbuch__suche--label">Suche lateinisches Wort oder Bedeutung </label>
            <div class="woerterbuch__suche--container flex_container_space_around">
                <!--SUCHFELD-->
                <input class="woerterbuch__suche--input input_text"
                       name="woerterbuch__suche--input"
                       placeholder="Lateinisches Wort oder Bedeutung eingeben..."
                       v-model="search">

                <!--RADIO-GROUP SUCHART-->
                <div class="woerterbuch__suche--radio-group radio_group">
                    <div class="woerterbuch__suche--radio-container radio">
                        <label class='woerterbuch__suche--radio-label radio_element'>
                            <input class="woerterbuch__suche--radio__schnellsuche" type="radio" name="suche_art" value="schnellsuche"
                                   checked>
                            <span class="woerterbuch__suche--radio-title radio_label"> Schnellsuche</span>
                        </label>
                    </div>
                    <div class="woerterbuch__suche--radio-container radio">
                        <label class='woerterbuch__suche--radio-label radio_element'>
                            <input class="woerterbuch__suche--radio__volltextsuche" type="radio" name="suche_art" value="volltextsuche">
                            <span class="woerterbuch__suche--radio-title radio_label"> Volltextsuche</span>
                        </label>
                    </div>
                </div>
            </div>
        </div>


        <!--ERGEBNISSE-->
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
                            <span class="woerterbuch__ergebnisse__body--result-wortart" v-text="wort.wortart"></span>
                            <span class="woerterbuch__ergebnisse__body--result-bedeutung" v-text="wort.bedeutung"></span>
                        </div>


                    </a>
                </div>
            </div>

        </div>
        <!--LOADER-->
        <vue-loading-dots v-if="isSearching()"></vue-loading-dots>

    </div>
</template>

<script>

    import {VueLoading} from "../loader/hermeneus.vue_loading";

    export default {
        name: "woerterbuch",
        data: function () {
            return {
                search: '',
            };
        },
        components: {
            'vue-loading-dots': VueLoading
        },
        props: ['vocab'],
        computed: {
            /**
             *
             */
            results () {
                return this.vocab.filter(wort => {
                    let string_match = new RegExp(this.search, 'gi');
                    return wort.lemma.match(string_match) !== null || wort.bedeutung.match(string_match) !== null;
                });
            }
        },
        methods: {
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
                    uri: '/glossarium/' + wort.route_name + '/' + wort.id,
                };
                EventBus.$emit('OpenModalEvent', JSON.stringify(ModalData));
            }
        },

    }
</script>

<style scoped>
    /**
     *  SUCHKOMPONENTEN
     */

    .woerterbuch {
        background: #fff;
    }

    .woerterbuch {
        padding: 0.75rem;
    }

    .woerterbuch .woerterbuch__container {
        padding: 0.5rem;
        min-height: 10rem;
        text-align: center;
    }

    .woerterbuch__suche--label {
        display: block;
        margin-bottom: 1rem;
        font-size: 1.1rem;
        text-align: center;
    }

    .woerterbuch__suche--radio-group {
        display: none;
        text-align: left;
        width: 15%;
        min-width: 10rem;
    }

    .woerterbuch__suche--radio-container.radio {
        display: inline-block;
        margin-left: 0;
    }

    .woerterbuch__suche--radio-label {
        font-size: 1rem;
        margin-bottom: 0;
    }

    .woerterbuch__suche--container {
        margin: auto;
    }

    .woerterbuch__suche--input {
        /*max-width: 75%;*/
        min-width: 15rem;
        margin-bottom: 1rem;
        height: 3rem;
        background: #eee;
        box-shadow: inset 0 2px 5px 0 #ddd;
        border: 1px solid #ddd;
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

    .woerterbuch__suche--input {
        width: 100%;
        box-shadow: 0px 1px 3px #ddd;
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