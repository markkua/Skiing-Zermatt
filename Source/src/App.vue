<style>
/*.top_button {*/
/*  !*TODO: why do we need this?*!*/
/*  display: inline-block;*/
/*  position: absolute;*/
/*  top: 5px;*/
/*  left: 5px;*/
/*  !*margin: 10px;*!*/
/*  !*background-color: white;*!*/
/*}*/
</style>
<template>
  <!--  <img alt="Vue logo" src="./assets/logo.png" />-->

  <CesiumViewer ref="CesiumViewer" msg="Welcome to Skiing Zermatt" />
  <!--  add ref to call function in CesiumViewer.vue-->

  <!-- Menu -->
  <div class="top_button btn-group">
    <button
      type="button"
      class="top_button btn btn-primary dropdown-toggle"
      data-toggle="dropdown"
      aria-haspopup="true"
      aria-expanded="false"
    >
      Menu
    </button>
    <div class="dropdown-menu">
      <button
        class="dropdown-item"
        v-on:click="menu_layer_ski_route"
        v-bind:class="{ active: is_ski_active }"
      >
        <span v-html="layer_icon"></span> <b> Ski Route</b>
        <span v-html="ski_tag"></span>
      </button>
      <button
        class="dropdown-item"
        v-on:click="menu_layer_photo"
        v-bind:class="{ active: is_photo_active }"
      >
        <span v-html="layer_icon"></span> <b>Instagram Spot</b>
        <span v-html="photo_tag"></span>
      </button>
      <button
        class="dropdown-item"
        v-on:click="menu_layer_weather"
        v-bind:class="{ active: is_weather_active }"
      >
        <span v-html="layer_icon"></span> <b>Real-time Weather</b>
        <span v-html="weather_tag"></span>
      </button>
      <!--      <a class="dropdown-item" href="#">Instagram Spot</a>-->
      <!--      <a class="dropdown-item" href="#">Weather</a>-->
      <!--      <a class="dropdown-item" href="#">Population Density</a>-->
      <div class="dropdown-divider"></div>
      <button class="dropdown-item" v-on:click="menu_profile_map">
        <span v-html="line_down_icon"></span> Show Profile Map
      </button>
      <div class="dropdown-divider"></div>
      <button class="dropdown-item" v-on:click="menu_fly_to_zermatt">
        <span v-html="geo_icon"></span> Fly to Zermatt
      </button>
      <button class="dropdown-item" v-on:click="menu_virtual_flight">
        <span v-html="cursor_icon"></span> {{ flight_msg }} Virtual Flight
      </button>
      <!--      <div id="log_camera">-->
      <!--        <a class="dropdown-item" v-on:click="menu_log_camera"-->
      <!--          >log Camera Status in console</a-->
      <!--        >-->
      <!--      </div>-->
      <div class="dropdown-divider"></div>
      <button
        id="menu_intro"
        class="dropdown-item"
        data-toggle="modal"
        data-target="#myModal"
      >
        <span v-html="info_icon"></span> Introduction
      </button>
      <button
        id="menu_hidden"
        class="dropdown-item"
        data-toggle="modal"
        data-target="#profile_modal"
        v-show="false"
      >
        hidden
      </button>
    </div>
  </div>

  <!-- Intro Modal -->
  <div
    class="modal fade"
    id="myModal"
    tabindex="-1"
    role="dialog"
    aria-labelledby="myModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h4 class="modal-title" id="myModalLabel">
            Skiing Zermatt - Introduction
          </h4>
        </div>
        <div class="modal-body" align="left">
          <span v-html="intro_text"></span>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-default" data-dismiss="modal">
            Close
          </button>
        </div>
      </div>
      <!-- /.modal-content -->
    </div>
    <!-- /.modal -->
  </div>
  <!-- Profile Map Modal -->
  <div
    class="modal"
    id="profile_modal"
    tabindex="-1"
    role="dialog"
    aria-labelledby="profile_label"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h4 class="modal-title" id="profile_title">
            Profile Map
          </h4>
        </div>
        <div id="main" style="width: 100%;height:400px;"></div>
        <div class="modal-footer">
          <button type="button" class="btn btn-default" data-dismiss="modal">
            Close
          </button>
        </div>
      </div>
      <!-- /.modal-content -->
    </div>
    <!-- /.modal -->
  </div>
</template>

<script>
import CesiumViewer from "./components/CesiumViewer.vue";
var echarts = require("echarts");

