<template>
    <!-- The Navbar -->
    <!-- transparent, to omit shadows -->
    <!-- using a fixed navbar, see https://bulma.io/documentation/components/navbar/#fixed-navbar -->
    <nav
        class="navbar is-transparent is-fixed-top"
        role="navigation"
        aria-label="main navigation"
    >
        <div class="navbar-brand">
            <a class="navbar-item" href="https://tic.replayer.app">
                <!-- HINT: Provide explicit height and widht as an optimization measure -->
                <img src="./assets/logo.png" width="28" height="28" />
            </a>
        </div>

        <!-- For this app, no closeable menu is used, buth all option are always displayed -->
        <div id="navbarApp" class="navbar-menu is-active">
            <!-- navbar-end makes the menu centered -->
            <div class="navbar-start">
                <router-link class="navbar-item" to="/">Metronome</router-link>
                <router-link class="navbar-item" to="/about">About</router-link>
            </div>
            <div class="navbar-end">
                <!-- currently, there is no end menu -->
            </div>
        </div>
    </nav>
    <!-- The Content -->
    <section class="section">
        <div class="container is-unselectable is-shadowless">
            <!-- To keep the audio within the metronome component running, 
            simply keep all components alive over route changes -->
            <router-view v-slot="{ Component }">
                <keep-alive>
                    <component :is="Component" />
                </keep-alive>
            </router-view>
        </div>
    </section>
</template>

<style lang="scss">
#app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
/** Use only little vertical padding, to better use the available space */
#app section.section {
    padding-top: 1em;
}
</style>
