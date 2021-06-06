<template>
    <!-- The plus/minus buttons (only shown on smaller devices) -->
    <div class="field is-horizontal is-hidden-desktop">
        <div class="field-label">
            <label for="bpm" class="label">BPM</label>
        </div>
        <div class="field-body">
            <div class="field is-grouped">
                <p class="control is-expanded">
                    <button
                        class="button is-danger is-large block"
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
                            is-display is-danger is-large is-readonly is-static
                        "
                        >{{ beatsPerMinute }}</span
                    >
                </p>
                <p class="control is-expanded">
                    <button
                        class="button is-danger is-large block"
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
                        class="input is-danger is-large"
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
                        classNames="input slider is-danger is-large"
                        type="range"
                        id="bpmFader"
                        :min="35"
                        :max="180"
                        :step="1"
                        v-model.number="beatsPerMinute"
                        @change="updateBpm"
                    />
                </div>
            </div>
        </div>
    </div>

    <!-- <div class="columns">
        <div class="column is-full">
            DEBUG: {{ sequnceTapCount }}{{ click }} BPM:{{
                beatsPerMinute
            }}
            Running: {{ isRunning }} Volume: {{ volume }}
        </div>
    </div> -->

    <!-- The volume slider -->
    <div class="field is-horizontal">
        <div class="field-label">
            <label for="volume" class="label">Volume</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Fader
                        classNames="input slider is-large "
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

    <!-- The running indicator -->
    <progress
        v-show="!isRunning"
        class="progress is-success"
        max="100"
        value="0"
    >
        .
    </progress>
    <progress
        v-show="isRunning"
        class="progress is-success"
        max="100"
        value="100"
    >
        Running
    </progress>

    <!-- Run and tap controls -->
    <div class="columns">
        <div class="column is-half">
            <button
                class="button is-danger is-large block"
                @mousedown="run"
                @keydown="run"
            >
                Run
            </button>
        </div>
        <div class="column is-half">
            <button
                class="button is-primary is-large is-tall block"
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
import { setTimeout } from 'timers'

/** A Wake Lock API polyfill, to avoid screen locks while running the metronome */
import NoSleep from 'nosleep.js'

//TODO this does not feel right, how to make this into the component?
let audioContext: AudioContext
let gainNode: GainNode
/** The loaded sample data, ready to be used with an AudioBufferSourceNode */
let sampleBuffer: AudioBuffer
/** The playable buffer source node, that emits the click sound
 * @remarks Defined here, because it is to be used only once at a time, and stopped/recreated as needed
 */
let sampleBufferSourceNode: AudioBufferSourceNode
let metronomeIntervalId: any

