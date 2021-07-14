A 3D web map application created in lecture *Cartography III* at d'BAUG, ETH Zurich, 2020.

This application is built on [Vue](https://vuejs.org/) framework, using [cesium](https://cesium.com) for 3D visualization.

![base](README.img/base.png)

![interface](README.img/interface.png)



## Project setup

```
npm install
```

### Replace OpenWeatherMap key

In file `Source/src/components/CesiumViewer.vue`, replace the following with your own [app id](https://openweathermap.org/api).

```json
weather_appid: "YOUR_OPENWEATHERMAP_APPID"
```



### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Lints and fixes files

```
npm run lint
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).



## [Report](https://github.com/markkua/Skiing-Zermatt/blob/master/Report/Cartography_III_Report_BingxinKe.pdf)



## [Presentation slides](https://github.com/markkua/Skiing-Zermatt/blob/master/Presentation/Skiing_Zermatt.pdf)

