<template>
  <div class="cesium-scene">
    <div id="cesium" />
  </div>
</template>
<script>
import { Viewer, Cartesian3, Rectangle } from "cesium";
import * as Cesium from "cesium";

import "cesium/Build/Cesium/Widgets/widgets.css";

import CesiumNavigation from "cesium-navigation-es6"; // compass

const _cesium = {
  viewer: null
};

// var init_date = new Date("2020-11-15 10:00:00+01");
var init_date = new Date("November 15, 2020 9:00 UTC");
// export custom cesium view
export default {
  data() {
    return {
      viewerObserver: null,
      viewerWidth: 0,
      viewerHeight: 0,
      // last_selected_line: null,
      unify_polyline_width: 200,
      default_polyline_material: new Cesium.PolylineGlowMaterialProperty({
        glowPower: 0.01,
        color: Cesium.Color.AQUA,
        outlineWidth: 5
      }),
      init_position: [4404082.080524122, 599281.4023706464, 4567612.496013145],
      // init_date: new Date("2020-11-15 10:00:00+01"),
      flight_cycle: 20, // second
      flight_pitch: Cesium.Math.toRadians(-25),
      flight_distance: 10000, // meter
      flight_center: { longitude: 7.744918, latitude: 46.014724, height: 2000 },
      flight_continue: false,
      // weather
      weather_appid: "YOUR_OPENWEATHERMAP_APPID"
    };
  },
  computed: {
    _cesium() {
      return _cesium;
    }
  },
  mounted() {
    if (_cesium.viewer === null) {
      this.createViewer();
      this.observeSize();
      this.loadScene();
      this.fly_to_zermatt();
    }
  },
  methods: {
    createViewer() {
      _cesium.viewer = new Viewer("cesium", {
        // add cesium options here
        homeButton: true,
        fullscreenButton: false,
        navigationHelpButton: false,
        timeline: true,
        animation: false,
        geocoder: true, // geocoder search
        shouldAnimate: true,
        baseLayerPicker: true,
        selectionIndicator: false
      });
      // _cesium.viewer.scene.globe.depthTestAgainstTerrain = true;
      if (window) {
        window.viewer = _cesium.viewer;
        console.log("viewer created");
      }
      // ************** Navigation and compass ****************
      var options = {};
      // 用于在使用重置导航重置地图视图时设置默认视图控制。接受的值是Cesium.Cartographic 和 Cesium.Rectangle.
      options.defaultResetView = Rectangle.fromDegrees(
        7.63,
        45.95,
        7.87,
        46.05
      );
      // 用于启用或禁用罗盘。true是启用罗盘，false是禁用罗盘。默认值为true。如果将选项设置为false，则罗盘将不会添加到地图中。
      options.enableCompass = true;
      // 用于启用或禁用缩放控件。true是启用，false是禁用。默认值为true。如果将选项设置为false，则缩放控件将不会添加到地图中。
      options.enableZoomControls = true;
      // 用于启用或禁用距离图例。true是启用，false是禁用。默认值为true。如果将选项设置为false，距离图例将不会添加到地图中。
      options.enableDistanceLegend = true;
      // 用于启用或禁用指南针外环。true是启用，false是禁用。默认值为true。如果将选项设置为false，则该环将可见但无效。
      options.enableCompassOuterRing = true;
      CesiumNavigation(viewer, options);

      // infobox css
      var frame = viewer.infoBox.frame;
      frame.addEventListener(
        "load",
        function() {
          var cssLink = frame.contentDocument.createElement("link");
          cssLink.href = Cesium.buildModuleUrl(
            "https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css"
          );
          cssLink.rel = "stylesheet";
          cssLink.type = "text/css";
          frame.contentDocument.head.appendChild(cssLink);
        },
        false
      );
    },
    observeSize() {
      this.viewerObserver = new ResizeObserver(entries => {
        entries.forEach(entry => {
          const { width, height } = entry.contentRect;
          this.viewerWidth = width;
          this.viewerHeight = height;
          this.$nextTick(() => this.onResize());
        });
      });
      this.viewerObserver.observe(this.$el);
    },
    onResize() {
      if (_cesium.viewer) {
        _cesium.viewer.resize();
      }
    },
    // *********************************************************
    //                         Load Data
    // *********************************************************
    loadScene() {
      console.log(`START: loadScene`);
      try {
        // Grant CesiumJS access to your ion assets
        Cesium.Ion.defaultAccessToken =
          "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJmYWIxOGY3NS00NGIzLTRiMTEtOGNmYS1jOGVlYWU1MDJhZDIiLCJpZCI6MzQ2NzUsImlhdCI6MTYwMTg5OTk1MH0.Ha5KDHOrnrgXYRHxnO5h7upVV4vSmR_3KHjp77VZGTQ";

        viewer.clock.currentTime = Cesium.JulianDate.fromDate(init_date);
        console.log("Initial date: ", init_date.toString());
        // 3d terrain
        this.load_3d_terrain();
        // ski route
        this.load_ski_route();
        // highlight selected route
        this.bind_route_selection();
        // 3d building
        this.load_3d_building();
        // image billboards
        this.load_image_billboard();
        // weather
        var weather_station_pos = [
          { lat: 45.9, lon: 7.79 },
          { lat: 46.016096, lon: 7.703184 },
          { lat: 46.038201, lon: 7.685007 },
          { lat: 46.025737, lon: 7.785879 },
          { lat: 45.981793, lon: 7.694278 }
        ];
        this.load_weather(weather_station_pos);
      } catch (error) {
        console.log(`ERROR: loadScene`, error);
      } finally {
        console.log(`END: loadScene`);
      }
    },
    // ***************** Load Ski_route_4326 ***************
    load_3d_terrain() {
      // 3D terrain
      viewer.terrainProvider = Cesium.createWorldTerrain({
        requestVertexNormals: true
      });
      viewer.scene.globe.enableLighting = true;
      console.log("Terrain loaded.");
    },

    // ***************** Load ski routes ***************
    load_ski_route() {
      var ski_route_assetId = 172458;
      // eslint-disable-next-line no-unused-vars
      var line_promise = Cesium.IonResource.fromAssetId(ski_route_assetId).then(
        function(resource) {
          return Cesium.GeoJsonDataSource.load(resource, {
            clampToGround: true
            // stroke: Cesium.Color.AQUA,
            // strokeWidth: 8
          });
        }
      );
      var unify_polyline_width = 200;
      var default_polyline_material = new Cesium.PolylineGlowMaterialProperty({
        glowPower: 0.01,
        color: Cesium.Color.AQUA,
        outlineWidth: 5
      });
      line_promise
        .then(function(dataSource) {
          // Add the new data as entities to the viewer
          // Get the array of entities
          var entities = dataSource.entities.values;
          for (var i = 0; i < entities.length; i++) {
            var entity = entities[i];
            if (Cesium.defined(entity.polyline)) {
              // entity styling code here
              entity.polyline.material = Cesium.clone(
                default_polyline_material
              );
              entity.polyline.show = true;
              entity.polyline.outline = false;
              entity.polyline.width = unify_polyline_width; // include glow width
              entity.name = "No." + (i + 1);
              entity.polyline.clampToGround = true;
            }
          }
          // viewer.dataSources.add(entities);
          console.log("Data ski_route loaded, assetId=", ski_route_assetId);
          window.route_promise = dataSource;
          window.line_entities = dataSource.entities.values;

          // ******** Calculate and update description *********
          // get altitude of start end points
          console.log("Start: Update Description");
          entities = window.line_entities;
          var start_end_points = [];
          for (i = 0; i < entities.length; i++) {
            entity = entities[i];
            // entity.description = "hi";
            var temp_positions = entity.polyline.positions.getValue();
            var temp_cartographic = Cesium.Ellipsoid.WGS84.cartesianArrayToCartographicArray(
              temp_positions
            );
            start_end_points.push(
              Cesium.Cartographic.fromDegrees(
                Cesium.Math.toDegrees(temp_cartographic[0].longitude),
                Cesium.Math.toDegrees(temp_cartographic[0].latitude)
              )
            );
            start_end_points.push(
              Cesium.Cartographic.fromDegrees(
                Cesium.Math.toDegrees(
                  temp_cartographic[temp_cartographic.length - 1].longitude
                ),
                Cesium.Math.toDegrees(
                  temp_cartographic[temp_cartographic.length - 1].latitude
                )
              )
            );
          }
          console.log("1");
          var height_promise = Cesium.sampleTerrainMostDetailed(
            viewer.terrainProvider,
            start_end_points
          );
          // update description
          height_promise.then(function(updatedPositions) {
            console.log("2");
            for (i = 0; i < entities.length; i++) {
              entity = entities[i];
              var h_start = updatedPositions[i * 2].height;
              var h_end = updatedPositions[i * 2 + 1].height;

              if (h_start < h_end) {
                var temp = h_start;
                h_start = h_end;
                h_end = temp;
              }
              var h_delta = h_start - h_end;
              var distance_ls = [];
              var total_distance = 0;
              var positions = entity.polyline.positions.getValue();
              for (let i = 1; i < positions.length; i++) {
                var distance = Cesium.Cartesian3.distance(
                  positions[i - 1],
                  positions[i]
                );
                distance_ls.push(distance);
                total_distance = total_distance + distance;
              }
              var avg_slop = h_delta / total_distance;
              entity.polyline.distance_ls = distance_ls;
              // console.log(entity.distance_ls);
              entity.description =
                '<table class="table table-dark table-striped table-bordered table-hover">' +
                "  <tbody>" +
                "    <tr>" +
                "      <td>Total Length\t</td>" +
                "      <td>" +
                total_distance.toFixed(2) +
                "      </td>" +
                "      <td>(m)</td>" +
                "    </tr>" +
                "    <tr>" +
                "      <td>Start Altitude\t</td>" +
                "      <td>" +
                h_start.toFixed(2) +
                "      </td>" +
                "      <td>" +
                "(m)" +
                "      </td>" +
                "    </tr>" +
                "    <tr>" +
                "      <td>End Altitude\t</td>" +
                "      <td>" +
                h_end.toFixed(2) +
                "      </td>" +
                "      <td>" +
                "(m)" +
                "      </td>" +
                "    </tr>" +
                "    <tr>" +
                "      <td>Altitude Drop\t</td>" +
                "      <td>" +
                Number(h_start - h_end).toFixed(2) +
                "      </td>" +
                "      <td>" +
                "(m)" +
                "      </td>" +
                "    </tr>" +
                "    <tr>" +
                "      <td>Average Slop\t</td>" +
                "      <td>" +
                Number(avg_slop * 100).toFixed(2) +
                "    % </td>" +
                "    </tr>" +
                "  </tbody>\n" +
                "</table>";
              // console.log("description: ", entity.description);
            }
            viewer.dataSources.add(dataSource);
            console.log("End: Update Description");
          });
        })
        .otherwise(function(error) {
          console.log(error);
        });
    },

    // ***************** Load 3D Building ******************
    load_3d_building() {
      // ***************** Load 3D Building ******************
      // eslint-disable-next-line no-unused-vars
      var tileset_3d_building = viewer.scene.primitives.add(
        new Cesium.Cesium3DTileset({
          url: Cesium.IonResource.fromAssetId(96188)
        })
      );
      viewer.scene.primitives.add(tileset_3d_building);
      console.log("3D Building loaded");
    },
    // ********************************************************************
    //                       Route Selection Response
    // ********************************************************************
    bind_route_selection() {
      var scene = viewer.scene;
      if (!scene.pickPositionSupported) {
        window.alert("This browser does not support pickPosition.");
      }
      // bound click event
      var handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
      handler.setInputAction(function(event) {
        try {
          var pickedObject = viewer.scene.pick(event.position);
          if (Cesium.defined(pickedObject.id.polyline)) {
            var picked_line = pickedObject.id.polyline;
            // console.log("position", pickedObject.id.polyline.positions[0]);
            console.log("click on object", pickedObject.id);
            highlight_selection(picked_line);
            ski_along_polyline(picked_line);
          }
        } catch (error) {
          console.log("not valid selection", error);
        }
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
      console.log("Click event bonded.");

      // var last_selected_line; // used for cancel highlight
      var local_unify_polyline_width = this.unify_polyline_width;
      var local_default_polyline_material = this.default_polyline_material;
      function highlight_selection(polyline) {
        clear_last_highlight();
        polyline.width = local_unify_polyline_width * 2;
        // change color
        var new_material = new Cesium.PolylineGlowMaterialProperty({
          glowPower: 0.01,
          color: Cesium.Color.CORAL,
          outlineWidth: 20
        });
        // console.log("new", new_material.color);
        polyline.material = Cesium.clone(new_material);
        window.last_selected_line = polyline;
      }
      // recover last highlight
      function clear_last_highlight() {
        if (Cesium.defined(window.last_selected_line)) {
          window.last_selected_line.material = Cesium.clone(
            local_default_polyline_material
          );
          // console.log("default", default_polyline_material.color);
          window.last_selected_line.width = local_unify_polyline_width;
        }
      }

      function ski_along_polyline(polyline) {
        // console.log("polyline: ", polyline);
        var positions = polyline.positions.getValue();
        // find out which side is higher
        var h_start = positions[0].z;
        var h_end = positions[positions.length - 1].z;
        if (h_start < h_end) {
          positions.reverse();
        }

        viewer.clock.shouldAnimate = true;
        viewer.clock.multiplier = 10;

        (async () => {
          const startTime = viewer.clock.currentTime;
          const hours = 0;
          const minutes = 2;
          const seconds = 0;
          const endTime = startTime.clone();
          Cesium.JulianDate.addSeconds(endTime, seconds, endTime);
          Cesium.JulianDate.addMinutes(endTime, minutes, endTime);
          Cesium.JulianDate.addHours(endTime, hours, endTime);

          // const positions = track.polyline.positions.getValue();
          const startPosition = positions[0];

          const secondsTotal = Cesium.JulianDate.secondsDifference(
            endTime,
            startTime
          );

          const distances = [];
          let distance;

          //calculate the length of each line segment (neglecting earth curvature)
          //source: https://gis.stackexchange.com/questions/175399/cesium-js-line-length/175430
          for (let i = 1; i < positions.length; i++) {
            distance = Cesium.Cartesian3.distance(
              positions[i - 1],
              positions[i]
            );
            distances.push(distance);
          }

          //calculate the total length of the polyline
          const distanceTotal = distances.reduce((a, b) => {
            return a + b;
          });

          // calculate the velocity which is needed that the 3D model reaches
          // the final position in the given time
          const velocity = distanceTotal / secondsTotal;

          const previousPosition = startPosition;
          let previousTime = startTime;

          const positionProperty = new Cesium.SampledPositionProperty();
          //begin the animation at the start position and time
          positionProperty.addSample(previousTime, previousPosition);

          for (let i = 1; i < positions.length; i++) {
            var position = positions[i];
            const currentDistance = distances[i - 1];
            //calculate the time needed to move along this line segment
            const secondsForLineSegment = currentDistance / velocity;

            previousTime = Cesium.JulianDate.addSeconds(
              previousTime,
              secondsForLineSegment,
              new Cesium.JulianDate()
            );
            positionProperty.addSample(previousTime, position);
          }

          var model = viewer.entities.add({
            // name: url,
            position: positionProperty,
            // orientation: orientation,
            name: "OLAF",
            model: {
              uri: "model/scene.gltf", // cautious: don't delete .bin file
              scale: 70,
              heightReference: Cesium.HeightReference.CLAMP_TO_GROUND
            }
          });

          // console.log("point_entity: ", point_entity);
          console.log("model_entity: ", model);
          // viewer.trackedEntity = model;
          // viewer.camera.moveBackward(5000);
          // viewer.zoomTo(model);
        })();
      }
    },

    // ****************** Load image billboards *****************
    load_image_billboard() {
      var image_billboard_list = [];
      var image_ls = [
        {
          longitude: 7.801096,
          latitude: 46.013557,
          url: require("../assets/46.013557_7.801096.png")
        },
        {
          longitude: 7.75293205,
          latitude: 45.993132995137145,
          url: require("../assets/45.993132995137145, 7.75293205417169.jpg")
        },
        {
          longitude: 7.791699258881414,
          latitude: 46.01163748623696,
          url: require("../assets/46.01163748623696, 7.791699258881414.png")
        },
        {
          longitude: 7.756228438827817,
          latitude: 46.02550612890773,
          url: require("../assets/46.02550612890773, 7.756228438827817.png")
        },
        {
          longitude: 7.801096693535728,
          latitude: 46.013557830041854,
          url: require("../assets/46.013557830041854, 7.801096693535728.png")
        },
        {
          longitude: 7.751005009887664,
          latitude: 46.022129482844576,
          url: require("../assets/46.022129482844576, 7.751005009887664.jpg")
        }
      ];
      var i;
      for (i = 0; i < image_ls.length; i++) {
        var image = image_ls[i];
        var temp_billboard = viewer.entities.add({
          position: new Cesium.Cartesian3.fromDegrees(
            image.longitude,
            image.latitude
          ),
          billboard: {
            image: image.url,
            verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
            heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND,
            scaleByDistance: new Cesium.NearFarScalar(1000, 0.4, 20000, 0.02)
          }
        });
        temp_billboard.show = false;
        image_billboard_list.push(temp_billboard);
      }
      window.image_billboard = image_billboard_list;
      console.log("Image billboard loaded");
    },

    // ****************** Load weather *****************
    load_weather(positions) {
      window.weather_label_ls = [];
      for (var i = 0; i < positions.length; i++) {
        var p = positions[i];
        this.load_weather_once(p, positions.length);
      }
    },

    load_weather_once(p, total_length) {
      var url =
        "http://api.openweathermap.org/data/2.5/weather?lat=" +
        p.lat +
        "&lon=" +
        p.lon +
        "&units=metric" +
        "&appid=" +
        this.weather_appid;
      var Http = new XMLHttpRequest();
      Http.open("GET", url);
      Http.send();
      Http.onload = () => {
        var resp_text = Http.responseText;
        // console.log("response: ", resp_text);
        var resp_obj = JSON.parse(resp_text);
        // console.log("weather: ", obj);
        var label_entity = viewer.entities.add({
          position: Cesium.Cartesian3.fromDegrees(p.lon, p.lat, 0),
          label: {
            id: p.lon + "-" + p.lat,
            text:
              resp_obj.weather[0].main +
              " (" +
              resp_obj.weather[0].description +
              ")\nTemperature: " +
              resp_obj.main.temp +
              "°C \n    (" +
              resp_obj.main.temp_min +
              "°C ～ " +
              resp_obj.main.temp_max +
              "°C)\n" +
              "Humidity: " +
              resp_obj.main.humidity +
              "%\n" +
              "Wind: " +
              resp_obj.wind.speed +
              "m/s, " +
              resp_obj.wind.deg +
              "°\n" +
              "Visibility: " +
              resp_obj.visibility +
              "m",
            horizontalOrigin: Cesium.HorizontalOrigin.LEFT,
            verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
            heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND,
            // scale: 1
            scaleByDistance: new Cesium.NearFarScalar(1000, 1, 20000, 0.2)
          },
          point: {
            pixelSize: 20,
            color: Cesium.Color.YELLOW,
            heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND,
            scaleByDistance: new Cesium.NearFarScalar(1000, 1, 20000, 0.02)
          }
        });
        // console.log("label entity: ", label_entity);
        label_entity.label.showBackground = true;
        label_entity.show = false;
        window.weather_label_ls.push(label_entity);
        if (window.weather_label_ls.length === total_length) {
          console.log("Weather loaded");
        }
      };
    },

    log_camera() {
      console.log(
        "Camera status: \nposition: ",
        _cesium.viewer.camera.position,
        ";\ndirection: ",
        _cesium.viewer.camera.direction,
        ";\nup: ",
        _cesium.viewer.camera.up
      );
    },

    fly_to_zermatt() {
      var camera_position = this.init_position;
      var camera_direction = [
        -0.9304258940880441,
        -0.2898300092519881,
        0.224290484299839
      ];
      var camera_up = [
        0.35348030562012994,
        -0.017280294741607624,
        0.9352823450447157
      ];
      _cesium.viewer.camera.flyTo({
        destination: Cartesian3.fromArray(camera_position),
        orientation: {
          direction: new Cartesian3.fromArray(camera_direction),
          up: new Cartesian3.fromArray(camera_up)
        },
        duration: 1
      });
      console.log("Fly to Zermatt");
    },

    start_virtual_flight() {
      var startTime = Cesium.JulianDate.fromDate(init_date);

      // var stopTime = Cesium.JulianDate.addSeconds(startTime, 10, new Cesium.JulianDate());

      viewer.clock.startTime = startTime.clone(); // 开始时间
      // viewer.clock.stopTime = stopTime.clone();     // 结速时间
      viewer.clock.currentTime = startTime.clone(); // 当前时间
      viewer.clock.clockRange = Cesium.ClockRange.CLAMPED; // 行为方式
      viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK; // 时钟设置为当前系统时间; 忽略所有其他设置。

      this.init_heading = viewer.camera.heading;

      this.flight_continue = true;

      viewer.clock.onTick.addEventListener(this.flight_execute);
    },

    flight_execute() {
      var position = Cesium.Cartesian3.fromDegrees(
        this.flight_center.longitude,
        this.flight_center.latitude,
        this.flight_center.height
      );
      // 相机看点的角度，如果大于0那么则是从地底往上看，所以要为负值，这里取-30度
      var pitch = this.flight_pitch;
      // 给定飞行一周所需时间，比如10s, 那么每秒转动度数
      var angle = 360 / this.flight_cycle;
      var distance = this.flight_distance;
      // past time
      var delTime = Cesium.JulianDate.secondsDifference(
        viewer.clock.currentTime,
        viewer.clock.startTime
      );

      var heading = Cesium.Math.toRadians(delTime * angle) + this.init_heading;
      viewer.scene.camera.setView({
        destination: position, // 点的坐标
        orientation: {
          heading: heading,
          pitch: pitch
        }
      });
      viewer.scene.camera.moveBackward(distance);

      if (
        Cesium.JulianDate.compare(
          viewer.clock.currentTime,
          viewer.clock.stopTime
        ) >= 0 ||
        !this.flight_continue
      ) {
        viewer.clock.onTick.removeEventListener(this.flight_execute);
      }
    },

    stop_virtual_flight() {
      this.flight_continue = false;
    },

    show_ski_route() {
      console.log("Show ski routes");
      var entities = window.route_promise.entities.values;
      for (var i = 0; i < entities.length; i++) {
        var entity = entities[i];
        if (Cesium.defined(entity.polyline)) {
          entity.show = true;
        }
      }
    },

    hide_ski_route() {
      console.log("Hide ski routes");
      var entities = window.route_promise.entities.values;
      for (var i = 0; i < entities.length; i++) {
        var entity = entities[i];
        if (Cesium.defined(entity.polyline)) {
          entity.show = false;
        }
      }
    },

    show_image_billboard() {
      // var billboard;
      var i;
      for (i = 0; i < window.image_billboard.length; i++) {
        window.image_billboard[i].show = true;
      }
    },

    hide_image_billboard() {
      var i;
      for (i = 0; i < window.image_billboard.length; i++) {
        window.image_billboard[i].show = false;
      }
    },

    show_weather() {
      for (var i = 0; i < window.weather_label_ls.length; i++) {
        window.weather_label_ls[i].show = true;
      }
    },

    hide_weather() {
      for (var i = 0; i < window.weather_label_ls.length; i++) {
        window.weather_label_ls[i].show = false;
      }
    },

    profile_data(positions, func) {
      var temp_cartographic = Cesium.Ellipsoid.WGS84.cartesianArrayToCartographicArray(
        positions
      );
      var deg_cartographic = [];
      for (var i = 0; i < positions.length; i++) {
        deg_cartographic.push(
          Cesium.Cartographic.fromDegrees(
            Cesium.Math.toDegrees(temp_cartographic[i].longitude),
            Cesium.Math.toDegrees(temp_cartographic[i].latitude)
          )
        );
      }
      var height_promise = Cesium.sampleTerrainMostDetailed(
        viewer.terrainProvider,
        deg_cartographic
      );
      // generate data series
      height_promise.then(function(updatedPositions) {
        var temp_distance_sum = [];
        var data = [];
        for (i = 0; i < positions.length; i++) {
          var distance = 0;
          if (i > 0) {
            distance = Cesium.Cartesian3.distance(
              positions[i - 1],
              positions[i]
            );
          }
          temp_distance_sum = Number(temp_distance_sum) + Number(distance);
          var height = updatedPositions[i].height;
          data.push([temp_distance_sum, height]);
        }
        func(data);
      });
    }
  } //method
};
</script>

<style scoped>
.cesium-scene,
#cesium {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
</style>
