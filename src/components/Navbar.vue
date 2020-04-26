<template>
    <nav>
    <v-app-bar flat app>
        <v-toolbar-title>
            <span class="font-weight-light">Goorman</span>
            <span>Weather</span>
        </v-toolbar-title>
        <v-spacer/>
        <v-btn depressed router :to="'/'">Home</v-btn>
        <v-btn depressed router :to="'/about'">About</v-btn>
    </v-app-bar>
    </nav>
</template>

<script>
export default {
    
}
</script>
