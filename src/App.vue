<template>
  <div
    id="app"
    style="display:flex;justify-content:center;align-items:center;height:100vh;background:#fafafa;"
  >
    <CalendarView
      v-model="selected"
      lang="ru"
      @select="onDateSelect"
    />
  </div>
</template>

<script>
import CalendarView from './components/CalendarView.vue'

export default {
  components: { CalendarView },
  data() {
    return {
      selected: null
    }
  },
  methods: {
    onDateSelect(date) {
      console.log(' Выбрана дата:', date.toISOString().slice(0, 10))
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
