<template>
    <div class="field is-horizontal">
        <div class="field-label">
            <label for="volume" class="label">Volume</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Fader
                        classNames="input is-link is-large is-info"
                        type="range"
                        id="volume"
                        :min="0"
                        :max="2"
                        :step="0.01"
                        v-model.number="volume"
                    />
                </div>
            </div>
        </div>
    </div>

    <!-- The plus/minus buttons (only shown on smaller devices) -->
    <div class="field is-horizontal is-hidden-desktop">
        <div class="field-label">
            <label for="bpm" class="label">BPM</label>
        </div>
        <div class="field-body">
            <div class="field is-grouped">
                <p class="control is-expanded">
                    <button
                        class="button is-info is-large is-outlined block"
                        @mousedown="changeBpm(-1)"
                        @keydown="changeBpm(-1)"
                    >
                        -
                    </button>
                </p>

                <p class="control is-expanded">
                    <!-- {{ beatsPerMinute }} -->
                    <span
                        class="
                            input
                            is-display is-info is-large is-readonly is-static
                        "
                        >{{ beatsPerMinute }}</span
                    >
                </p>
                <p class="control is-expanded">
                    <button
                        class="button is-info is-large is-outlined block"
                        @mousedown="changeBpm(1)"
                        @keydown="changeBpm(1)"
                    >
                        +
                    </button>
                </p>
            </div>
        </div>
    </div>

    <!-- The BPM Text input (not shown on smaller devices) -->
    <div class="field is-horizontal is-hidden-touch">
        <div class="field-label">
            <label for="bpmTextbox" class="label">BPM</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <input
                        class="input is-info is-large"
                        type="number"
                        id="bpmTextbox"
                        v-model.number="beatsPerMinute"
                    />
                </div>
            </div>
        </div>
    </div>

    <!-- The BPM Slider (always shown) -->
    <div class="field is-horizontal">
        <div class="field-label">
            <label for="bpmFader" class="label is-sr-only">BPM</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Fader
                        classNames="input is-info is-large"
                        type="range"
                        id="bpmFader"
                        :min="35"
                        :max="180"
                        :step="1"
                        v-model.number="beatsPerMinute"
                    />
                </div>
            </div>
        </div>
    </div>

    <div class="columns">
        <div class="column is-full">
            DEBUG: {{ sequnceTapCount }}{{ click }} BPM:{{
                beatsPerMinute
            }}
            Running: {{ isRunning }} Volume: {{ volume }}
        </div>
    </div>
    <div class="columns">
        <div class="column is-half">
            <button
                class="button is-info is-large is-outlined block"
                @mousedown="run"
                @keydown="run"
            >
                Run
            </button>
        </div>

        <div class="column is-half">
            <button
                class="
                    button
                    is-primary is-large is-double-height is-outlined
                    block
                "
                @mousedown="tap"
                @keydown="tap"
            >
                Tap/Stop
            </button>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Fader from '@/components/Fader.vue'
import samples from './samples.json'
import { decode } from 'base64-arraybuffer'

//TODO this does not feel right, how to make this into the component?
let audioContext: AudioContext
let gainNode: GainNode
let bufferSource: AudioBufferSourceNode
let metronomeIntervalId: any

/** This is metronome component that allows setting or tapping in a Beats-Per-Minute speed
 * @remarks It allows to tap along the rythm for some beats to adjust the speed properly to the sound
 * @devdoc To adjust, a tap sequence is started with a first tap (after a while), then counting taps as long as the user keeps tapping in a reasonable speed
 */