var visible_tag = {
  eye:
    ' <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-eye-fill" viewBox="0 0 16 16">\n' +
    '  <path d="M10.5 8a2.5 2.5 0 1 1-5 0 2.5 2.5 0 0 1 5 0z"/>\n' +
    '  <path fill-rule="evenodd" d="M0 8s3-5.5 8-5.5S16 8 16 8s-3 5.5-8 5.5S0 8 0 8zm8 3.5a3.5 3.5 0 1 0 0-7 3.5 3.5 0 0 0 0 7z"/>\n' +
    "</svg>",
  slash:
    ' <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-eye-slash-fill" viewBox="0 0 16 16">\n' +
    '  <path d="M10.79 12.912l-1.614-1.615a3.5 3.5 0 0 1-4.474-4.474l-2.06-2.06C.938 6.278 0 8 0 8s3 5.5 8 5.5a7.029 7.029 0 0 0 2.79-.588zM5.21 3.088A7.028 7.028 0 0 1 8 2.5c5 0 8 5.5 8 5.5s-.939 1.721-2.641 3.238l-2.062-2.062a3.5 3.5 0 0 0-4.474-4.474L5.21 3.089z"/>\n' +
    '  <path d="M5.525 7.646a2.5 2.5 0 0 0 2.829 2.829l-2.83-2.829zm4.95.708l-2.829-2.83a2.5 2.5 0 0 1 2.829 2.829z"/>\n' +
    '  <path fill-rule="evenodd" d="M13.646 14.354l-12-12 .708-.708 12 12-.708.708z"/>\n' +
    "</svg>"
};

var layer_icon_svg =
  '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-layers-half" viewBox="0 0 16 16">\n' +
  '  <path fill-rule="evenodd" d="M3.188 8L.264 9.559a.5.5 0 0 0 0 .882l7.5 4a.5.5 0 0 0 .47 0l7.5-4a.5.5 0 0 0 0-.882L12.813 8l-4.578 2.441a.5.5 0 0 1-.47 0L3.188 8z"/>\n' +
  '  <path fill-rule="evenodd" d="M7.765 1.559a.5.5 0 0 1 .47 0l7.5 4a.5.5 0 0 1 0 .882l-7.5 4a.5.5 0 0 1-.47 0l-7.5-4a.5.5 0 0 1 0-.882l7.5-4zM1.563 6L8 9.433 14.438 6 8 2.567 1.562 6z"/>\n' +
  "</svg>";

var cursor_svg =
  '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-cursor" viewBox="0 0 16 16">\n' +
  '  <path fill-rule="evenodd" d="M14.082 2.182a.5.5 0 0 1 .103.557L8.528 15.467a.5.5 0 0 1-.917-.007L5.57 10.694.803 8.652a.5.5 0 0 1-.006-.916l12.728-5.657a.5.5 0 0 1 .556.103zM2.25 8.184l3.897 1.67a.5.5 0 0 1 .262.263l1.67 3.897L12.743 3.52 2.25 8.184z"/>\n' +
  "</svg>";

var geo_svg =
  '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-geo-alt" viewBox="0 0 16 16">\n' +
  '  <path fill-rule="evenodd" d="M12.166 8.94C12.696 7.867 13 6.862 13 6A5 5 0 0 0 3 6c0 .862.305 1.867.834 2.94.524 1.062 1.234 2.12 1.96 3.07A31.481 31.481 0 0 0 8 14.58l.208-.22a31.493 31.493 0 0 0 1.998-2.35c.726-.95 1.436-2.008 1.96-3.07zM8 16s6-5.686 6-10A6 6 0 0 0 2 6c0 4.314 6 10 6 10z"/>\n' +
  '  <path fill-rule="evenodd" d="M8 8a2 2 0 1 0 0-4 2 2 0 0 0 0 4zm0 1a3 3 0 1 0 0-6 3 3 0 0 0 0 6z"/>\n' +
  "</svg>";

var info_circle_svg =
  '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-info-circle" viewBox="0 0 16 16">\n' +
  '  <path fill-rule="evenodd" d="M8 15A7 7 0 1 0 8 1a7 7 0 0 0 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z"/>\n' +
  '  <path d="M8.93 6.588l-2.29.287-.082.38.45.083c.294.07.352.176.288.469l-.738 3.468c-.194.897.105 1.319.808 1.319.545 0 1.178-.252 1.465-.598l.088-.416c-.2.176-.492.246-.686.246-.275 0-.375-.193-.304-.533L8.93 6.588z"/>\n' +
  '  <circle cx="8" cy="4.5" r="1"/>\n' +
  "</svg>";

