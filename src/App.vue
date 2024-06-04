<script setup>
import { ref, watch } from 'vue'
import '@eox/map'
import '@eox/map/dist/eox-map-advanced-layers-and-sources.js'
import '@eox/map/dist/eox-map.js'

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

const onMapLoadendOnce = () => {
  mapLoaded.value = true
}

const onFeatureSelect = (evt) => {
  const { id, feature } = evt.detail
  if (feature && id === 'selectInteraction') {
    selectedFeature.value = feature
    selectedIndex.value = feature.getId()
    console.log(selectedFeature.value, selectedIndex.value)
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
  console.log(layers.values)
}

const translatePoint = (distance, angleDegrees) => {
  console.log(angleDegrees)
  let angleRadians = angleDegrees * (Math.PI / 180)

  let x1 = distance * Math.cos(angleRadians)
  let y1 = distance * Math.sin(angleRadians)

  return [x1, y1]
}

watch(selectedIndex, (newIndex) => {
  if (featureCollection.value && featureCollection.value.features[newIndex]) {
    selectedFeature.value = featureCollection.value.features[newIndex]
  }
})

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
      {
        style: {
          'circle-radius': 5,
          'circle-stroke-color': 'black'
        }
        // style: {
        //   'icon-src': 'https://www.svgrepo.com/show/904/photo-camera.svg',
        //   'icon-width': 20,
        //   'icon-height': 20
        // }
      },
      {
        style: {
          'icon-src': 'https://www.svgrepo.com/show/173372/triangle.svg',
          'icon-width': 10,
          'icon-height': 10,
          'icon-rotation': ['get', 'image_direction']
        }
      }
    ]
  },
  { type: 'Tile', source: { type: 'OSM' } }
]
</script>

<template>
  <eox-map
    id="eoxMap"
    :center.props="[16.1783173618171, 48.3518231799334]"
    :layers.props="layers"
    :controls.props="{}"
    :zoom.props="16"
    style="width: 500px; height: 300px"
    @select="onFeatureSelect"
    @loadend.once="onMapLoadendOnce"
    ><eox-map-tooltip></eox-map-tooltip
  ></eox-map>
  <v-carousel
    v-model="selectedIndex"
    v-if="mapLoaded && featureCollection && selectedFeature"
    height="400"
    hide-delimiter-background
    :show-arrows="false"
  >
    <!-- <template v-slot:prev="{ props }">
      <v-btn color="success" variant="elevated" @click="props.onClick">Prev</v-btn>
    </template>
    <template v-slot:next="{ props }">
      <v-btn color="info" variant="elevated" @click="props.onClick">Next</v-btn>
    </template> -->
    <v-carousel-item v-for="(feature, key) in featureCollection.features" :key="key" cover>
      <v-img :src="feature.properties.url"></v-img>
    </v-carousel-item>
    <div>Please select an image</div>
  </v-carousel>
  <h2 v-else>Please select a feature on the map!</h2>
</template>

<style scoped></style>
