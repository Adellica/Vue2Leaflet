<template></template>

<script>
import eventsBinder from '../utils/eventsBinder.js';

const events = ['click'];

const props = {
  visible: {
    type: Boolean,
    custom: true,
    default: true,
  },
  geojson: {
    type: Object,
  },
  options: {
    type: Object,
    default: () => ({}),
  },
};

export default {
  props: props,
  mounted() {
    this.$geoJSON = L.geoJSON(this.geojson, this.options);
    eventsBinder(this, this.$geoJSON, events);

    if (this.$parent._isMounted) {
      this.deferredMountedTo(this.$parent.$geoJSON);
    }
  },
  beforeDestroy() {
    this.setVisible(false);
  },
  watch: {
    geojson: {
      handler(newVal) {
        this.$geoJSON.clearLayers()
        this.addGeoJSONData(newVal);
      },
      deep: true,
    },
    visible(newVal, oldVal) {
      this.setVisible(newVal, oldVal);
    },
  },
  methods: {
    deferredMountedTo(parent) {
      this.parent = parent;
      if (this.visible) {
        this.$geoJSON.addTo(parent);
      }
      for (var i = 0; i < this.$children.length; i++) {
        this.$children[i].deferredMountedTo(parent);
      }
    },
    addGeoJSONData(geojsonData) {
      this.$geoJSON.addData(geojsonData);
    },
    getGeoJSONData() {
      return this.$geoJSON.toGeoJSON();
    },
    getBounds() {
      return this.$geoJSON.getBounds();
    },
    setVisible(newVal, oldVal) {
      if (newVal === oldVal) return;
      if (newVal) {
        this.$geoJSON.addTo(this.parent);
      } else {
        this.parent.removeLayer(this.$geoJSON);
      }
    },
  }
};
</script>
