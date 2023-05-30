<template>
    <div>
        <b-switch v-model="darkMode" :class="{ 'has-text-white': darkMode }">Dark Mode</b-switch>
    </div>
</template>
  
<script>
export default {
    data() {
        return {
            darkMode: false
        }
    },
    mounted() {
        // Retrieve dark mode state from localStorage on component mount
        this.darkMode = localStorage.getItem('darkMode') === 'true'
    },
    watch: {
        darkMode(newVal) {
            // Method to be triggered when the switch is toggled
            this.$emit('dark-mode-toggle', this.darkMode)
        }
    },
}
</script>
  