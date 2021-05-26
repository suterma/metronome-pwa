<template>
    <h1>Metronome</h1>

    <div class="field is-horizontal">
        <div class="field-label is-small">
            <label for="volume" class="label">Volume</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Fader
                        class="input is-link is-large is-info"
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
        <div class="field-label is-small">
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
                    <!-- <input
                        class="input is-info is-large is-readonly is-static"
                        type="number"
                        id="bpm"
                        v-model="beatsPerMinute"
                        step="1"
                    /> -->
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
        <div class="field-label is-small">
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
        <div class="field-label is-small">
            <label for="bpmFader" class="label is-sr-only">BPM</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Fader
                        class="input is-info is-large"
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

    <!-- <div class="field has-addons">
        <label for="bpm" class="label">BPM</label>
        <div class="control">
            <input
                class="input is-large"
                type="number"
                id="bpm"
                v-model="beatsPerMinute"
                step="1"
            />
        </div>
        <p class="help is-success">This username is available</p>
    </div> -->

    <!-- <div class="field is-horizontal has-addons">
        <div class="field-label is-small">
            <label for="bpm" class="label">BPM</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <button
                        class="button is-info is-large is-outlined"
                        @mousedown="changeBpm(-1)"
                        @keydown="changeBpm(-1)"
                    >
                        -
                    </button>
                </div>
                <div class="control is-expanded">
                    <input
                        class="input is-large"
                        type="number"
                        id="bpm"
                        v-model="beatsPerMinute"
                        step="1"
                    />
                </div>
                <div class="control">
                    <button
                        class="button is-info is-large is-outlined"
                        @mousedown="changeBpm(1)"
                        @keydown="changeBpm(1)"
                    >
                        +
                    </button>
                </div>
            </div>
        </div>
    </div> -->

    <!-- <div class="columns">
        <div class="column is-one-fifth">
            <label for="volume" class="is-hidden-mobile">Volume</label>
        </div>

        <div class="column">
            <input
                class="is-info is-large is-outlined block"
                type="range"
                id="volume"
                min="0"
                max="2"
                v-model="volume"
                step="0.01"
            />
        </div>
        <div class="column is-one-fifth">
            <span class="is-hidden-mobile">{{ volume }}</span>
        </div>
    </div>

    <div class="columns">
        <div class="column is-one-fifth">
            <label for="bpm" class="is-hidden-mobile">BPM</label>
        </div>
        <div class="column">
            <span class="block">
                <button
                    class="button is-info is-large is-outlined"
                    @mousedown="changeBpm(-1)"
                    @keydown="changeBpm(-1)"
                >
                    -
                </button>
                <input
                    class="is-info is-large is-outlined"
                    type="number"
                    id="bpm"
                    name="bpm"
                    :min="bpmMin"
                    :max="bpmMax"
                    step="1"
                    v-model="beatsPerMinute"
                    @change="updateBpm"
                />
                <button
                    class="button is-info is-large is-outlined"
                    @mousedown="changeBpm(1)"
                    @keydown="changeBpm(1)"
                >
                    +
                </button>
            </span>
        </div>
        <div class="column is-one-fifth">
            <span>{{ beatsPerMinute.toLocaleString() }}</span>
        </div>
    </div> -->
    <div class="columns">
        <div class="column is-full">
            {{ sequnceTapCount }}{{ click }} BPM:{{ beatsPerMinute }} Running:
            {{ isRunning }} Volume: {{ volume }}
            <!-- The audio element is not shown. All relevant controls are provided as separate elements -->
            <audio src="../../audio/drumsticks.wav" id="soundelement"></audio>
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
                class="button is-primary is-large is-outlined block"
                @mousedown="tap"
                @keydown="tap"
            >
                Tap/Stop
            </button>
        </div>
    </div>

    <h1>Credits</h1>
    <p>
        <a href="https://freesound.org/people/PrimeJunt/sounds/135627/#"
            >Drum stick sample</a
        >
        (c) by PrimeJunt, minified. Licensed under
        <a href="https://creativecommons.org/licenses/by/3.0/"
            >Creative Commons Attribution 3.0 Unported (CC BY 3.0)</a
        >
    </p>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Fader from '@/components/Fader.vue'

//TODO this does not feel right, how to make this into the component?
let audioContext: AudioContext
let gainNode: GainNode
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
        /** Plays the click sound */
        playClick(): void {
            console.debug('Metronome::playClick')
            this.click = true

            //TODO how to properly address with distict ID?
            const mediaElement = document.getElementById(
                'soundelement',
            ) as HTMLMediaElement
            //Create the audio context (must be called after the first user interaction to avoid exception)
            if (!audioContext) {
                console.log('creating new AudioContext')
                audioContext = new AudioContext()

                //Prepare the sound source
                const track =
                    audioContext.createMediaElementSource(mediaElement)

                //Allow gain control
                gainNode = audioContext.createGain()

                //Wire the chain up
                track.connect(gainNode).connect(audioContext.destination)
            }
            mediaElement.play()

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
/** For this app, use center-aligned controls */
.column {
}
.control-group {
    /* width: 100%; */
}
/** A button, sized for fingered input */
input.is-large {
    /* min-width: 2.5cm;
    min-height: 2.5cm; */
}

/** Metronome buttons have a distinct outlined style */
.button.is-outlined {
    /* border-width: 3mm;
    background-color: transparent; */
}

/** Only very slightly indicate hover */
.button.is-outlined:hover {
    /* background-color: #803535dd; */
}

/** Fill the available space */
.block {
    display: block;
    width: 100%;
    text-align: center;
}

input#bpm {
    /* max-width: 6em; */
}

/** Control, that is only used to display data, do not use the default width */
.is-display.is-readonly.is-static {
    max-width: unset;
    width: auto;
}

/* //TODO additionally make the tap/stop button twice as high */
</style>
