<template>
    <div class="content">
        <Metronome :bpm-min="35" :bpm-max="200" />
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Metronome from '@/components/Metronome.vue'
export default defineComponent({
    name: 'Home',
    components: {
        Metronome,
    },

    mounted() {
        console.debug('Home::mounted')
    },
    unmounted() {
        console.debug('Home::unmounted')
    },
})
</script>
