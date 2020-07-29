<template>
    <v-container :style="containerStyle">
        <div id="datavis"></div>
    </v-container>
</template>

<script>

// Uses https://d3plus.org/

const dependencies = [
    'https://fonts.googleapis.com/css?family=Roboto',
    'https://d3plus.org/css/styles.css?v=3',
    'https://d3plus.org/js/d3.min.js',
    'https://d3plus.org/js/d3plus.min.js',
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
            fetch(this.items[0].url).then(resp => resp.json())
            .then(data => {
                // convert regular network json file into form used by Rings viz
                let links = data.links.map(link => { return { source: data.nodes[link.source].id, target: data.nodes[link.target].id } })
                new d3plus.Rings()
                    .select('#datavis')
                    .links(links)
                    .label(d => d.id)
                    .center("Anna Maria Luisa de' Medici")
                    .render()
            })
        }
    }
  }
</script>