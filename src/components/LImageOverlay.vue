<script>
import { onMounted, ref, inject, nextTick } from "vue";
import {
  remapEvents,
  propsBinder,
  WINDOW_OR_GLOBAL,
  GLOBAL_LEAFLET_OPT,
} from "../utils.js";
import {
  props as imageOverlayProps,
  setup as imageOverlaySetup,
} from "../functions/imageOverlay";
import { render } from "../functions/layer";

/**
 * ImageOverlay component, render a plain image instead of a geospatial map.
 */
export default {
  compatConfig: {
    RENDER_FUNCTION: false,
  },
  name: "LImageOverlay",
  props: imageOverlayProps,
  setup(props, context) {
    const leafletRef = ref({});
    const ready = ref(false);

    const useGlobalLeaflet = inject(GLOBAL_LEAFLET_OPT);
    const addLayer = inject("addLayer");

    const { options, methods } = imageOverlaySetup(props, leafletRef, context);

    onMounted(async () => {
      const { imageOverlay, DomEvent } = useGlobalLeaflet
        ? WINDOW_OR_GLOBAL.L
        : await import("leaflet/dist/leaflet-src.esm");
      leafletRef.value = imageOverlay(props.url, props.bounds, options);

      const listeners = remapEvents(context.attrs);
      DomEvent.on(leafletRef.value, listeners);
      propsBinder(methods, leafletRef.value, props);
      addLayer({
        ...props,
        ...methods,
        leafletObject: leafletRef.value,
      });
      ready.value = true;
      nextTick(() => context.emit("ready", leafletRef.value));
    });

    return { ready, leafletObject: leafletRef };
  },
  render() {
    return render(this.ready, this.$slots);
  },
};
</script>
