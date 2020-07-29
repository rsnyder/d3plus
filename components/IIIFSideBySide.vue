<template>
  <v-container id="container" :style="containerStyle">
    <div id="map"></div>
  </v-container>
</template>

<script>

const dependencies = [
    'https://unpkg.com/leaflet@1.3.1/dist/leaflet.css',
    'https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js',
    'https://unpkg.com/leaflet@1.3.1/dist/leaflet.js',
    'https://cdn.rawgit.com/mejackreed/Leaflet-IIIF/v2.0.1/leaflet-iiif.js',
    'https://cdn.rawgit.com/digidem/leaflet-side-by-side/6ea4567fe8d1b3e6ea3b0e9ca05d311f5be125a3/leaflet-side-by-side.js'
]

module.exports = {
    name: 'IIIFSideBySide',
    props: { items: Array, width: Number, height: Number },
    computed: {
        containerStyle() { return { width: `${this.width}px`, height: `${this.height}px`, overflowY: 'auto !important' } },
    },
    mounted() {
        console.log(this.$options.name, this.items)
        if (typeof L === 'object') {
            this.init()
        } else {
            this.loadDependencies(dependencies, 0, this.init)
        }
    },
    methods: {
        init() {
            const map = L.map('map', { center: [0, 0], crs: L.CRS.Simple, zoom: 0 })

            var layer1 = L.tileLayer.iiif(
                'https://iiif.visual-essays.app/images/88b53551f91eaeea450ff6c196b4fd5f9fd6955b356d616543af591f/info.json', 
                { fitBounds: false }).addTo(map)
            var layer2 = L.tileLayer.iiif(
                'https://iiif.visual-essays.app/images/a58d7d72db0b59290c54d57e414d88d892efee1b88cc639ee0f6dc3f/info.json',
                { }).addTo(map)

            L.control.sideBySide(layer1, layer2).addTo(map)
        },
        load(url, callback) {
            let e
            if (url.split('.').pop() === 'js') {
                e = document.createElement('script')
                e.src = url
                e.type='text/javascript'
            } else {
                e = document.createElement('link')
                e.href = url
                e.rel='stylesheet'
            }
            e.addEventListener('load', callback)
            document.getElementsByTagName('head')[0].appendChild(e)
        },
        loadDependencies(dependencies, i, callback) {
            this.load(dependencies[i], () => {
                if (i < dependencies.length-1) {
                    this.loadDependencies(dependencies, i+1, callback) 
                } else {
                    callback()
                }
            })
        }
    }
}

</script>

<style>
    #map{
        height: 100%;
        margin: 0;
    }
</style>