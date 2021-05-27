<template>
    <!-- The Navbar -->
    <nav class="navbar" role="navigation" aria-label="main navigation">
        <div class="navbar-brand">
            <a class="navbar-item" href="https://metronome.replayer.ch">
                metrononme
            </a>

            <div class="navbar-end">
                <div class="navbar-item">
                    <About />
                </div>
            </div>
        </div>
    </nav>
    <!-- The Content -->
    <section class="section">
        <div class="container is-unselectable is-shadowless">
            <Metronome :bpm-min="35" :bpm-max="200" />
        </div>
    </section>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Metronome from './components/Metronome.vue'
import About from './components/About.vue'

export default defineComponent({
    name: 'App',
    components: {
        Metronome,
        About,
    },
})
</script>

<style>
#app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
</style>
