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

<style scoped lang="css">
/* input[type='range'] {
    -webkit-appearance: none;
    margin: 18px 0;
    width: 100%;
}
input[type='range']:focus {
    outline: none;
}
input[type='range']::-webkit-slider-runnable-track {
    width: 100%;
    height: 8.4px;
    cursor: pointer;

    background: #3071a9;
    border-radius: 1.3px;
    border: 0.2px solid #010101;
}
input[type='range']::-webkit-slider-thumb {
    border: 1px solid #000000;
    height: 36px;
    width: 16px;
    border-radius: 3px;
    background: #ffffff;
    cursor: pointer;
    -webkit-appearance: none;
    margin-top: -14px;
}
input[type='range']:focus::-webkit-slider-runnable-track {
    background: #367ebd;
}
input[type='range']::-moz-range-track {
    width: 100%;
    height: 8.4px;
    cursor: pointer;

    background: #3071a9;
    border-radius: 1.3px;
    border: 0.2px solid #010101;
}
input[type='range']::-moz-range-thumb {
    border: 1px solid #000000;
    height: 36px;
    width: 16px;
    border-radius: 3px;
    background: #ffffff;
    cursor: pointer;
}
input[type='range']::-ms-track {
    width: 100%;
    height: 8.4px;
    cursor: pointer;
    background: transparent;
    border-color: transparent;
    border-width: 16px 0;
    color: transparent;
}
input[type='range']::-ms-fill-lower {
    background: #2a6495;
    border: 0.2px solid #010101;
    border-radius: 2.6px;
}
input[type='range']::-ms-fill-upper {
    background: #3071a9;
    border: 0.2px solid #010101;
    border-radius: 2.6px;
}
input[type='range']::-ms-thumb {
    border: 1px solid #000000;
    height: 36px;
    width: 16px;
    border-radius: 3px;
    background: #ffffff;
    cursor: pointer;
}
input[type='range']:focus::-ms-fill-lower {
    background: #3071a9;
}
input[type='range']:focus::-ms-fill-upper {
    background: #367ebd;
} */
</style>
