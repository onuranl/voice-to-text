<script>
import Modal from '@/components/modal.vue'

import { setCORS } from "google-translate-api-browser";
const translate = setCORS("http://cors-anywhere.herokuapp.com/");

export default {
    name: 'IndexPage',
    components: {
        Modal,
    },
    data: () => ({
        command: 'hey translate',
        commanded: false,
        started: false,
        speaking: false,
        error: false,
        text: null,
        translated: null,
        languages: null,
        current: 'en',
        to: 'tr'
    }),
    created() {
        fetch('languages.json').then(response => response.json()).then(result => this.languages = result)
    },
    mounted() {
        this.startSpeechRecognition()
    },
    watch: {
        text(val) {
            translate(val, { current: this.current, to: this.to })
                .then(res => {
                    this.translated = res.text
                })
                .catch(err => {
                    this.error = err
                    console.error(err);
                });
        }
    },
    methods: {
        startSpeechRecognition() {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
            const recognition = SpeechRecognition ? new SpeechRecognition() : false

            if (!recognition) {
                return alert('Speech Recognition is not available on this browser. Please use Chrome or Firefox')
            }

            recognition.continuous = true
            // recognition.grammars
            recognition.interimResults = true
            // recognition.lang

            recognition.addEventListener('audioend', event => {
                console.log('audioend', event)
            })

            recognition.addEventListener('audiostart', event => {
                console.log('audiostart', event)
            })

            recognition.addEventListener('end', event => {
                console.log('end', event)
                this.error = 'end'
                this.started = false
            })

            recognition.addEventListener('error', event => {
                this.error = event.error
            })

            recognition.addEventListener('nomatch', event => {
                console.log('nomatch', event)
            })

            recognition.addEventListener('result', event => {
                const text = Array.from(event.results).map(result => result[0]).map(result => result.transcript).join('')

                console.log(text)

                if (!this.commanded) {
                    text.toLowerCase().search(this.command) > -1 ? this.commanded = true : null
                } else {
                    this.text = text
                }
            })

            recognition.addEventListener('soundend', event => {
                console.log('soundend', event)
            })

            recognition.addEventListener('soundstart', event => {
                console.log('soundstart', event)
            })

            recognition.addEventListener('speechend', event => {
                console.log('speechend', event)
                this.speaking = false
            })

            recognition.addEventListener('speechstart', event => {
                console.log('speechstart', event)
                this.speaking = true
            })

            recognition.addEventListener('start', event => {
                console.log('start', event)
                this.started = true
            })

            recognition.start()
        },
        textToSpeech() {
            const utterThis = new SpeechSynthesisUtterance()
            const synth = window.speechSynthesis

            utterThis.text = this.translated
            utterThis.lang = this.to

            synth.speak(utterThis)
        },
        changeCurrentLanguage(val) {
            this.current = val
        }
    }
}
</script>

<template>
    <div>
        <modal v-if="error" type="error" :error="error" />
        <div v-else>
            <modal v-if="!commanded" type="initial" :language="current" :languages="languages" :speaking="speaking"
                @changeCurrentLanguage="changeCurrentLanguage" />
            <c-flex v-else direction="column" align="center" justify="center">
                <c-flex align="center" justify="center" p="5">
                    <c-textarea :value="text" mr="5" placeholder="Your words" />
                    <c-flex align="center"> <c-icon name="arrow-forward" /> </c-flex>
                    <c-textarea :value="translated" ml="5" placeholder="Translated words" />
                </c-flex>
                <c-box v-if="languages">
                    <c-select v-model="to" :placeholder="languages.find(el => el.code === to).name">
                        <option v-for="item in languages" :key="item.code" :value="item.code"> {{ item.name }} </option>
                    </c-select>
                </c-box>
                <div @click="textToSpeech()">speak</div>

            </c-flex>
        </div>
    </div>
</template>