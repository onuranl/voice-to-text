<script>
export default {
    props: ["type", "speaking", "error", "language", "languages"],
    computed: {
        currentLang: {
            get() {
                this.language
            },
            set(val) {
                this.$emit('changeCurrentLanguage', val)
            }
        }
    },
}
</script>

<template>
    <c-modal :is-open="true" size="xl" is-centered>
        <c-modal-content ref="content">
            <c-modal-body>
                <div v-if="type === 'initial'">
                    <c-box align="center" justify="center">
                        <c-text fontSize="4xl">Say "Hey Translate" to start</c-text>
                        <c-box w="150px" my="3" v-if="languages && language">
                            <c-select v-model="currentLang"
                                :placeholder="languages.find(el => el.code === language).name">
                                <option v-for="item in languages" :key="item.code" :value="item.code">{{ item.name }}
                                </option>
                            </c-select>
                        </c-box>
                        <c-spinner v-if="speaking" thickness="4px" speed="0.65s" empty-color="green.200" color="vue.500"
                            size="xl" />
                    </c-box>
                </div>
                <div v-else-if="type === 'error'">
                    {{ typeof error === 'string' || 'object' ? error : 'Please be active your microphone permission' }}
                </div>
            </c-modal-body>
        </c-modal-content>
        <c-modal-overlay />
    </c-modal>
</template>