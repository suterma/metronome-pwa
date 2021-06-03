<template>
    <!-- The Navbar -->
    <nav class="navbar" role="navigation" aria-label="main navigation">
        <div class="navbar-brand">
            <a class="navbar-item" href="https://tic.replayer.ch">
                <img src="./assets/logo.png" height="28" />
            </a>

            <!-- For this app, no closeable menu is used, buth all option are always displayed -->
            <div id="navbarApp" class="navbar-menu is-active">
                <div class="navbar-end">
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
// import SelectBpm from './components/SelectBpm.vue'

export default defineComponent({
    name: 'App',
    components: {
        Metronome,
        About,
        // SelectBpm,
    },
})
</script>

<style>
#app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

/** Use only little vertical padding, to better use the available space */
#app section.section {
    padding-top: 1em;
}
</style>
