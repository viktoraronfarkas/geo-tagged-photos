<script setup>
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
          id: 'selectInteraction',
          condition: 'pointermove',
          style: {
            'stroke-color': 'red',
            'stroke-width': 3
          }
        }
      }
    ]
  },
  { type: 'Tile', source: { type: 'OSM' } }
]

const controlsConfig = {
  Zoom: {},
  Attribution: {},
  OverviewMap: {
    collapsed: false,
    layers: [
      {
        type: 'Tile',
        source: {
          type: 'OSM'
        }
      }
    ]
  },
  Geolocation: {
    tracking: true,
    trackHeading: true,
    centerWhenReady: false,
    highAccuracy: true,
    trackAccuracy: true
  }
}
</script>

<template>
  <eox-map
    :center.props="[16.1783173618171, 48.3518231799334]"
    :layers.props="layers"
    :controls.props="controlsConfig"
    :zoom.props="16"
    style="width: 500px; height: 300px"
    ><eox-map-tooltip></eox-map-tooltip
  ></eox-map>
  <v-carousel height="400" hide-delimiter-background show-arrows>
    <template v-slot:prev="{ props }">
      <v-btn color="success" variant="elevated" @click="props.onClick">Prev</v-btn>
    </template>
    <template v-slot:next="{ props }">
      <v-btn color="info" variant="elevated" @click="props.onClick">Next</v-btn>
    </template>
    <v-carousel-item src="https://cdn.vuetifyjs.com/images/cards/docks.jpg" cover></v-carousel-item>

    <v-carousel-item src="https://cdn.vuetifyjs.com/images/cards/hotel.jpg" cover></v-carousel-item>

    <v-carousel-item
      src="https://cdn.vuetifyjs.com/images/cards/sunshine.jpg"
      cover
    ></v-carousel-item>
  </v-carousel>
</template>

<style scoped></style>
