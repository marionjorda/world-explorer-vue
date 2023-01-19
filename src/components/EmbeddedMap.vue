<template>
  <div>
    <select v-model="selectRoad">
      <option value="">Carte Routière</option>
      <option value="k">Satellite</option>
    </select>
    <iframe
      :src="mapSrc"
      frameborder="0"
      scrolling="no"
      marginheight="0"
      marginwidth="0"
      class="map-container"
    ></iframe>
  </div>
</template>
<script>
export default {
  name: "EmbeddedMap",
  props: {
    query: {
      type: String,
      default: "Place Bellecour, Lyon, France",
    },
    zoom: {
      type: Number,
      default: 2,
    },
    roadType:{
      type: String,
      default: "",
    },
  },
  computed: {
    mapSrc: function () {
      return `https://maps.google.com/maps?q=${this.query}&t=${this.selectRoad}&z=${this.zoom}&ie=UTF8&iwloc=&output=embed`;
    },
  },
  data() {
    return {
      selectRoad: ""
    }
  },
  mounted() {
    this.selectRoad = this.roadType;
  },
  watch: {
    roadType: function (newType) {
      this.selectRoad = newType;
    },
    selectRoad: function (newType) {
      this.$emit("changeRoadType", newType);
    },
  },
};
</script>
<style scoped>
.map-container {
  width: 100%;
  height: 500px;
}
</style>