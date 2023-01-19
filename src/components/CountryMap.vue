<template>
  <embedded-map :query="mapQuery" :zoom="areaZoom" :road-type="roadType" @changeRoadType="updateRoadType"></embedded-map>
</template>
<script>
import EmbeddedMap from "./EmbeddedMap.vue";

export default {
  name: "CountryMap",
  components: { EmbeddedMap },
  props: {
    country: {
      type: Object,
      default: null,
    },
    roadType: {
      type : String,
      default: "",
    },
  },
  computed: {
    mapQuery: function () {
      return this.country.name.common;
    },
    areaZoom() {
      const area = this.country.area;

      if (area >= 10000000) return 2;

      if (area >= 1000000) return 3;

      if (area >= 500000) return 4;

      if (area >= 100000) return 5;

      if (area >= 50000) return 6;

      return 9;
    },
  },
  methods: {
    updateRoadType: function (newType) {
      this.$emit("updateRoadType", newType);
    }
  },
};
</script>