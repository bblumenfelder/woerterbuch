<template>
    <div>
        <div class="woerterbuch__container">
            <label class="woerterbuch__suche--label">Suche lateinisches Wort oder Bedeutung </label>
            <div class="woerterbuch__suche--container flex_container_space_around">
                <!--SUCHFELD-->
                <input class="woerterbuch__suche--input input_text"
                       name="woerterbuch__suche--input"
                       v-model="suchbegriff">

                <!--RADIO-GROUP SUCHART-->
                <div class="woerterbuch__suche--radio-group radio_group">
                    <div class="woerterbuch__suche--radio-container radio">
                        <label class='woerterbuch__suche--radio-label radio_element'>
                            <input class="woerterbuch__suche--radio__schnellsuche" type="radio" name="suche_art" value="schnellsuche" checked>
                            <span class="woerterbuch__suche--radio-title radio_label"> Schnellsuche</span>
                        </label>
                    </div>
                    <div class="woerterbuch__suche--radio-container radio">
                        <label class='woerterbuch__suche--radio-label radio_element'>
                            <input  class="woerterbuch__suche--radio__volltextsuche" type="radio" name="suche_art" value="volltextsuche">
                            <span class="woerterbuch__suche--radio-title radio_label"> Volltextsuche</span>
                        </label>
                    </div>
                </div>
            </div>
        </div>

        <!--LOADER-->
        <vue-loading-dots v-if="results"></vue-loading-dots>

        <!--ERGEBNISSE-->
        <div class="woerterbuch__ergebnisse_container">
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
                       class="woerterbuch__ergebnisse__body--result-btn"
                       v-for="wort in results"
                       href="#"
                       @click.prevent="viewVocab(wort)"
                       v-bind:vocab_id="wort.id"
                       v-bind:route_name="wort.route_name">

                        <div class="woerterbuch__ergebnisse__body--result-row">
                            <span class="woerterbuch__ergebnisse__body--result-lemma" v-text="wort.lemma"></span>
                            <span class="woerterbuch__ergebnisse__body--result-wortart" v-text="wort.wortart"></span>
                            <span class="woerterbuch__ergebnisse__body--result-bedeutung" v-text="wort.bedeutung"></span>
                        </div>

                    </a>
                </div>
            </div>
            <div class="woerterbuch__ergebnisse__scout">

                <!-- AJAX-->

            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "woerterbuch"
    }
</script>

<style scoped>
.suche-view{
    padding: 0.75rem;
}

.suche_container {
    padding: 0.5rem;
    min-height: 10rem;
    text-align: center;
}

.suche_container label {
    display: block;
    margin-bottom: 1rem;
    font-size: 1.1rem;
}

.radio_group {
    text-align: left;
    width: 15%;
    min-width: 10rem;
}

.radio {
    display: inline-block;
    margin-left: 0;
}

.radio label {
    font-size: 1rem;
    margin-bottom: 0;
}

.suche_switch {
    display: flex;
    justify-content: space-between;
}

.suche_switch_span:before {
    content: "Schnellsuche";
}

.suche_switch_span:before input:checked + .suche_switch_span:before {
    content: "Volltextsuche";
}

.suche_input_container {
    margin: auto;
}

.ipt_search_vocab {
    max-width: 75%;
    min-width: 15rem;
    margin-bottom: 1rem;
    height: 3rem;
    background: #eee;
    box-shadow: inset 0 2px 5px 0 #ddd;
    border: 1px solid #ddd;
}

.label_switch {
    width: 20%;
    height: 3rem;
}

.input_switch {
    background: #eee;
    box-shadow: inset 0 2px 5px 0 #ddd;
    border: 1px solid #ddd;
}

.input_switch:before {
    font-size: 0.8rem;
    overflow: hidden;
    color: #fff;
    background: #B22E2F;
    height: calc(2rem - 1px);
    width: 7rem;
    text-align: center;
    padding-top: 0.25rem;
    left: 0.5rem;
    bottom: 0.5rem;
}

.input_switch:before input:checked + .input_switch:before {
    -webkit-transform: translateX(calc(100% - 1rem));
    -ms-transform: translateX(calc(100% - 1rem));
    transform: translateX(calc(100% - 1rem));
}

.vocab_search_head {
    color: #B22E2F;
    padding: 0.5rem;
}

.vocab_search_head span {
    text-align: center;
}

.results_header {
    text-align: left;
}




</style>