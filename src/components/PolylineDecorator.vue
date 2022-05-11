<script>
import 'leaflet-polylinedecorator';
import {
  GLOBAL_LEAFLET_OPT, propsBinder, remapEvents, WINDOW_OR_GLOBAL,
} from '@vue-leaflet/vue-leaflet/src/utils';
import {
  inject, ref, onMounted, nextTick,
} from 'vue';
import { props as propsData, setup as polylineSetup } from '@vue-leaflet/vue-leaflet/src/functions/polyline';
import { render } from '@vue-leaflet/vue-leaflet/src/functions/layer';
import * as L from 'leaflet';

import trenchIcon from '../assets/trench-base.svg';

export default {
  name: 'PolylineDecorator',
  props: propsData,
  setup(props, context) {
    const leafletRef = ref({});
    const ready = ref(false);

    const useGlobalLeaflet = inject(GLOBAL_LEAFLET_OPT);
    const addLayer = inject('addLayer');

    const { methods } = polylineSetup(props, leafletRef, context);

    onMounted(async () => {
      const { DomEvent } = useGlobalLeaflet
        ? WINDOW_OR_GLOBAL.L
        : await import('leaflet/dist/leaflet-src.esm');

      leafletRef.value = L.polylineDecorator(props.latLngs, {
        patterns: [{
          offset: 20,
          repeat: '15px',
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
      });

      const listeners = remapEvents(context.attrs);
      DomEvent.on(leafletRef.value, listeners);

      propsBinder(methods, leafletRef.value, props);

      addLayer({
        ...props,
        ...methods,
        leafletObject: leafletRef.value,
      });
      ready.value = true;
      await nextTick(() => context.emit('ready', leafletRef.value));
    });
    return { ready, leafletObject: leafletRef };
  },
  render() {
    return render(this.ready, this.$slots);
  },
};
</script>
