<template>
    <!-- About Card -->
    <a href="#" @click="show" class="navbar-item">Select BPM</a>
    <div :class="isActive ? 'modal is-active' : 'modal'">
        <div class="modal-background" @click="hide"></div>
        <div class="modal-card">
            <section class="modal-card-body">
                <div class="content">
                    <h1 class="title">Select BPM</h1>
                    <div class="buttons">
                        <button
                            class="button is-info is-large"
                            @click="select(40)"
                        >
                            40
                        </button>

                        <button
                            class="button is-info is-large"
                            @click="select(60)"
                        >
                            60
                        </button>

                        <button
                            class="button is-info is-large"
                            @click="select(80)"
                        >
                            80
                        </button>
                    </div>
                </div>
            </section>
            <footer class="modal-card-foot">
                <button class="button is-primary is-large" @click="hide">
                    Close
                </button>
            </footer>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

/** This is an about component for the metronome app
 */
export default defineComponent({
    components: {},
    props: {},
    data() {
        return {
            /** Whether the about modal is shown */
            isActive: false,
        }
    },
    methods: {
        show(): void {
            this.isActive = true
        },
        hide(): void {
            this.isActive = false
            console.debug('SelectBpm::hide')
            this.$emit('update:modelValue', null)
        },
        select(beatsPerMinute: number): void {
            this.isActive = false
            console.debug('SelectBpm::select: ', beatsPerMinute)
            this.$emit('update:modelValue', beatsPerMinute)
        },
    },
})
</script>

<style scoped></style>
