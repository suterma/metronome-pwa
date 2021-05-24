<template>
    <section class="section">
        <div class="container is-unselectable is-shadowless">
            <img alt="Vue logo" src="./assets/logo.png" />
            <Metronome :bpm-min="35" :bpm-max="200" />
        </div>
    </section>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Metronome from './components/Metronome.vue';

export default defineComponent({
    name: 'App',
    components: {
        Metronome,
    },
});
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>
