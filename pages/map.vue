<template>
  <div>
    <div id="map">
      <div class="right-card">
        <!-- <v-card max-width="200" class="pa-2"> -->
        <!-- <v-card-text> -->
        <v-container class="pa-2">
          <v-row>
            <input type="checkbox" id="checkbox" v-model="checked" />
            <label for="checkbox">Heatmap?</label>
          </v-row>
          <v-row class="justify-end">
            <v-btn light elevation="2" @click="simulateParticles">
              Simulate
            </v-btn>
          </v-row>
          <!-- <v-row class="justify-end">
            <v-btn light elevation="2" @click="clearAll"> Clear All </v-btn>
          </v-row> -->
          <!-- <v-row class="justify-end">
            <v-btn light dense elevation="2" @click=""> Random 50 </v-btn>
          </v-row> -->
        </v-container>
        <!-- </v-card-text> -->
        <!-- </v-card> -->
      </div>

      <div id="left" class="sidebar flex-center left collapsed">
        <div
          style="background: #003755"
          class="sidebar-content rounded-rect flex-center"
        >
          <v-container class="pa-2 text-center">
            <v-row>
              <v-col class="text-h4 font-weight-bold text-uppercase text-left">
                CyberWaves
              </v-col>
            </v-row>
            <v-row>
              <v-divider></v-divider>
            </v-row>
            <v-row>
              <v-col class="text-justify text-subtitle-2 font-weight-light">
                This application simulates the trajectory of pieces of plastic
                over a course of 30 days using real world oceanographic data and
                physical equations via the OceanParcels library. This can be
                used as a research tool, for ocean cleanup organisations, as
                well as for general education on the ocean pollution issue.
              </v-col>
            </v-row>
            <v-row>
              <v-col class="text-left">
                <v-divider class="mb-3"></v-divider>
                <p class="text-h6">Share this map</p>
                <p>
                  Lorem ipsum dolor sit amet consectetur adipisicing elit. Alias
                  libero odit fuga, id magnam voluptates harum nobis quas in?
                  Nesciunt.
                </p>

                <div>
                  <v-icon>mdi-facebook</v-icon>
                  <v-icon>mdi-instagram</v-icon>
                  <v-icon>mdi-whatsapp</v-icon>
                  <v-icon>mdi-github</v-icon>
                </div>

                <v-divider class="mt-3"></v-divider>
              </v-col>
            </v-row>
            <v-row>
              <v-col class="text-left">
                <p class="text-h6">Credits</p>
                <a href="https://ocean.ust.hk:8443/SiteMapApi/new/index.jsp"
                  >Hong Kong Universtiy of Science and Technology</a
                >
                <a href="https://oceanparcels.org/">Ocean Parcels</a>

                <v-divider></v-divider>
              </v-col>
            </v-row>
          </v-container>

          <div class="sidebar-toggle rounded-rect left" @click="toggleSidebar">
            <v-icon color="blue" dense> mdi-arrow-collapse-right </v-icon>
            <!-- <v-icon color="blue" dense v-else> mdi-arrow-collapse-left </v-icon> -->
          </div>
        </div>
      </div>
      <v-overlay :value="isSimulating">
        <v-progress-circular
          indeterminate
          color="primary"
          v-if="isSimulating"
        ></v-progress-circular>
      </v-overlay>
    </div>
  </div>
</template>
<style scoped>
.right-card {
  position: absolute;
  z-index: 2;
  right: 0;
  bottom: 0;
}

* {
  margin: 0;
  padding: 0;
}

#map {
  position: relative;
  width: 100%;
  height: 100vh;
}
.flex-center {
  position: absolute;
  display: flex;
  justify-content: left;
  align-items: center;
}

.flex-center.left {
  left: 0px;
}

.sidebar-content {
  position: absolute;
  width: 95%;
  height: 100%;
}

.sidebar-toggle {
  position: absolute;
  width: 1em;
  height: 1em;
  font-size: 32px;
  overflow: visible;
  display: flex;
  justify-content: center;
  align-items: center;
}

