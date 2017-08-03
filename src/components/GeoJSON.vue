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
    this.mapObject = L.geoJSON(this.geojson, this.options);
    eventsBinder(this, this.mapObject, events);

    if (this.$parent._isMounted) {
      this.deferredMountedTo(this.$parent.mapObject);
    }
  },
  beforeDestroy() {
    this.setVisible(false);
  },
  watch: {
    geojson: {
      handler(newVal) {
        this.mapObject.clearLayers()
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
        this.mapObject.addTo(parent);
      }
      for (var i = 0; i < this.$children.length; i++) {
        this.$children[i].deferredMountedTo(parent);
      }
    },
    addGeoJSONData(geojsonData) {
      this.mapObject.addData(geojsonData);
    },
    getGeoJSONData() {
      return this.mapObject.toGeoJSON();
    },
    getBounds() {
      return this.mapObject.getBounds();
    },
    setVisible(newVal, oldVal) {
      if (newVal === oldVal) return;
      if (newVal) {
        this.mapObject.addTo(this.parent);
      } else {
        this.parent.removeLayer(this.mapObject);
      }
    },
  }
};
</script>
