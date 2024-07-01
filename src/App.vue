<script setup>
import { ref, watch } from 'vue'
import '@eox/map'
import '@eox/map/dist/eox-map-advanced-layers-and-sources.js'
import '@eox/map/dist/eox-map.js'
import Feature from 'ol/Feature.js'

const photos = {
  type: 'FeatureCollection',
  features: [
    {
      type: 'Feature',
      geometry: {
        type: 'Point',
        coordinates: [16.178622518003, 48.352363979636]
      },
      properties: {}
    },
    {
      type: 'Feature',
      geometry: {
        type: 'Point',
        coordinates: [16.1783173618171, 48.3518231799334]
      },
      properties: {}
    },
    {
      type: 'Feature',
      geometry: {
        type: 'Point',
        coordinates: [16.1796884872377, 48.3509195144757]
      },
      properties: {}
    }
  ]
}

let selectedFeature = ref(null)
let mapLoaded = ref(false)
let featureCollection = ref(null)
let selectedIndex = ref(0)
let date = ref(null)

const onMapLoadendOnce = () => {
  mapLoaded.value = true
}

const onFeatureSelect = (evt) => {
  const { id, feature } = evt.detail
  if (feature && id === 'selectInteraction') {
    selectedFeature.value = feature
    selectedIndex.value = feature.getId()
    date.value = selectedFeature.value.get('date')
  }
}

const fetchFeatures = async () => {
  try {
    const response = await fetch('https://api.npoint.io/d3e114016d1b10840108')
    featureCollection = await response.json()
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}

fetchFeatures()

const getImages = () => {
  const layers = document.getElementById('eoxMap').map.getLayers()
}

const getClickInteraction = () => {
  return document.getElementById('eoxMap').selectInteractions.selectInteraction
}

const prev = () => {
  selectedIndex.value = Math.max(selectedIndex.value - 1, 0)
  getClickInteraction().highlightById(selectedIndex.value.toString())
  selectedFeature.value = featureCollection.features.find(
    (feature) => feature.id === selectedIndex.value.toString()
  )
  date.value = selectedFeature.value.properties.date
}

const next = () => {
  selectedIndex.value = Math.min(selectedIndex.value + 1, featureCollection.features.length - 1)
  getClickInteraction().highlightById(selectedIndex.value.toString())
  selectedFeature.value = featureCollection.features.find(
    (feature) => feature.id === selectedIndex.value.toString()
  )

  date.value = selectedFeature.value.properties.date
}

const layers = [
  {
    type: 'Vector',
    source: {
      type: 'Vector',
      format: 'GeoJSON',
      url: 'https://api.npoint.io/d3e114016d1b10840108'
    },
    interactions: [
      {
        type: 'select',
        options: {
          id: 'hoverInteraction',
          condition: 'pointermove'
        }
      },
      {
        type: 'select',
        options: {
          id: 'selectInteraction',
          condition: 'click'
        }
      }
    ],
    style: [
      // {
      //   // style: {
      //   //   'icon-src': 'https://www.svgrepo.com/show/904/photo-camera.svg',
      //   //   'icon-width': 20,
      //   //   'icon-height': 20
      //   // }
      // },
      // {
      //   style: {
      //     'circle-radius': 3,
      //     'circle-stroke-color': 'red',
      //     'circle-stroke-width': 1
      //   }
      // },
      {
        style: {
          'icon-src': 'https://svgshare.com/i/17RV.svg',
          'icon-width': 20,
          'icon-height': 30,
          'icon-offset': [0, 5],
          'icon-rotation': ['get', 'image_direction_rad']
        }
      }
    ]
  },
  {
    type: 'Image',
    properties: {
      id: 'imageLayer'
    },
    source: {
      type: 'ImageWMS',
      url: 'https://agri-dev-qaywsx.demo.hub.eox.at/vs/ows',
      params: {
        LAYERS: 'S2L2A__TRUE_COLOR',
        time: '2022-09-07T00:00:00Z/2022-09-07T23:59:59Z'
      }
    }
  }
]
</script>

<template>
  <eox-map
    id="eoxMap"
    :center.props="[16.1783173618171, 48.3518231799334]"
    :layers.props="layers"
    :controls.props="{}"
    :zoom.props="16"
    style="width: 500px; height: 300px; top: 50%; margin-right: 50px"
    @select="onFeatureSelect"
    @loadend.once="onMapLoadendOnce"
    ><eox-map-tooltip></eox-map-tooltip>
  </eox-map>
  <v-carousel
    v-model="selectedIndex"
    v-if="mapLoaded && featureCollection && selectedFeature"
    height="300"
    hide-delimiter-background
    hide-delimiters
    :show-arrows="true"
    continuous="false"
    class="carousel"
  >
    <template v-slot:prev="{ props }">
      <v-btn color="success" variant="elevated" @click="prev">Prev</v-btn>
    </template>
    <template v-slot:next="{ props }">
      <v-btn color="info" variant="elevated" @click="next">Next</v-btn>
    </template>
    <v-carousel-item v-for="(feature, key) in featureCollection.features" :key="key" cover>
      <v-img :src="feature.properties.url" style="max-height: 300px"></v-img>
    </v-carousel-item>
    <h2 class="date">{{ date }}</h2>
  </v-carousel>
  <h2 class="carousel" v-else>Please select a feature on the map!</h2>
</template>

<style scoped>
.carousel {
  position: relative;
  height: 300px;
}
.date {
  position: absolute;
  background-color: white;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1rem;
  padding: 5px 10px;
}

.v-img__img--contain {
  object-fit: cover;
}
</style>
