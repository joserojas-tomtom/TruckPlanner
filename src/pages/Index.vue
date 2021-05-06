<template>
  <q-page class="flex-center">
    <TomTomMap city='Portland, OR' apikey='Vn26cA8knt2E8sl0WBEWvAgWGRUf59mm'/>
  </q-page>
</template>

<script>
import TomTomMap from 'components/TomTomMap.vue'
export default {
  name: 'PageIndex',
  components: { TomTomMap }
}
</script>