.sidebar-toggle.left {
  right: -1.1em;
}

.sidebar {
  transition: transform 1s;
  z-index: 100;
  width: 300px;
  height: 100%;
}

.left.collapsed {
  transform: translateX(-295px);
}

.rounded-rect {
  background: white;
  border-radius: 10px;
  box-shadow: 0 0 50px -25px black;
}
</style>

<script>
import mapboxgl from "mapbox-gl";
import MapboxDraw from "@mapbox/mapbox-gl-draw";
import "@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css";

class extendDrawBar {
  constructor(opt) {
    let ctrl = this;
    ctrl.draw = opt.draw;
    ctrl.buttons = opt.buttons || [];
    ctrl.onAddOrig = opt.draw.onAdd;
    ctrl.onRemoveOrig = opt.draw.onRemove;
  }
  onAdd(map) {
    let ctrl = this;
    ctrl.map = map;
    ctrl.elContainer = ctrl.onAddOrig(map);
    ctrl.buttons.forEach((b) => {
      ctrl.addButton(b);
    });
    return ctrl.elContainer;
  }
  onRemove(map) {
    let ctrl = this;
    ctrl.buttons.forEach((b) => {
      ctrl.removeButton(b);
    });
    ctrl.onRemoveOrig(map);
  }
  addButton(opt) {
    let ctrl = this;
    var elButton = document.createElement("button");
    var elIcon = document.createElement("i");
    // var oText = document.createTextNode("mdi-plus");
    // elIcon.appendChild(oText);
    // elIcon.setAttribute("color", "yellow");
    elButton.className = "mapbox-gl-draw_ctrl-draw-btn";
    if (opt.classes instanceof Array) {
      opt.classes.forEach((c) => {
        elIcon.classList.add(c);
      });
    }
    elButton.addEventListener(opt.on, opt.action);
    elButton.appendChild(elIcon);
    ctrl.elContainer.appendChild(elButton);

    opt.elButton = elButton;
  }
  removeButton(opt) {
    opt.elButton.removeEventListener(opt.on, opt.action);
    opt.elButton.remove();
  }
}

