<script>
import { onMounted, ref, inject, nextTick } from "vue";
import {
  props,
  setup as attributionControlSetup,
} from "../functions/controlAttribution";
import { propsBinder, WINDOW_OR_GLOBAL, GLOBAL_LEAFLET_OPT } from "../utils.js";

export default {
  compatConfig: {
    RENDER_FUNCTION: false,
  },
  name: "LControlAttribution",
  props,
  setup(props, context) {
    const leafletRef = ref({});

    const useGlobalLeaflet = inject(GLOBAL_LEAFLET_OPT);
    const registerControl = inject("registerControl");

    const { options, methods } = attributionControlSetup(props, leafletRef);

    onMounted(async () => {
      const { control } = useGlobalLeaflet
        ? WINDOW_OR_GLOBAL.L
        : await import("leaflet/dist/leaflet-src.esm");

      leafletRef.value = control.attribution(options);
      propsBinder(methods, leafletRef.value, props);
      registerControl({ leafletObject: leafletRef.value });
      nextTick(() => context.emit("ready", leafletRef.value));
    });
    return { leafletObject: leafletRef.value };
  },
  render() {
    return null;
  },
};
</script>
