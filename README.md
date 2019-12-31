# zb-hotpot

基于vue和element的运营后台配置图片热区图的组件

前台组件可用map+area完成对接

### hotpot组件使用说明

#### 1. 需要传入的参数

```bigImg: String```

该参数是“新建编辑”要传入编辑图，可传入相对路径和绝对路径

```oldDataList: Object```

该参数是“重新编辑”要传入的数据对象，该对象形如：

<span style="color:red;">一般用returnData拿到的数据对象放回来就好</span>
```javascript
const oldDataList = {
  imgInfo:{
    url: '', //图片地址
    width: 100, //图片宽
    height: 100 //图片高
  },
  // 热区信息
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
  // 用于重新编辑拿数据 保存数据
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
```

#### 2.拿数据

在父组件使用
```html
<!-- html -->
<hotpot ref="hotpot" :bigImg="bigImg" :oldDataList="oldDataList"></hotpot>
```
```javascript
// js
console.log(this.$refs.hotpot.returnData())
```

## Project setup
```
npm install
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