var graph_down_svg =
  '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-graph-down" viewBox="0 0 16 16">\n' +
  '  <path fill-rule="evenodd" d="M0 0h1v15h15v1H0V0zm10 11.5a.5.5 0 0 0 .5.5h4a.5.5 0 0 0 .5-.5v-4a.5.5 0 0 0-1 0v2.6l-3.613-4.417a.5.5 0 0 0-.74-.037L7.06 8.233 3.404 3.206a.5.5 0 0 0-.808.588l4 5.5a.5.5 0 0 0 .758.06l2.609-2.61L13.445 11H10.5a.5.5 0 0 0-.5.5z"/>\n' +
  "</svg>";

export default {
  name: "App",
  components: {
    CesiumViewer
  },
  data: function() {
    return {
      is_ski_active: true,
      ski_tag: visible_tag.eye,
      is_photo_active: false,
      photo_tag: visible_tag.slash,
      is_weather_active: false,
      weather_tag: visible_tag.slash,
      is_flight_active: false,
      flight_msg: "Start",
      modal_hidden: false,
      // icons
      layer_icon: layer_icon_svg,
      cursor_icon: cursor_svg,
      geo_icon: geo_svg,
      info_icon: info_circle_svg,
      line_down_icon: graph_down_svg,
      // echart
      orgOptions: {},
      // content
      intro_text:
        "This project, Skiing Zermatt, is a 3D web map application aiming at helping young people discover ski trails that suit them in Zermatt. The biggest upgrade is the ski route interaction on a real 3D terrain, compared with other existing ski maps." +
        "<br><br>" +
        "This map provides a fancy overview of ski routes. Glowing lines, free perspective of true 3D mountains, skiing animation and a calculated profile map give users an overview of the ski route about the length, location and difficulty. At the same time, real-time weather information is provided." +
        "<br><br>" +
        "Considering the needs of young people taking Instagram pictures, this map not only provides ski route information, but also gives photograph spots. They can see pictures uploaded by other users and where they were taken. (In this version, only a small number of pictures are used, and uploading is disabled.)" +
        "<br><br>" +
        "The UI design takes our target user, young people, into account. Firstly, it’s totally compatibility on mobile end. Secondly, the interface is neat, fancy and easy to use."
    };
  },
  mounted: function() {
    document.getElementById("menu_intro").click();
    console.log("app mounted");
  },
  methods: {
    emit_intro_modal() {},
    menu_layer_ski_route() {
      if (this.is_ski_active) {
        this.is_ski_active = false;
        this.ski_tag = visible_tag.slash;
        this.$refs.CesiumViewer.hide_ski_route();
      } else {
        this.is_ski_active = true;
        this.ski_tag = visible_tag.eye;
        this.$refs.CesiumViewer.show_ski_route();
      }
    },

    menu_layer_photo() {
      if (this.is_photo_active) {
        this.is_photo_active = false;
        this.photo_tag = visible_tag.slash;
        this.$refs.CesiumViewer.hide_image_billboard();
      } else {
        this.is_photo_active = true;
        this.photo_tag = visible_tag.eye;
        this.$refs.CesiumViewer.show_image_billboard();
      }
    },

    menu_layer_weather() {
      if (this.is_weather_active) {
        this.is_weather_active = false;
        this.weather_tag = visible_tag.slash;
        this.$refs.CesiumViewer.hide_weather();
      } else {
        this.is_weather_active = true;
        this.weather_tag = visible_tag.eye;
        this.$refs.CesiumViewer.show_weather();
      }
    },

    menu_fly_to_zermatt() {
      this.$refs.CesiumViewer.fly_to_zermatt();
    },

    menu_virtual_flight() {
      if (this.is_flight_active) {
        this.is_flight_active = false;
        this.flight_msg = "Start";
        this.$refs.CesiumViewer.stop_virtual_flight();
      } else {
        this.is_flight_active = true;
        this.flight_msg = "Stop";
        this.$refs.CesiumViewer.start_virtual_flight();
      }
    },

    menu_log_camera() {
      this.$refs.CesiumViewer.log_camera();
    },

    menu_profile_map() {
      if (typeof window.last_selected_line == "undefined") {
        alert("Invalid selection! Please select a ski track first.");
      } else {
        var positions = window.last_selected_line.positions.getValue();
        this.$refs.CesiumViewer.profile_data(positions, func);
      }
      function func(data) {
        console.log("return data: ", data);
        document.getElementById("menu_hidden").click();
        var myChart = echarts.init(document.getElementById("main"));
        // 绘制图表
        myChart.setOption(
          {
            xAxis: {
              type: "value"
            },
            yAxis: {
              type: "value"
            },
            series: [
              {
                data: data,
                type: "line",
                areaStyle: {}
              }
            ]
          },
          true
        );
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.top_button {
  /*TODO: why do we need this?*/
  display: inline-block;
  position: absolute;
  top: 5px;
  left: 5px;
  /*margin: 10px;*/
  /*background-color: white;*/
}
</style>
