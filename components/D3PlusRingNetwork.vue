<template>
    <v-container :style="containerStyle">
        <div id="viz"></div>
    </v-container>
</template>

<script>

// Uses https://d3plus.org/

const dependencies = [
    'https://fonts.googleapis.com/css?family=Roboto',
    'https://d3plus.org/css/styles.css?v=3',
    // 'https://d3plus.org/css/syntax.css',
    // 'https://d3plus.org/css/blog.css',
    // 'https://d3plus.org/css/sidecar.css',
    // 'https://d3plus.org/assets/font-awesome-4.7.0/css/font-awesome.min.css',
    'https://d3plus.org/js/d3.min.js',
    'https://d3plus.org/js/d3plus.min.js',
    // 'https://sidecar.gitter.im/dist/sidecar.v1.js',
    'https://d3plus.org/js/d3plus-network.v0.6.full.min.js'
]

module.exports = {
    name: 'D3PlusRingNetwork',
    props: {
      items: Array,
      width: Number,
      height: Number
    },
    computed: {
      containerStyle() { return { width: `${this.width}px`, height: `${this.height}px`, overflowY: 'auto !important', marginLeft: '24px' } },
    },
    mounted() {
        console.log(this.$options.name, this.items)
        if (typeof d3plus === 'object') {
            this.init()
        } else {
            this.loadDependencies(dependencies, 0, this.init)
        }
    },
    methods: {
        init() {
            // instantiate our D3plus viz object
            var viz = d3plus.viz()

            // here we get all the data we need from the server, firt the raw data, then
            // each of the attribute files
            d3.json("artists.json", function(artists){
                d3.json("albums.json", function(albums){
                    d3.json("songs.json", function(songs){
                        d3.json("artist_similarity.json", function(network){
                        
                            var attrs = {"artist_id": artists, "album_id": albums, "song_id": songs}
                    
                            viz
                            .type("rings")
                            .attrs(attrs)
                            .aggs({"hotttnesss":"mean"})
                            .dev(true)
                            .value("hotttnesss")
                            .id("artist_id")
                            .text("names")
                            .tooltip(["names","location"])
                            .year(2010)
                            .year_var("year")
                            .nodes(network.nodes)
                            .links(network.edges)
                            .color("hotttnesss")
                            .highlight("ARJ7KF01187B98D717")

                            d3.select("#viz")
                            .datum(songs)
                            .call(viz)
                            
                    
                        })
                    })
                })
            })

        }
    }
  }
</script>

<style scoped>
  body {
    margin: 0;
    overflow: hidden;
  }
</style>