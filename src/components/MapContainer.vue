<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import AMapLoader from '@amap/amap-jsapi-loader'

let map = null
let polyEditor = null
let currentPoly = null

onMounted(() => {
  initMap()
})

function initMap() {
  AMapLoader.load({
    key: '', // 申请好的Web端开发者Key，首次调用 load 时必填
    version: '2.0', // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
    plugins: [
      'AMap.PolygonEditor',
      'AMap.Scale',
      'AMap.ToolBar',
      'AMap.CitySearch',
    ], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
  })
    .then(async AMap => {
      map = await new AMap.Map('container', {
        // 设置地图容器id
        viewMode: '3D', // 是否为3D地图模式
        zoom: 11, // 初始化地图级别
        // center: [116.397428, 39.90923], // 初始化地图中心点位置
      })
      console.log('🚀 ~ initMap ~ map:', map)

      // var scale = new AMap.Scale()
      // map.addControl(scale)

      // var toolbar = new AMap.ToolBar() //创建工具条插件实例
      // map.addControl(toolbar) //添加工具条插件到页面

      addInitPoly()

      showCityInfo()

      // 初始化多变形编辑器
      polyEditor = new AMap.PolygonEditor(map)
    })
    .catch(e => {
      console.log(e)
    })
}

// 初始化电子围栏
function addInitPoly() {
  // 本地化的数据
  const path1 = localStorage.getItem('paths')
  let pathParser = []

  if (path1) {
    pathParser = JSON.parse(path1)
  }

  var path = [
    [
      [113.661648, 34.762461],
      [113.660703, 34.750555],
      [113.664292, 34.750353],
      // [113.589846, 34.761365],
    ],
    [
      [113.653322, 34.760255],
      [113.660703, 34.757555],
      [113.652292, 34.752353],
      [113.649846, 34.751365],
    ],
  ]

  const conf = {
    strokeWeight: 3,
    strokeOpacity: 0.2,
    fillOpacity: 0.4,
    zIndex: 50,
    bubble: true,
    strokeColor: '#FF33FF',
    fillColor: '#1791fc',
  }

  // 构造多边形
  const polygons = pathParser.map(
    path =>
      new AMap.Polygon({
        path,
        ...conf,
      })
  )

  // 添加矢量图层
  map.add([...polygons])

  // 缩放地图到合适的视野级别
  map.setFitView()

  // polygon.on('click', () => {
  //   polyEditor.setTarget(polygon)
  //   polyEditor.open()
  // })
  // polygon1.on('click', () => {
  //   polyEditor.setTarget(polygon1)
  //   polyEditor.open()
  // })

  // 给多边形绑定单击事件
  polygons.forEach(polygon => {
    polygon.on('click', () => {
      polyEditor.setTarget(polygon)
      polyEditor.open()
      currentPoly = polygon
    })
  })

  // 如果需要在打开地图时就显示编辑状态解开下面注释
  // polyEditor.addAdsorbPolygons(polygon)
  // polyEditor.open()

  // 创建信息框
  // let infoWindow = new AMap.InfoWindow({
  //   content: '22', //使用默认信息窗体框样式，显示信息内容
  //   position: [113.661648, 34.762461],
  // })
  // infoWindow.open(map)
}

// 创建
function createPolygon() {
  polyEditor.close()
  polyEditor.setTarget()
  polyEditor.open()
}

// 保存
function savePolygon() {
  const paths = map.getAllOverlays('polygon').map(polygon => polygon.getPath())
  localStorage.setItem('paths', JSON.stringify(paths))
}

// 删除
function removePolygon() {
  console.log('🚀 ~ removePolygon ~ currentPoly:', currentPoly)
  if (currentPoly) {
    // polyEditor.removeAdsorbPolygons(currentPoly.getPath())
    map.remove(currentPoly)
  }
}

onUnmounted(() => {
  map?.destroy()
})

//获取用户所在城市信息
function showCityInfo() {
  console.log('获取用户所在城市信息', map.getCenter())

  //实例化城市查询类
  var citysearch = new AMap.CitySearch()
  //自动获取用户IP，返回当前城市
  citysearch.getLocalCity(function (status, result) {
    console.log('🚀 ~ status, result:', status, result)
    if (status === 'complete' && result.info === 'OK') {
      if (result && result.city && result.bounds) {
        var cityinfo = result.city
        var citybounds = result.bounds
        document.getElementById('info').innerHTML =
          '您当前所在城市：' + cityinfo
        //地图显示当前城市
        map.setBounds(citybounds)
        console.log('🚀 ~ cityinfo:', cityinfo)
      }
    } else {
      document.getElementById('info').innerHTML = result.info
      console.log('🚀 ~ result.info:', result.info)
    }
  })
}

// 结束编辑
const endEdit = () => {
  polyEditor.close()
  // 监听绘制
  polyEditor.on('end', function (event) {
    console.log('🚀 ~ polyEditor.on ~ event.target:', event.lnglat)
  })
}
</script>

<template>
  <div style="background-color: #fff">
    <div>
      <button class="btn" @click="polyEditor.open()">开始编辑</button>
      <button class="btn" @click="endEdit">结束编辑</button>
      <button class="btn" @click="removePolygon">删除多边形</button>
      <button class="btn" @click="createPolygon">新建</button>
      <button class="btn" @click="savePolygon">保存</button>
      <span id="info"></span>
    </div>
    <div id="container"></div>
  </div>
</template>

<style scoped>
#container {
  margin: 0 auto;
  width: 80%;
  height: 800px;
}
</style>