export default {
  data() {
    return {
      access_token:
        "pk.eyJ1IjoiamFtZXMwMDdsb2wiLCJhIjoiY2w3ZzF2eTRvMDA5NDNwcGJia3UzaWw3OSJ9.ypuRCdADgICM3tv2JFwEBg",
      map: {},
      center: [114.1694, 22.3193],
      isCollapsed: true,
      mapLoaded: false,
      isSimulating: false,
      checked: false,
    };
  },
  methods: {
    clearAll() {
      this.draw.deleteAll();
    },
    toggleSidebar() {
      const elem = document.getElementById("left");
      const collapsed = elem.classList.toggle("collapsed");
      const padding = {};
      padding["left"] = collapsed ? 0 : 300;
      this.map.easeTo({
        padding: padding,
        duration: 1000,
      });
    },
    getPoints() {
      const data = this.draw.getAll();
      console.log(data);
      const coords = data.features.map((obj) => {
        return obj.geometry.coordinates;
      });
      console.log(coords);
      //   [[lon, lat]]
      return coords;
    },
    createMap() {
      mapboxgl.accessToken = this.access_token;

      this.map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/james007lol/cl797cotb000214rrhdkv7byt",
        zoom: 11,
        center: this.center,
        attributionControl: false,
      });

      this.map.addControl(
        new mapboxgl.AttributionControl({ compact: true }),
        "bottom-left"
      );
      this.map.addControl(new mapboxgl.NavigationControl(), "top-left");

      var LotsOfPointsMode = {};

      // When the mode starts this function will be called.
      // The `opts` argument comes from `draw.changeMode('lotsofpoints', {count:7})`.
      // The value returned should be an object and will be passed to all other lifecycle functions
      LotsOfPointsMode.onSetup = function (opts) {
        var state = {};
        state.count = opts.count || 0;
        return state;
      };

      // Whenever a user clicks on the map, Draw will call `onClick`
      LotsOfPointsMode.onClick = function (state, e) {
        // `this.newFeature` takes geojson and makes a DrawFeature
        var point = this.newFeature({
          type: "Feature",
          properties: {
            count: state.count,
          },
          geometry: {
            type: "Point",
            coordinates: [e.lngLat.lng, e.lngLat.lat],
          },
        });
        this.addFeature(point); // puts the point on the map
      };

      // Whenever a user clicks on a key while focused on the map, it will be sent here
      LotsOfPointsMode.onKeyUp = function (state, e) {
        if (e.keyCode === 27) return this.changeMode("simple_select");
      };

      // This is the only required function for a mode.
      // It decides which features currently in Draw's data store will be rendered on the map.
      // All features passed to `display` will be rendered, so you can pass multiple display features per internal feature.
      // See `styling-draw` in `API.md` for advice on making display features
      LotsOfPointsMode.toDisplayFeatures = function (state, geojson, display) {
        display(geojson);
      };

      this.draw = new MapboxDraw({
        defaultMode: "lots_of_points",
        displayControlsDefault: false,
        userProperties: true,
        modes: Object.assign(
          {
            lots_of_points: LotsOfPointsMode,
          },
          MapboxDraw.modes
        ),
        styles: [
          {
            id: "gl-draw-point",
            type: "circle",
            filter: ["all", ["==", "$type", "Point"]],
            paint: {
              "circle-radius": 4,
              "circle-color": "#ffa500",
            },
          },
        ],
      });
      // var drawBar = new extendDrawBar({
      //   draw: this.draw,
      //   buttons: [
      //     {
      //       on: "click",
      //       classes: ["fa-solid", "fa-location-dot", "black--text"],
      //     },
      //   ],
      // });

      this.map.addControl(this.draw, "top-right");
      //   this.map.addLayer({});
      //   this.map.on("load");

      this.map.on("draw.create", this.getPoints);
      this.map.on("load", () => {
        this.mapLoaded = true;
      });
      //   this.map.on("draw.update", this.getPoints);
      //   this.map.on("draw.delete", this.getPoints);

      //   function getPoints(e){
      //     const data = draw.getAll();
      //     console.log(data)
      //   }
      //   this.map.on("load", async () => {
      //     const json = JSON.stringify({
      //       particles: [
      //         [22.300483, 114.204068],
      //         [22.300485, 114.204065],
      //       ],
      //     });
      //     let res = await this.$axios.$post("/execute", json, {
      //       headers: { "Content-Type": "application/json" },
      //     });
      //     console.log(res);
      //     this.map.addSource("route", {
      //       type: "geojson",
      //       data: {
      //         type: "Feature",
      //         properties: {},
      //         geometry: {
      //           type: "LineString",
      //           coordinates: res.trajectories[1],
      //         },
      //       },
      //     });
      //     this.map.addLayer({
      //       id: "route",
      //       type: "line",
      //       source: "route",
      //       layout: {
      //         "line-join": "round",
      //         "line-cap": "round",
      //       },
      //       paint: {
      //         "line-color": "#FFBF00",
      //         "line-width": 8,
      //       },
      //     });
      //   });
    },
    async simulateParticles() {
      this.isSimulating = true;

      // simulate
      const coords = this.getPoints();
      const json = JSON.stringify({
        particles: coords,
      });
      console.log(json);
      let res = await this.$axios.$post("/execute", json, {
        headers: { "Content-Type": "application/json" },
      });
      this.isSimulating = false;
      console.log(res.trajectories);
      const trajectories = res.trajectories;
      //   console.log(trajectories.entries());
      //   for (let trajectory in trajectories) {
      //     console.log("hello");
      //   }

      trajectories.forEach((trajectory, index) => {
        var uniqueId = "trace-" + index;
        var uniqueId2 = "trace2-" + index;
        console.log(uniqueId);
        console.log(uniqueId2);
        // Add path
        this.map.addSource(uniqueId, {
          type: "geojson",
          data: {
            type: "Feature",
            properties: {},
            geometry: {
              type: "LineString",
              coordinates: trajectory,
            },
          },
        });

        // Add end point
        this.map.addSource(uniqueId2, {
          type: "geojson",
          data: {
            type: "FeatureCollection",
            features: [
              {
                type: "Feature",
                geometry: {
                  type: "Point",
                  coordinates: trajectory[trajectory.length - 1],
                },
              },
            ],
          },
        });
        // Path layer
        this.map.addLayer({
          id: uniqueId,
          type: "line",
          source: uniqueId,
          layout: {
            "line-join": "round",
            "line-cap": "round",
          },
          paint: {
            "line-color": "yellow",
            "line-opacity": 0.5,
            "line-width": 1,
          },
        });

        // endpoint layer
        if (this.checked === true) {
          // use heatmap
          this.map.addLayer({
            id: uniqueId2,
            type: "heatmap",
            source: uniqueId2,
            maxzoom: 15,
            paint: {
              // increase weight as diameter breast height increases
              "heatmap-weight": {
                property: "dbh",
                type: "exponential",
                stops: [
                  [1, 0],
                  [62, 1],
                ],
              },
              // increase intensity as zoom level increases
              "heatmap-intensity": {
                stops: [
                  [11, 1],
                  [15, 3],
                ],
              },
              // assign color values be applied to points depending on their density
              "heatmap-color": [
                "interpolate",
                ["linear"],
                ["heatmap-density"],
                0,
                "rgba(236,222,239,0)",
                0.2,
                "rgb(208,209,230)",
                0.4,
                "rgb(166,189,219)",
                0.6,
                "rgb(103,169,207)",
                0.8,
                "rgb(28,144,153)",
              ],
              // increase radius as zoom increases
              "heatmap-radius": {
                stops: [
                  [11, 15],
                  [15, 20],
                ],
              },
              // decrease opacity to transition into the circle layer
              "heatmap-opacity": {
                default: 1,
                stops: [
                  [14, 1],
                  [15, 0],
                ],
              },
            },
          });
          this.map.addLayer({
            id: uniqueId2 + "point",
            type: "circle",
            source: uniqueId2,
            minzoom: 14,
            paint: {
              // increase the radius of the circle as the zoom level and dbh value increases
              "circle-radius": {
                property: "dbh",
                type: "exponential",
                stops: [
                  [{ zoom: 15, value: 1 }, 5],
                  [{ zoom: 15, value: 62 }, 10],
                  [{ zoom: 22, value: 1 }, 20],
                  [{ zoom: 22, value: 62 }, 50],
                ],
              },
              "circle-color": {
                property: "dbh",
                type: "exponential",
                stops: [
                  [0, "rgba(236,222,239,0)"],
                  [10, "rgb(236,222,239)"],
                  [20, "rgb(208,209,230)"],
                  [30, "rgb(166,189,219)"],
                  [40, "rgb(103,169,207)"],
                  [50, "rgb(28,144,153)"],
                  [60, "rgb(1,108,89)"],
                ],
              },
              "circle-stroke-color": "white",
              "circle-stroke-width": 1,
              "circle-opacity": {
                stops: [
                  [14, 0],
                  [15, 1],
                ],
              },
            },
          });
        } else {
          this.map.addLayer({
            id: uniqueId2,
            type: "circle",
            source: uniqueId2,
            paint: {
              "circle-radius": 4,
              "circle-color": "#ff0000",
            },
          });
        }
      });

      // for (const [index, trajectory] in trajectories.entries()) {
      //   console.log(index);

      // }
    },
  },
  mounted() {
    this.createMap();
    // this.getData();
  },
  head: {
    link: [
      {
        rel: "stylesheet",
        href: "https://api.mapbox.com/mapbox-gl-js/v2.10.0/mapbox-gl.css",
      },
    ],
    script: [
      {
        src: "https://kit.fontawesome.com/f1e2cbc899.js",
      },
    ],
  },
};
</script>
