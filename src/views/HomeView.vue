<template>
  <div class="home">
    <div id="chinaMap" class="map-container"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import 'echarts-gl'

var option
export default {
  name: "HomeView",
  mounted() {
    this.initChinaMap()
  },
  methods: {
    async initChinaMap() {
      const chartDom = document.getElementById('chinaMap')
      const myChart = echarts.init(chartDom)

      // 获取中国地图 GeoJSON 数据
      const response = await fetch('https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json')
      const chinaJson = await response.json()
      echarts.registerMap('china', chinaJson)

      // 各省模拟数据
      const provinceData = [
        { name: '北京', value: [116.46, 39.92, 580] },
        { name: '上海', value: [121.48, 31.22, 650] },
        { name: '广州', value: [113.23, 23.16, 520] },
        { name: '深圳', value: [114.07, 22.62, 480] },
        { name: '成都', value: [104.06, 30.67, 380] },
        { name: '杭州', value: [120.19, 30.26, 420] },
        { name: '武汉', value: [114.31, 30.52, 350] },
        { name: '西安', value: [108.95, 34.27, 320] },
        { name: '重庆', value: [106.55, 29.56, 360] },
        { name: '南京', value: [118.78, 32.04, 400] },
        { name: '天津', value: [117.2, 39.13, 390] },
        { name: '苏州', value: [120.62, 31.3, 450] },
        { name: '郑州', value: [113.65, 34.76, 280] },
        { name: '长沙', value: [113, 28.21, 300] },
        { name: '沈阳', value: [123.43, 41.8, 250] },
        { name: '青岛', value: [120.38, 36.07, 350] },
        { name: '济南', value: [116.98, 36.67, 280] },
        { name: '大连', value: [121.62, 38.91, 320] },
        { name: '厦门', value: [118.1, 24.46, 280] },
        { name: '昆明', value: [102.71, 25.04, 260] },
        { name: '哈尔滨', value: [126.63, 45.75, 240] },
        { name: '长春', value: [125.35, 43.88, 220] },
        { name: '福州', value: [119.3, 26.08, 260] },
        { name: '石家庄', value: [114.48, 38.03, 280] },
        { name: '太原', value: [112.53, 37.87, 250] },
        { name: '兰州', value: [103.73, 36.03, 180] },
        { name: '乌鲁木齐', value: [87.68, 43.77, 160] },
        { name: '呼和浩特', value: [111.65, 40.82, 200] },
        { name: '南宁', value: [108.33, 22.84, 220] },
        { name: '贵阳', value: [106.71, 26.57, 200] },
        { name: '海口', value: [110.35, 20.02, 150] },
        { name: '拉萨', value: [91.11, 29.97, 80] },
        { name: '西宁', value: [101.74, 36.56, 120] },
        { name: '银川', value: [106.27, 38.47, 140] },
        { name: '乌鲁木齐', value: [87.68, 43.77, 160] }
      ]

      const option = {
        tooltip: {
          trigger: 'item',
          formatter: (params) => {
            if (params.value && params.value[2]) {
              return `${params.name}: ${params.value[2]}`
            }
            return params.name
          }
        },
        geo3D: {
          map: 'china',
          roam: true,
          itemStyle: {
            color: '#1d4e89',
            opacity: 0.95,
            borderWidth: 0.5,
            borderColor: '#4db8ff'
          },
          light: {
            main: {
              intensity: 1.2,
              shadow: true,
              shadowQuality: 'high'
            },
            ambient: {
              intensity: 0.4
            }
          },
          viewControl: {
            autoRotate: false,
            distance: 100,
            alpha: 40,
            beta: 20,
            center: [0, 0, 0]
          },
          label: {
            show: true,
            formatter: '{b}',
            textStyle: {
              color: '#fff',
              fontSize: 10
            }
          },
          emphasis: {
            itemStyle: {
              color: '#ffd700'
            },
            label: {
              show: true,
              textStyle: {
                color: '#fff',
                fontSize: 14
              }
            }
          },
          shading: 'realistic',
          realisticMaterial: {
            roughness: 0.6,
            metalness: 0.1
          }
        },
        series: [
          {
            type: 'bar3D',
            coordinateSystem: 'geo3D',
            data: provinceData,
            shading: 'realistic',
            barSize: 1,
            silent: true,
            itemStyle: {
              color: '#ff6b6b',
              opacity: 0.9
            },
            emphasis: {
              itemStyle: {
                color: '#ffd700'
              }
            }
          }
        ]
      }

      myChart.setOption(option)
    }
  }
}
</script>

<style scoped>
.home {
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
  background: #0a0e27;
}

.map-container {
  width: 100%;
  height: 100%;
}
</style>