let noSleep = new NoSleep()

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
        /** Watches for changes of the running state and triggers the Wake Lock feature
         * @remarks The screen is prevented from sleep while the metronome is running to allow immediate control to the user.
         * When not running sleep is not prevented, to enable power saves for assumed.
         * @devdoc This currently uses a polyfill for the Wake Lock API (see https://github.com/richtr/NoSleep.js)
         */
        isRunning(newVal, oldVal) {
            if (newVal === true /*running*/) {
                noSleep.enable()
            }
            if (newVal === false) {
                noSleep.disable()
            }
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

        /** Updates the BPM rate (period) to the given value
         * @remarks If the metronome was not running, it's started now.
         * @remarks The Audio Context is expected to be ready
         * @param period The period of the beat, in seconds
         */
        updatePeriod(period: number): void {
            console.debug('Metronome::updatePeriod', period)

            if (sampleBufferSourceNode) {
                sampleBufferSourceNode.disconnect()
                //throw away
            }

            sampleBufferSourceNode = new AudioBufferSourceNode(audioContext)
            sampleBufferSourceNode.buffer = sampleBuffer
            sampleBufferSourceNode.loop = true
            sampleBufferSourceNode.loopStart = 0 //Default, start from the beginning
            sampleBufferSourceNode.loopEnd = period
            sampleBufferSourceNode.connect(gainNode)
            sampleBufferSourceNode.start()

            this.isRunning = true
            console.debug('Metronome::playClick::loop')
        },

        /** Handles the tap buttons by starting and stoping the recognition of tap-in sequences and handling the click intervals */
        tap(event: Event): void {
            let thisTap = performance.now()

            //stop looping an already running node
            //TODO fix: when the user clicks early in a sequence (e.g the loop has just not yet restarted), this will cause that there is no audible click
            if (sampleBufferSourceNode) {
                sampleBufferSourceNode.loop = false
            }

            console.log('tap')

            //calculate the last period (tap span)
            let lastPeriod = (thisTap - this.lastTapTimeStamp) / 1000

            //if the last period is reasonable for a tap-in squence, start the metronome with an updated interval
            if (lastPeriod < this.getMaxTapPeriod) {
                //start a new tap-in sequence, if there is none
                var isNewSequence = this.sequnceTapCount === 0
                if (isNewSequence) {
                    console.debug('NEW Sequence!')
                    this.firstTapOfSequenceTimeStamp = this.lastTapTimeStamp
                }
                this.sequnceTapCount++

                //Get the average tap period in seconds, and use it as new BPM setting
                var averagePeriod =
                    (thisTap - this.firstTapOfSequenceTimeStamp) /
                    this.sequnceTapCount /
                    1000

                this.beatsPerMinute = 60 / averagePeriod

                if (isNewSequence) {
                    //immediately start with the inferred period
                    this.updatePeriod(averagePeriod)
                } else {
                    //let the current click play out, then start with the loop again
                    setTimeout(() => {
                        this.updatePeriod(averagePeriod)
                    }, averagePeriod * 1000)
                }
            } else {
                //do not continue with the sequence for now
                this.isRunning = false
                console.debug('sequence timeout')
                this.firstTapOfSequenceTimeStamp = 0
                this.sequnceTapCount = 0

                //TODO detect, whether a click is due soon, then not play again (prevent double cklic).
                //BEWARE: First check, whether this line is actually the problematic one
                this.playClickOnce()
            }

            this.lastTapTimeStamp = thisTap
        },
        /** Handles the run button by (re-)starting the metronome with the current BPM */
        run(): void {
            var period = 60 / this.beatsPerMinute
            this.updatePeriod(period) //also sets the isRunning state to true
            console.log('run')
        },
        /** Inizializes the Audio Context
         * @devdoc Initializing the audio context and playing a sample is only allowed after the first user interaction
         */
        initializeAudio(): void {
            if (!audioContext) {
                console.log('creating new AudioContext')
                audioContext = new AudioContext()

                //Wire the processing chain up
                gainNode = audioContext.createGain()
                gainNode.connect(audioContext.destination)

                //Prepare the sound source
                console.debug('Special buffer sound')
                let byteArray = decode(samples[0].buffer)

                console.debug('Decoding audio data')
                audioContext.decodeAudioData(
                    byteArray,
                    function (buffer) {
                        //This decoded buffer will be later used to actually play the sample, using the sampleBufferSourceNode, which is created only when actual playing is needed
                        sampleBuffer = buffer
                    },
                    function (err) {
                        console.log('err(decodeAudioData): ' + err)
                    },
                )
            }
        },
        /** Plays the click sound once
         * @devdoc Initializing the audio context and playing a sample is only allowed after the first user interaction
         */
        playClickOnce(): void {
            console.debug('Metronome::playClickOnce')
            this.click = true

            sampleBufferSourceNode = new AudioBufferSourceNode(audioContext)
            sampleBufferSourceNode.buffer = sampleBuffer
            sampleBufferSourceNode.connect(gainNode)
            sampleBufferSourceNode.start()

            //TODO later provide an additional visual click cue
        },
    },
    created() {
        console.log('Metronome::created')
        this.initializeAudio()
    },
    unmounted() {
        console.log('Metronome::unmounted')
        if (audioContext && audioContext.state !== 'closed') {
            audioContext.close().then(function () {
                console.log('AudioContext closed')
            })
        }
    },
    computed: {
        /** Returns the maximum expected tap-in period in seconds, according to the minimum set BPM value */
        getMaxTapPeriod(): number {
            return 60 / this.bpmMin
        },
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
