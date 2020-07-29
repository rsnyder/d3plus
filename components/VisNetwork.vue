<template>
  <v-container :style="containerStyle">
    <div id="vis">
      <div id="mynetwork"></div>
    </div>
  </v-container>
</template>

<script>

const dependencies = [
    'https://unpkg.com/vis-network/styles/vis-network.min.css',
    'https://unpkg.com/vis-data@latest/peer/umd/vis-data.min.js',
    'https://unpkg.com/vis-network@latest/peer/umd/vis-network.min.js'
]

const defaults = {
  popupOptions: { autoClose: false, closeButton: false, closeOnClick: false }
}


   module.exports = {
      name: 'visNetwork',
      props: {
         items: { type: Array, default: () => ([]) },
         selected: String,
         width: Number,
         height: Number,
         hoverItemID: String,
         selectedItemID: String
      },
      data: () => ({
         activeWindow: undefined,
         popups: {},
         active: new Set(),
      }),
      computed: {
         containerStyle() { return { width: `${this.width}px`, height: `${this.height}px`, overflowY: 'auto !important' } },
         activeElements() { return this.$store.getters.activeElements },
         entities() { return this.itemsInActiveElements.filter(item => item.tag === 'entity') },
         itemsInActiveElements() { return this.$store.getters.itemsInActiveElements },
         apiBaseURL() { return window.location.origin },
         componentsBaseURL() {return window.location.hostname === 'localhost' ? '' : 'https://jstor-labs.github.io/visual-essays'},
      
      },
      mounted() {
        console.log(this.$options.name, this.items)
        this.loadDependencies(dependencies, 0, this.init)
      },
      methods: {
        init() {
         var self = this;

         var nodes = null;
         var edges = null;
         var network = null;

         var nodeslist = [];
         var edgeslist = [];

         //get input data here from file
         self.getInput(this.items[0].file)
            .then(function(result){
                self.parseText(result, nodeslist, edgeslist);     
            })
            .then(function(result){
               self.getImages(nodeslist)
            })
            .then(function(result){
                console.log(result)
               //creating nodes and edges from file data
               nodes = new vis.DataSet(nodeslist);
               edges = new vis.DataSet(edgeslist);

               console.log('nodeslist', nodeslist);
               console.log('edgeslist', edgeslist);

               var container = document.getElementById("mynetwork");
               var data = {
                  nodes: nodes,
                  edges: edges
               };
               var options = {
                  interaction: { hover: true },
                  layout: {
                     randomSeed: undefined,
                     improvedLayout:true,
                     clusterThreshold: 150,
                     hierarchical: false,
                  }
               };

               //init network
               var network = new vis.Network(container, data, options);

               network.on( 'click', function(properties) {
                  var ids = properties.nodes;
                  var clickedNodes = nodes.get(ids);
                  console.log('clicked nodes:', clickedNodes);
                  self.setSelectedItemID(clickedNodes[0].qid);
               });
               network.body.emitter.emit('_dataChanged');
               network.redraw();

            })
        },
        renderGraph() {
            console.log('renderGraph')
        },
         setHoverItemID(itemID) {
            this.$emit('hover-id', itemID)
         },
         setSelectedItemID(itemID) {
            console.log('in setSelectedItemID', itemID);
            this.$emit('selected-id', itemID)
         },
         addEventHandlers(elem, itemId) {
            elem.on('click', () => { this.setSelectedItemID(itemId) })
            elem.on('mouseover', () => { this.setHoverItemID(itemId) })
            elem.on('mouseout', () => { this.setHoverItemID() })
         },
         addPopup(id, label, latLng, offset) {
            if (!this.popups[id]) {
            const popup = L.popup({ ...defaults.popupOptions, ...{ offset: L.point(0, offset || 0)}})
            popup.setLatLng(latLng)
            popup.setContent(`<h1 data-eid="${id}">${label}</h1>`)
            popup.options.id = id
            this.popups[id] = popup
            }
         },
         
         getInput() {
            let filepath = this.items[0].file
            //console.log('filepath', filepath);
            return fetch(filepath).then(resp => resp.text())
         },

         parseText(text, nodes, edges){
                  var lines = text.split(/\r?\n/);
                  var edgeBreak = lines.length;

                  for (var i = 2; i < edgeBreak-2; i++){
                     //find edgeBreak
                     if (lines[i].includes('Edges')){
                        edgeBreak = i;
                     }
                     var obj = lines[i].split('\t');
                     
                     //populate nodes array
                     //make sure nodeobj has at least id and title
                     if (!(isNaN(obj[0])) && obj[0] != ""){
                        nodeobj = {
                           id: obj[0],
                           qid: obj[2],
                           title: obj[1],
                           label: obj[1],
                        }
                        nodes.push(nodeobj);
                     }
                  }

                  for (var i = edgeBreak+2; i < lines.length; i++){
                     var obj = lines[i].split('\t');
                     //populate edges array
                     if (!(isNaN(obj[0])) && obj[0] != ""){
                        edgeobj = {
                           from: obj[0],
                           to: obj[1],
                           title: obj[2],
                        }
                        edges.push(edgeobj);
                     }
                  }

         },
         /*
         callEntities(nodeslist){
            for (var i = 0; i < nodeslist.length; i++){
               console.log('nodeslist[i]', nodeslist[i]);
               if (nodeslist[i].qid != ""){
                  var imageobj;
                  this.getEntity(nodeslist[i].qid).then(function(result){
                     imageobj = result['summary info'].originalimage;
                  
                  });
                  nodeslist[i].image = imageobj.source;
               }
            }
         },
         */
         //get entity (image) for a single node
         getImage(node){
            return this.getEntity(node.qid).then(function(result){
               console.log('result from getEntity', result);
               var imageobj = result['summary info'].originalimage;
               node.image = imageobj.source;
               //node.title = imageobj.source;
               node.shape = "circularImage";
               return node
            })
            // return node;
         },

         getImages(nodeslist){
             const promises = nodeslist
                .filter(node => node.qid)
                .map(node => this.getImage(node))

            console.log(promises)
            Promise.all(promises)
            .then(resp => {
                console.log('Promises.all', resp)
                return resp
            })
            /*
            return nodeslist.forEach((node) => {
               if (node.qid != ""){
                     this.getImage(node);
                  }
            });
            */
         },

         toQueryString(args) {
            const parts = []
            Object.keys(args).forEach((key) => {
            parts.push(`${key}=${encodeURIComponent(args[key])}`)
            })
            return parts.join('&')
         },
         getEntity(eid) {
            let url = `https://visual-essays.app/entity/${encodeURIComponent(eid)}`
            const args = {}
            if (this.context) args.context = this.context
            //if (this.entity.article) args.article = this.entity.article
            if (Object.keys(args).length > 0) {
            url += `?${this.toQueryString(args)}`
            }
            console.log(`getEntity=${url}`)
            return fetch(url).then(resp => resp.json())
         },
         getSummaryInfo() {
            console.log('getSummaryInfo', this.eid, this.entity)
            if (this.entity['summary info'] === undefined && !this.requested.has(this.entity.id)) {
            this.requested.add(this.entity.id)
            this.getEntity()
               .then((updated) => {
                  if (!updated['summary info']) {
                  updated['summary info'] = null
                  }
                  updated.id = this.eid
                  this.$store.dispatch('updateItem', updated)
               })
            }
         }

      },
      /*
      watch: {
         selectedItemID: {
            handler: function (itemID, prior) {
            const layer = Object.values(this.layers).find(layer => layer.options.id === (itemID || prior))
            console.log(`VisNetwork.selectedItemID: value=${itemID} prior=${prior}`)
            },
            immediate: true
         },
         selected: {
            handler: function (value, prior) {
            const map = document.getElementById('map')
            if (this.isSelected && map && map.style.display === 'none') {
               map.style.display = 'block'
            }
            },
            immediate: true
         },
      },
      */
   }
</script>

<style>
   .vis-network {overflow:visible;}

   body {
      color: #d3d3d3;
      font: 12pt arial;
      background-color: #222222;
   }

   #mynetwork {
   width: 600px;
   height: 600px;
   border: 1px solid lightgray;
   }
</style>
