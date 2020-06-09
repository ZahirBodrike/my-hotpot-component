<template>
  <div id="app">
    <div>
      <h5 @click="change">热区</h5>
      <hotpot ref="hotpot" :bigImg="bigImg" :oldDataList="old"></hotpot>
      <hotpot ref="hotpot" :bigImg="smallImg"></hotpot>
      <hotpot ref="hotpot" :bigImg="bigImg"></hotpot>
    </div>
  </div>
</template>

<script>
import hotpot from './components/hotpot.vue'

export default {
  name: 'app',
  components: {
    hotpot
  },
  methods: {
    change() {
      this.bigImg = 'https://img.alicdn.com/tfs/TB11cdpoUH1gK0jSZSyXXXtlpXa-520-280.jpg_q90_.webp'
    }
  },
  data() {
    return {
      testlist: 1,
      bigImg: 'https://res.stroage.zuzuche.com/group1/M00/04/08/wKgKRFw8AcCAG3pIAAJgDWrl1TY29.jpeg',
      smallImg: require('./assets/banner.jpg'),
      old: {
        imgInfo: {
          // url: 'https://img.alicdn.com/tfs/TB11cdpoUH1gK0jSZSyXXXtlpXa-520-280.jpg_q90_.webp',
          url: require('./assets/banner.jpg'),
          width: 1000,
          height: 500
        },
        positionList:[
          {
          number: 1,
          x: 100,
          y: 100,
          width: 100,
          height: 100
          },
          {
            number: 2,
            x:50,
            y:50,
            width: 50,
            height: 50
          }
        ],
        oldDataList: [
          {
            number:1,
            x1:100,
            y1:100,
            x2:200,
            y2:200,
            gotolink:'www.baidu.com',
            name:'faker',
            id: Math.floor(new Date().getTime() * Math.random())
          },
          {
            number:2,
            x1: 50,
            y1: 50,
            x2: 100,
            y2: 100,
            gotolink: '123',
            name: '456',
            id: Math.floor(new Date().getTime() * Math.random())
          }
        ]
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0 auto;
}
</style>
