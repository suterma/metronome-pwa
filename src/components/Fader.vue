<template>
    <span>
        <!-- min{{ min }} max{{ max }} -->
        <input
            :class="classNames"
            type="range"
            :id="id"
            :min="min"
            :max="max"
            :value="rangeValue"
            :step="step"
            @input="changeValue"
            @change="changeValue"
        />
        <span>
            <!-- slider {{ sliderValue }} model {{ modelValue }} range -->
            <!-- {{ rangeValue }} -->
        </span>
    </span>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

/** This is a slider element (range input), working with numerical values.
 *  It is styled as an audio fader control
 * @devdoc The styling has been adapted from https://css-tricks.com/styling-cross-browser-compatible-range-inputs-css/
 */
export default defineComponent({
    props: {
        classNames: String,
        id: String,
        min: {
            type: Number,
            required: false,
            default: 0,
        },
        max: {
            type: Number,
            required: false,
            default: 100,
        },
        step: {
            type: Number,
            required: false,
            default: 1,
        },
        /** The fader's value */
        modelValue: {
            type: Number,
            required: true,
            default: 50,
        },
    },
    data() {
        return {
            /** The model value as a reactive property */
            sliderValue: this.modelValue,
        }
    },
    emits: ['update:modelValue'],
    methods: {
        changeValue(event: Event) {
            if (event && event.target) {
                let value = (event.target as HTMLInputElement).value
                console.debug('Fader::changeValue: ', value)
                this.$emit('update:modelValue', value)
            }
        },
    },
    computed: {
        /** Return a string representation of the model value */
        rangeValue(): string {
            if (this.modelValue) {
                return this.modelValue.toFixed(2)
            }
            return ''
        },
    },
})
</script>

<style scoped lang="css"></style>
