<template>
    <div class="hello">
        <h1>{{ msg }}</h1>

        <div>
            <label for="bpm">BPM</label>
            <input
                type="range"
                id="bpm"
                name="bpm"
                min="40"
                max="160"
                step="1"
                v-model="beatsPerMinute"
            />
            <span>{{ beatsPerMinute }}</span>
        </div>
        <div>
            <label for="volume">Volume</label>
            <input
                type="range"
                id="volume"
                name="volume"
                min="0"
                max="100"
                step="5"
                v-model="volume"
            />
            <span>{{ volume }}</span>
        </div>
        <div>
            <label for="run">Run</label>
            <input
                type="checkbox"
                id="run"
                name="run"
                checked="true"
                v-model="run"
            />
            <span>{{ run }}</span>
        </div>
        <div>
            <label for="tap">Tap</label>
            <input
                type="button"
                id="tap"
                name="tap"
                @click="tapped"
                value="Tap"
            />
            <span>{{ tap }}</span>
        </div>
        <p>
            For a guide and recipes on how to configure / customize this
            project,<br />
            check out the
            <a href="https://cli.vuejs.org" target="_blank" rel="noopener"
                >vue-cli documentation</a
            >.
        </p>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
    name: 'HelloWorld',
    props: {
        msg: String,
    },
    data() {
        return { beatsPerMinute: 80, volume: 50, run: true };
    },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
    margin: 40px 0 0;
}
ul {
    list-style-type: none;
    padding: 0;
}
li {
    display: inline-block;
    margin: 0 10px;
}
a {
    color: #42b983;
}
</style>
