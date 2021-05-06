<template>
  <div class='text-h4'></div>
</template>

<script>
import tt from '@tomtom-international/web-sdk-services'

export default {
  name: 'TomTomSearch',
  props: ['apikey', 'location'],
  mounted () {
    const _this = this
    tt.services.fuzzySearch({
      key: this.apikey,
      query: this.location
    }).then(function (response) {
      console.log(response)
      _this.$emit('locationFound', response.results[0].position)
      _this.address = response.results[0].address.freeformAddress
    })
  },
  data () {
    return {
      address: 'Hello, I am searching...'
    }
  }
}
</script>
