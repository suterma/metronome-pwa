<template>
    <h1>Metronome</h1>

    <div>
        <label for="bpm">BPM</label>
        <button
            class="button-finger"
            @mousedown="changeBpm(-1)"
            @keydown="changeBpm(-1)"
        >
            -
        </button>
        <input
            type="range"
            id="bpm"
            name="bpm"
            :min="bpmMin"
            :max="bpmMax"
            step="1"
            v-model="beatsPerMinute"
            @change="updateBpm"
        />
        <span>{{ beatsPerMinute }}</span>
        <button
            class="button-finger"
            @mousedown="changeBpm(1)"
            @keydown="changeBpm(1)"
        >
            +
        </button>
    </div>
    <div>
        <button class="button-finger" @mousedown="run" @keydown="run">
            Run
        </button>
    </div>
    <div>
        {{ sequnceTapCount }}{{ click }} BPM:{{ beatsPerMinute }} Running:
        {{ isRunning }}
        <!-- The audio element is not shown. All relevant controls are provided as separate elements -->
        <audio src="../../audio/drumsticks.wav" id="soundelement"></audio>
    </div>
    <div>
        <button class="button-finger" @mousedown="tap" @keydown="tap">
            Tap/Stop
        </button>
    </div>
    <div>
        <input
            type="range"
            id="volume"
            min="0"
            max="2"
            v-model="volume"
            step="0.01"
        />Volume: {{ volume }}
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
import { defineComponent } from 'vue';

//TODO this does not feel right, how to make this into the component?
let audioContext: AudioContext;
let gainNode: GainNode;
let metronomeIntervalId: any;

/** This is metronome component that allows setting or tapping in a Beats-Per-Minute speed
 * @remarks It allows to tap along the rythm for some beats to adjust the speed properly to the sound
 * @devdoc To adjust, a tap sequence is started with a first tap (after a while), then counting taps as long as the user keeps tapping in a reasonable speed
 */
export default defineComponent({
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
        };
    },
    watch: {
        /** Watches for changes on the volume and immediately applies them
         * @remarks using the change event on the input control does only trigger the function call upon mouse up
         */
        volume(newVal, oldVal) {
            //console.debug('observed new volume', newVal);
            gainNode.gain.value = newVal;
        },
    },
    methods: {
        /** Follows the updated BPM rate (by slider, not tap) with re-running the interval
         * @devdoc This intentionally handles the slider change, and does not watch the BPM value
         * to only handle the slider change
         */
        updateBpm(event: Event): void {
            console.debug('Metronome::updateBpm', this.beatsPerMinute);
            this.run();
        },

        /** Increases the BPM rate by one (by button, not tap) with re-running the interval
         */
        changeBpm(beats: number): void {
            console.debug('Metronome::changeBpm', beats);

            //Add beats, cast to Number to make absolutely sure we are not string concatenating here
            if (beats > 0) {
                this.beatsPerMinute =
                    Number(Math.floor(this.beatsPerMinute)) + Number(beats);
            }
            if (beats < 0) {
                this.beatsPerMinute =
                    Number(Math.ceil(this.beatsPerMinute)) + Number(beats);
            }

            if (this.isRunning) {
                //Start again with updated BPM
                this.run();
            }
        },

        /** Handles the tap buttons by starting and stoping the recognition of tap-in sequences and handling the click intervals */
        tap(event: Event): void {
            let thisTap = performance.now();
            this.playClick();
            console.log('tap');

            //Just in case, anything was running, cancel it
            clearTimeout(metronomeIntervalId);

            //calculate the last tap span
            let lastSpan = thisTap - this.lastTapTimeStamp;
            console.debug('lastSpan:', lastSpan);

            //if the last span is reasonable for a tap-in squence, start the metronome with an updated interval
            if (lastSpan < 2500) {
                //start a new tap-in sequence, if there is none
                if (this.sequnceTapCount === 0) {
                    console.debug('NEW Sequence!');
                    this.firstTapOfSequenceTimeStamp = this.lastTapTimeStamp;
                }
                this.sequnceTapCount++;

                //Process the tap-in sequence
                // console.log('Tap-In detected', {
                //     firstTapOfSequenceTimeStamp:
                //         this.firstTapOfSequenceTimeStamp,
                //     thisTap: thisTap,
                //     sequnceTapCount: this.sequnceTapCount,
                // });
                var averageSpan =
                    (thisTap - this.firstTapOfSequenceTimeStamp) /
                    this.sequnceTapCount;

                this.isRunning = true;
                metronomeIntervalId = setInterval(this.playClick, averageSpan);
                this.beatsPerMinute = 60 / (averageSpan / 1000);
            } else {
                //do not continue with the sequence for now
                this.isRunning = false;
                console.debug('sequence timeout');
                this.firstTapOfSequenceTimeStamp = 0;
                this.sequnceTapCount = 0;
            }

            this.lastTapTimeStamp = thisTap;
        },
        /** Handles the run button by (re-)starting the metronome with the current BPM */
        run(): void {
            //Just in case, anything was running, cancel it
            clearTimeout(metronomeIntervalId);
            this.isRunning = true;
            var interval = (60 / this.beatsPerMinute) * 1000;
            metronomeIntervalId = setInterval(this.playClick, interval);

            this.playClick();
            console.log('run');
        },
        /** Plays the click sound */
        playClick(): void {
            console.debug('Metronome::playClick');
            this.click = true;

            //TODO how to properly address with distict ID?
            const mediaElement = document.getElementById(
                'soundelement'
            ) as HTMLMediaElement;
            //Create the audio context (must be called after the first user interaction to avoid exception)
            if (!audioContext) {
                console.log('creating new AudioContext');
                audioContext = new AudioContext();

                //Prepare the sound source
                const track =
                    audioContext.createMediaElementSource(mediaElement);

                //Allow gain control
                gainNode = audioContext.createGain();

                //Wire the chain up
                track.connect(gainNode).connect(audioContext.destination);
            }
            mediaElement.play();

            //TODO later remove, and use a separate visual click cue

            // mediaElement.play().then(() => {
            //     console.debug('playing done');
            //     this.click = false;
            // });
        },
    },
    created() {
        console.log('Metronome::created');
    },
    unmounted() {
        console.log('Metronome::unmounted');
        if (audioContext && audioContext.state !== 'closed') {
            audioContext.close().then(function () {
                console.log('AudioContext closed');
            });
        }
    },
});
</script>

<style scoped>
/** A button, sized for fingered input */
.button-finger {
    min-width: 2.5cm;
    min-height: 2.5cm;
}
</style>
