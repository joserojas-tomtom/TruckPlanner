<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          My truck planner
        </q-toolbar-title>

      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      content-class="bg-grey-1"
    >
      <q-list>
        <div class='q-ma-md text-h5'>Planner</div>
        <q-input class='q-ma-sm' color="orange" standout bottom-slots
          @blur='originKey += 1'
          v-model="origin" label="Origin"  clearable>
          <template v-slot:prepend>
            <q-icon name="place" />
          </template>
          <template v-slot:hint>
            Starting point - enter an address
          </template>
        </q-input>
        <TomTomSearch :key='originKey' :apikey='apikey' :location='origin' @locationFound='originFound'/>

        <q-separator class='q-my-lg' inset/>

        <q-input class='q-ma-sm' color="orange" standout
          @blur='destinationKey += 1'
          bottom-slots v-model="destination" label="Destination"  clearable>
        <template v-slot:prepend>
          <q-icon name="place" />
        </template>

        <template v-slot:hint>
          Final destination - enter an address
        </template>
      </q-input>
      <TomTomSearch :key='destinationKey' :apikey='apikey' :location='destination' @locationFound='destinationFound'/>
      <q-separator class='q-my-lg' inset/>
      <q-select class='q-ma-sm' color="red" bg-color="lightgrey" filled v-model="drivingTime" :options="options" label="Driving time">
        <template v-slot:prepend>
          <q-icon name="watch" />
        </template>
      </q-select>
        <div class='row justify-center'>
          <q-btn class='fit q-ma-sm' color="primary" label="Create Route" @click='createRoute'/>
        </div>
        <div class='row justify-center'>
          <q-btn class='fit q-ma-sm' color="primary" label="Reset" @click='reset'/>
        </div>
        <div class='q-ma-sm text-h5'>
          {{ routeInfo }}
        </div>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import TomTomSearch from 'components/Search.vue'
export default {
  name: 'MainLayout',
  components: { TomTomSearch },
  methods: {
    reset () {
      this.routeInfo = ''
      this.$root.$emit('resetRoute')
    },
    updateRouteInfo (info) {
      this.routeInfo = info
    },
    originFound (location) {
      console.log('origin:', location)
      this.$root.$emit('originMarker', location)
    },
    destinationFound (location) {
      console.log('destination:', location)
      this.$root.$emit('destinationMarker', location)
    },
    createRoute () {
      this.$root.$emit('createRoute')
    }
  },
  mounted () {
    this.$root.$on('routeInfoUpdate', this.updateRouteInfo)
  },
  beforeDestroy () {
    this.$root.$off('routeInfoUpdate')
  },
  data () {
    return {
      apikey: 'Vn26cA8knt2E8sl0WBEWvAgWGRUf59mm',
      routeInfo: '',
      leftDrawerOpen: false,
      origin: 'berlin',
      destination: 'amsterdam',
      drivingTime: '4 hours',
      options: [
        { label: '4 Hours', value: 4 },
        { label: '6 Hours', value: 6 },
        { label: '8 Hours', value: 8 },
        { label: '10 Hours', value: 10 },
        { label: '12 Hours', value: 12 },
        { label: '14 Hours', value: 14 }
      ],
      originKey: 0,
      destinationKey: 100000
    }
  }
}
</script>
