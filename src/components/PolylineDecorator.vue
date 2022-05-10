<template>
  <div style="display: none;">
    <slot v-if="ready"></slot>
  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet-polylinedecorator';
import { propsBinder, remapEvents } from '@vue-leaflet/vue-leaflet/src/utils';
import {
  inject, ref, onMounted, getCurrentInstance,
} from 'vue';

export default {
  name: 'PolylineDecorator',
  props: {
    paths: {
      type: Array,
      default: () => [],
    },
    patterns: {
      type: Array,
      default: () => [],
    },
    visible: {
      type: Boolean,
      custom: true,
      default: true,
    },
  },
  setup(props) {
    const ready = ref(false);
    const addLayer = inject('addLayer');
    const instance = getCurrentInstance();

    const polyline = L.polyline(props.paths);
    const options = { patterns: props.patterns };
    const mapObject = L.polylineDecorator(polyline, options);
    const listeners = remapEvents(instance.parent.attrs);

    onMounted(async () => {
      const { DomEvent } = await import('leaflet/dist/leaflet-src.esm');
      const methods = {
        setSmoothFactor(smoothFactor) {
          mapObject.setStyle({ smoothFactor });
        },
        setNoClip(noClip) {
          mapObject.setStyle({ noClip });
        },
        addLatLng(latLng) {
          mapObject.addLatLng(latLng);
        },
      };

      DomEvent.on(mapObject, listeners);

      propsBinder(methods, mapObject, props);
      ready.value = true;

      addLayer({
        ...methods,
        ...props,
        leafletObject: mapObject,
      }, false);
    });

    return {
      ready,
    };
  },
};
</script>
