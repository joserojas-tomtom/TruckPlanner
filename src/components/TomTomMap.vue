<template>
  <div id='map' class='window-height'></div>
</template>

<script>
import tt from '@tomtom-international/web-sdk-maps'
import ttapi from '@tomtom-international/web-sdk-services'

export default {
  name: 'TomTomMap',
  props: ['city', 'apikey'],
  components: { },
  methods: {
    getFormattedSummary (summary) {
      // const meters = summary.lengthInMeters
      let restingTime = 0
      this.waypoints.forEach((value) => {
        restingTime += value.rest
      })

      const time = summary.travelTimeInSeconds + (restingTime * 3600)
      let toReturn = ''
      if (time < 60) toReturn = '⏰ ' + time + ' segundos'
      else if (time < 3600) {
        toReturn = '⏰ ' + (time / 60).toFixed(1) + ' minutos'
      } else {
        toReturn = '⏰ ' + (time / 3600).toFixed(1) + ' hrs'
      }
      const distance = summary.lengthInMeters
      if (distance < 1000) toReturn += '\n⌀ ' + distance + ' mts'
      else {
        toReturn += '\n⌀ ' + (distance / 1000).toFixed(1) + ' kmts'
      }

      if (restingTime > 0) {
        toReturn += '\nIncluding ' + restingTime + 'hr(s) of resting time'
      }
      return toReturn
    },
    resetRoute () {
      this.clearRoute()
      this.waypoints.forEach((value) => {
        value.marker.remove()
      })
      this.waypoints.length = 0
    },
    drawRoute (json) {
      this.routeGroup = {}
      this.routeGroup.outlinelayer = {
        id: 'routeOutline',
        type: 'line',
        source: {
          type: 'geojson',
          data: json
        },
        paint: {
          'line-color': 'black',
          'line-width': 12
        }
      }
      this.routeGroup.linelayer = {
        id: 'routeline',
        type: 'line',
        source: {
          type: 'geojson',
          data: json
        },
        paint: {
          'line-color': 'yellow',
          'line-width': 8
        }
      }

      this.ttmap.addLayer(this.routeGroup.outlinelayer)
      this.ttmap.addLayer(this.routeGroup.linelayer)
    },
    clearRoute () {
      if (this.routeGroup) {
        this.ttmap.removeLayer('routeline')
        this.ttmap.removeSource('routeline')
        this.ttmap.removeLayer('routeOutline')
        this.ttmap.removeSource('routeOutline')
      }
      this.routeGroup = null
    },
    createRoute () {
      this.clearRoute()
      const locations = []
      locations.push(this.originMarker.getLngLat())
      this.waypoints.forEach((value) => {
        locations.push(value.location)
      })
      locations.push(this.destinationMarker.getLngLat())
      const routeOptions = {
        key: this.apikey,
        locations: locations,
        travelMode: 'truck'
      }
      const _this = this
      ttapi.services.calculateRoute(routeOptions).then(function (routeData) {
        console.log(routeData.toGeoJson())
        _this.drawRoute(routeData.toGeoJson())
        _this.$root.$emit('routeInfoUpdate', _this.getFormattedSummary(routeData.routes[0].summary))
      })
    },
    addOriginMarker (location) {
      if (this.originMarker) {
        this.originMarker.remove()
      }
      this.originMarker = this.createMarker(location, 'blue', 'Your starting point')
      this.originMarker.addTo(this.ttmap)
    },
    addDestinationMarker (location) {
      if (this.destinationMarker) {
        this.destinationMarker.remove()
      }
      this.destinationMarker = this.createMarker(location, 'green', 'Your destination')
      this.destinationMarker.addTo(this.ttmap)
    },
    handleSearchResult (location) {
      this.ttmap.easeTo({
        center: location,
        zoom: 16
      })
    },
    createMarker (position, color, popupText) {
      const markerContentElement = document.createElement('div')
      markerContentElement.className = 'waypoint'
      markerContentElement.style.backgroundColor = color

      const popup = new tt.Popup({ offset: 30 }).setText(popupText)
      // add marker to map
      return new tt.Marker({ element: markerContentElement, anchor: 'bottom' })
        .setLngLat(position)
        .setPopup(popup)
    },
    addWaypoint (lngLat) {
      const _this = this
      this.$q.dialog({
        title: 'Adding a rest in your journey',
        message: 'How long do you plan to rest here?:',
        options: {
          type: 'radio',
          model: '1',
          // inline: true
          items: [
            { label: '1 hr approx - coffee?', value: '1' },
            { label: '2 hrs', value: '2' },
            { label: '4 hrs - probably I eat something', value: '4' },
            { label: '6 hrs - maybe a nap', value: '6' }
          ]
        }
      }).onOk(data => {
        console.log('>>>> OK, received', data)
        const newWaypoint = _this.createMarker(lngLat, 'orange', 'Rest ' + data + 'hrs')
        newWaypoint.addTo(_this.ttmap)
        _this.waypoints.push({
          location: lngLat,
          rest: parseFloat(data),
          marker: newWaypoint
        })
      })
    }
  },
  mounted () {
    this.$root.$on('originMarker', this.addOriginMarker)
    this.$root.$on('destinationMarker', this.addDestinationMarker)
    this.$root.$on('createRoute', this.createRoute)
    this.$root.$on('resetRoute', this.resetRoute)
    this.ttmap = tt.map({
      container: 'map',
      key: this.apikey
    })
    this.ttmap.on('click', (event) => {
      this.addWaypoint(event.lngLat)
    })
  },
  beforeDestroy () {
    this.$root.$off('resetRoute')
    this.$root.$off('originMarker')
    this.$root.$off('destinationMarker')
  },
  data () {
    return {
      ttmap: undefined,
      originMarker: undefined,
      destinationMarker: undefined,
      waypoints: [],
      routeGroup: undefined
    }
  }
}
</script>
<style>
@import 'https://api.tomtom.com/maps-sdk-for-web/cdn/6.x/6.13.0/maps/maps.css';

.waypoint {
    background: #c30b82;
    border-radius: 50% 50% 50% 50%;
    height: 20px;
    position: absolute;
    width: 20px;
}
</style>
