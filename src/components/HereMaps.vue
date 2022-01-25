
<template>
  <div id="map">
    <div
      id="mapContainer"
      style="height: 600px; width: 100%"
      ref="hereMap"
    ></div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "HereMap",
  props: {
    center: Object,
  },
  data() {
    return {
      platform: null,
      // You can get the API KEY from developer.here.com
      apikey: "vp4HEEwxi9hzjExMyRLkcR63UmO2dWxLTBrPHt_kzwQ",
      positions: []
    };
  },
  async mounted() {
    const platform = new window.H.service.Platform({
      apikey: this.apikey,
    });
    this.platform = platform;
    this.fetchData();
  },
  methods: {
    initializeHereMap() {
      const mapContainer = this.$refs.hereMap;
      const H = window.H;
      var maptypes = this.platform.createDefaultLayers();

      var map = new H.Map(mapContainer, maptypes.vector.normal.map, {
        zoom: 5,
        center: this.center,
      });
      const ui = H.ui.UI.createDefault(map, maptypes);

      this.startClustering(map, this.positions);

      const svgMarkup =
        '<svg width="16" height="16" ' +
        'xmlns="http://www.w3.org/2000/svg">' +
        '<rect stroke="white" fill="#800080" x="1" y="1" width="16" ' +
        'height="16" /><text x="12" y="18" font-size="12pt" ' +
        'font-family="Arial" font-weight="bold" text-anchor="middle" ' +
        'fill="white">' +
        "</text></svg>";

      let icon = new H.map.Icon(svgMarkup);
      this.positions.forEach((location) => {
        let marker = new H.map.Marker(
          {
            lat: location.LAT,
            lng: location.LON,
          },
          { icon: icon }
        );

        let tooltipDiv =
          "<div>" +
          "<h2>MMSI: " +
          location.MMSI +
          "</h2>" +
          "<br/><b>Speed: " +
          location.SPEED +
          " kn</b>" +
          "<br/><b>Timestamp: " +
          this.toDateString(location.TIMESTAMP);
        +"</b>" + "</div>";
        marker.setData(tooltipDiv);

        marker.addEventListener(
          "tap",
          (evt) => {
            const bubble = new H.ui.InfoBubble(evt.target.getGeometry(), {
              content: evt.target.getData(),
            });
            ui.addBubble(bubble);
          },
          false
        );
        map.addObject(marker);
      });

      addEventListener("resize", () => map.getViewPort().resize());

      new H.mapevents.Behavior(new H.mapevents.MapEvents(map));
    },
    startClustering(map, data) {
      const H = window.H;
      let dataPoints = data.map(function (item) {
        return new H.clustering.DataPoint(item.LAT, item.LON);
      });

      let clusteredDataProvider = new H.clustering.Provider(dataPoints, {
        clusteringOptions: {
          eps: 32,
          minWeight: 2,
        },
      });
      let clusteringLayer = new H.map.layer.ObjectLayer(clusteredDataProvider);
      map.addLayer(clusteringLayer);
    },
    toDateString(timestamp) {
      var parts = timestamp.match(/(\d+)/g);
      return new Date(parts[0]).toUTCString();
    },
    async fetchData() {
      let url =
        "https://services.marinetraffic.com/api/exportvesseltrack/cf8f05df0b57bfae43e762cc61fd381239c4c042/v:3/period:daily/days:5/mmsi:239297000";
      axios
        .get(url)
        .then((response) => {
          let xmlString = response.data;
          var convert = require("xml-js");
          let result1 = convert.xml2json(xmlString, {
            compact: true,
            spaces: 4,
          });
          let tracks = JSON.parse(result1);
          let atr = tracks.VESSELTRACK.POSITION;
          atr.forEach((obj) => {
            this.positions.push(obj._attributes);
          });
          this.initializeHereMap();
        })
        .catch((e) => {
          this.errors.push(e);
        });
      return {};
    }
  },

};
</script>

<style scoped>
#map {
  width: 60vw;
  min-width: 360px;
  text-align: center;
  margin: 5% auto;
  background-color: #ccc;
}
</style>
