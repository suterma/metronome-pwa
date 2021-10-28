<template>
    <div class="content">
        <h1 class="title">About</h1>
        <p>
            <a href="https://tic.replayer.app">tic (metronome)</a>
            (c) by
            <a href="https://marcelsuter.ch">Marcel Suter</a>. Licensed under
            the
            <a href="https://www.gnu.org/licenses/gpl-3.0.txt"
                >GNU General Public License, Version 3 (GPLv3)</a
            >
        </p>

        <h1 class="title">Credits</h1>
        <ul>
            <li>
                <a href="https://vuejs.org/"
                    >Vue.js, The Progressive JavaScript Framework</a
                >
                © 2014-2021 Evan You. Licensed under the
                <a href="https://opensource.org/licenses/MIT">MIT License</a>
            </li>
            <li>
                <a href="https://bulma.io/">Bulma CSS Framework</a>
                © Jeremy Thomas. Licensed under the
                <a href="https://opensource.org/licenses/MIT">MIT License</a>
            </li>
            <li>
                <a href="https://jenil.github.io/bulmaswatch/nuclear/"
                    >Bulmaswatch Nuclear Theme</a
                >
                © Jenil Gogari. Licensed under the
                <a href="https://opensource.org/licenses/MIT">MIT License</a>
            </li>
            <li>
                <a href="https://freesound.org/people/PrimeJunt/sounds/135627/#"
                    >Drum stick sample</a
                >
                © PrimeJunt, edited. Licensed under
                <a href="https://creativecommons.org/licenses/by/3.0/"
                    >Creative Commons Attribution 3.0 Unported (CC BY 3.0)</a
                >
            </li>
            <li>
                <a href="https://www.npmjs.com/package/base64-arraybuffer"
                    >base64-arraybuffer</a
                >
                © 2012 Niklas von Hertzen. Licensed under the
                <a href="https://opensource.org/licenses/MIT">MIT License</a>
            </li>
            <li>
                <a href="https://www.svgrepo.com/svg/30590/drumstick"
                    >Drumstick SVG Vector</a
                >
                Icon by SVG Repo. Licensed under the
                <a
                    href="https://creativecommons.org/share-your-work/public-domain/cc0/"
                    >CC0 License</a
                >
            </li>
            <li>
                <a href="https://github.com/richtr/NoSleep.js">NoSleep.js</a>
                a Wake Lock polyfill by Rich Tibbett. Licensed under the
                <a href="https://opensource.org/licenses/MIT">MIT License</a>
            </li>
        </ul>
    </div>
</template>
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
    name: 'About',
    components: {},

    mounted() {
        console.debug('About::mounted')
    },
    unmounted() {
        console.debug('About::unmounted')
    },
})
</script>