export default defineComponent({
    components: {
        Fader,
    },
    props: {
        /** The minimum BPM value accepted
         * @remarks Slower taps will not get recognised as a sequence
         * @remarks The BPM slider will use this as lower end of the range
         */
        bpmMin: {
            type: Number,
            default: 40,
            required: false,
        },
        /** The maximum BPM value accepted
         * @remarks The BPM slider will use this as upper end of the range
         */
        bpmMax: {
            type: Number,
            default: 180,
            required: false,
        },
    },
    data() {
        return {
            volume: 1,
            sequnceTapCount: 0,
            click: false,
            firstTapOfSequenceTimeStamp: 0,
            lastTapTimeStamp: 0,
            beatsPerMinute: 60,
            isRunning: false,
        }
    },
    watch: {
        /** Watches for changes on the volume and immediately applies them
         * @remarks using the change event on the input control does only trigger the function call upon mouse up
         */
        volume(newVal, oldVal) {
            //console.debug('observed new volume', newVal);
            gainNode.gain.value = newVal
        },
    },
    methods: {
        /** Follows the updated BPM rate (by slider, not tap) with re-running the interval
         * @devdoc This intentionally handles the slider change, and does not watch the BPM value
         * to only handle the slider change
         */
        updateBpm(event: Event): void {
            console.debug('Metronome::updateBpm', this.beatsPerMinute)
            if (this.isRunning) {
                //Start again with updated BPM
                this.run()
            }
        },

        /** Increases the BPM rate by one (by button, not tap) with re-running the interval
         */
        changeBpm(beats: number): void {
            console.debug('Metronome::changeBpm', beats)

            //Add beats, cast to Number to make absolutely sure we are not string concatenating here
            if (beats > 0) {
                this.beatsPerMinute =
                    Number(Math.floor(this.beatsPerMinute)) + Number(beats)
            }
            if (beats < 0) {
                this.beatsPerMinute =
                    Number(Math.ceil(this.beatsPerMinute)) + Number(beats)
            }

            if (this.isRunning) {
                //Start again with updated BPM
                this.run()
            }
        },

        /** Handles the tap buttons by starting and stoping the recognition of tap-in sequences and handling the click intervals */
        tap(event: Event): void {
            let thisTap = performance.now()
            this.playClick()
            console.log('tap')

            //Just in case, anything was running, cancel it
            clearTimeout(metronomeIntervalId)

            //calculate the last tap span
            let lastSpan = thisTap - this.lastTapTimeStamp
            console.debug('lastSpan:', lastSpan)

            //if the last span is reasonable for a tap-in squence, start the metronome with an updated interval
            if (lastSpan < 2500) {
                //start a new tap-in sequence, if there is none
                if (this.sequnceTapCount === 0) {
                    console.debug('NEW Sequence!')
                    this.firstTapOfSequenceTimeStamp = this.lastTapTimeStamp
                }
                this.sequnceTapCount++

                //Process the tap-in sequence
                // console.log('Tap-In detected', {
                //     firstTapOfSequenceTimeStamp:
                //         this.firstTapOfSequenceTimeStamp,
                //     thisTap: thisTap,
                //     sequnceTapCount: this.sequnceTapCount,
                // });
                var averageSpan =
                    (thisTap - this.firstTapOfSequenceTimeStamp) /
                    this.sequnceTapCount

                this.isRunning = true
                metronomeIntervalId = setInterval(this.playClick, averageSpan)
                this.beatsPerMinute = 60 / (averageSpan / 1000)
            } else {
                //do not continue with the sequence for now
                this.isRunning = false
                console.debug('sequence timeout')
                this.firstTapOfSequenceTimeStamp = 0
                this.sequnceTapCount = 0
            }

            this.lastTapTimeStamp = thisTap
        },
        /** Handles the run button by (re-)starting the metronome with the current BPM */
        run(): void {
            //Just in case, anything was running, cancel it
            clearTimeout(metronomeIntervalId)
            this.isRunning = true
            var interval = (60 / this.beatsPerMinute) * 1000
            metronomeIntervalId = setInterval(this.playClick, interval)

            this.playClick()
            console.log('run')
        },
        /** Plays the click sound once
         * @remarks Also initializes the audio context.
         * @devdoc Initializing the audio context and playing a sample is only allowed after the first user interaction
         */
        playClick(): void {
            console.debug('Metronome::playClick')
            this.click = true

            //Create the audio context (must be called after the first user interaction to avoid exception)
            if (!audioContext) {
                console.log('creating new AudioContext')
                audioContext = new AudioContext()

                //Prepare the sound source
                console.debug('Special buffer sound')
                let byteArray = decode(samples[0].buffer)

                console.debug('Decoding audio data')
                audioContext.decodeAudioData(
                    byteArray,
                    function (buffer) {
                        bufferSource = audioContext.createBufferSource() // creates a sound source
                        bufferSource.buffer = buffer
                        bufferSource.connect(gainNode) // connect the source to the destination
                        bufferSource.start()
                    },
                    function (err) {
                        console.log('err(decodeAudioData): ' + err)
                    },
                )

                // var buffer = await audioContext.decodeAudioData(byteArray)
                // var bufferSource = audioContext.createBufferSource()
                // bufferSource.buffer = buffer;

                //Wire the chain up
                gainNode = audioContext.createGain()
                gainNode.connect(audioContext.destination)
            }
            //TODO continue here....
            //TODO try to initialize the sound api as much as possible at startup, at least loading the sound
            //TODO use Lighthouse to assess the performance implications of loading loading from base64
            //TODO this does not work the first time, should start in async success method above
            //TODO use this as the single, looped call upon changing the BPM, start the run
            if (bufferSource) {
                bufferSource.loop = true
                bufferSource.loopStart = 0 //Default, start from the beginning
                bufferSource.loopEnd = 0.5
                bufferSource.start()
            }

            //TODO later provide an additional visual click cue
        },
    },
    created() {
        console.log('Metronome::created')
    },
    unmounted() {
        console.log('Metronome::unmounted')
        if (audioContext && audioContext.state !== 'closed') {
            audioContext.close().then(function () {
                console.log('AudioContext closed')
            })
        }
    },
})
</script>

<style scoped>
/** Fill the available space */
.block {
    display: block;
    width: 100%;
    text-align: center;
}

/** Control, that is only used to display data, do not use the default width */
.is-display.is-readonly.is-static {
    max-width: unset;
    width: auto;
}
</style>
