<template>
  <div class="vue-map">
    <l-map
        ref="map"
        v-model="zoom"
        v-model:zoom="zoom"
        :center="center"
        @update:zoom="zoomUpdated"
        @update:center="centerUpdated"
        @update:bounds="boundsUpdated"
    >
      <l-tile-layer
          v-for="tileProvider in tileProviders"
          :key="tileProvider.name"
          :name="tileProvider.name"
          :visible="tileProvider.visible"
          :url="tileProvider.url"
          :subdomains="tileProvider.subdomains"
          :attribution="tileProvider.attribution"
          :max-zoom="tileProvider.maxZoom"
          :min-zoom="3"
          layer-type="base"/>

      <polyline-decorator
          :lat-lngs="linePaths"
      />
    </l-map>
  </div>
</template>

<script>
import { ref } from 'vue';
import {
  LMap,
  LTileLayer,
} from '@vue-leaflet/vue-leaflet';
import * as L from 'leaflet';
import 'leaflet-polylinedecorator';
import 'leaflet/dist/leaflet.css';
import trenchIcon from './assets/trench-base.svg';

import PolylineDecorator from './components/PolylineDecorator';

export default {
  name: 'App',
  components: {
    LMap,
    LTileLayer,
    PolylineDecorator,
  },
  setup() {
    const center = ref(L.latLng(46.65297120968618, 32.38494873046876));
    const zoom = ref(12);
    const bounds = ref(null);

    function zoomUpdated(zoomNew) {
      zoom.value = zoomNew;
    }

    function centerUpdated(centerNew) {
      center.value = centerNew;
    }

    function boundsUpdated(boundsNew) {
      bounds.value = boundsNew;
    }

    return {
      zoomUpdated,
      centerUpdated,
      boundsUpdated,
      center,
      zoom,
      bounds,
      trenchIcon,
    };
  },
  data: () => ({
    tileProviders: [
      {
        name: 'Цифрова карта',
        visible: true,
        maxZoom: 18,
        attribution:
            '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
        url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      },
      {
        name: 'Супутникові знімки',
        visible: false,
        url: 'https://{s}.google.com/vt/lyrs=s,h&x={x}&y={y}&z={z}',
        subdomains: ['mt0', 'mt1', 'mt2', 'mt3'],
        maxZoom: 20,
      },
    ],
    patterns: [{
      repeat: 15,
      symbol: L.Symbol.marker({
        rotate: false,
        markerOptions: {
          icon: L.icon({
            iconUrl: trenchIcon,
            iconSize: [16, 10],
            iconAnchor: [15, 0],
          }),
        },
      }),
    }],
    linePaths: [[46.6473470, 32.3355103], [46.6332255, 32.3822021], [46.6406687, 32.4106979]],
  }),
};
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Lato&family=Roboto&display=swap');
.vue-map {
  width: 100%;
  height: 100vh;
}
</style>
