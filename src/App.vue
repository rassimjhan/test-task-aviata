<template>
  <AirTickets :air_tickets="air_tickets"/>
</template>

<script>
import AirTickets from './components/AirTickets.vue'
import results from './results.json'

export default {
  name: 'App',
  components: {
    AirTickets
  },
  data() {
    return {
      air_tickets: results
    }
  }
}
</script>

<style>

</style>
