<template>
  <div ref="img" class="container" :style="{ 'width': imgSize.width + 'px' }">
    <div
      :style="{ fontSize: 0 }"
      @mousedown="mousedown"
      @mousemove="mousemove"
      @mouseup="mouseup"
      @mouseleave="mouseleave"
      @mouseenter="mouseenter"
    >
      <!-- 热区list -->
      <div
        v-for="(item,index) in areaList"
        :key="index"
        class="imgArea"
        :style="styleObject(item)"
      >
        <i class="close el-icon-close" @mousedown.stop="delItem(index)" />
        {{ item.number }}
      </div>
      <img class="banner" :src="img">
    </div>

    <!-- 热区链接配置 -->
    <div class="form">
      <div v-for="(item,index) in areaList" :key="index" class="form-row">
        <div class="form-item">
          <el-tag>热区{{ item.number }}</el-tag>
        </div>
        <div class="form-item">
          <label class="label">名称</label><el-input v-model="item.name" :style="linkInputStyle" />
        </div>
        <div class="form-item">
          <label class="label">链接</label><el-input v-model="item.gotolink" :style="linkInputStyle" />
        </div>
        <i class="el-icon-delete" @click="delItem(index)" />
      </div>
    </div>
    <div :style="{ margin: '10px 0' }">
      <el-button @click="clearList">
        重置
      </el-button>

      <el-button @click="getData">
        getdata
      </el-button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    /* 新建待编辑图片 */
    bigImg: {
      type: String,
      default: ''
    },

    /* 修改待编辑图片、位置、大小等信息 */
    oldDataList: {
      type: Object,
      default: () => {}
    },

    /* 热区对应链接的输入框宽度配置 */
    linkInputStyle: {
      type: Object,
      default: () => {
        return { width: '250px' }
      }
    }
  },
  data() {
    return {
      /* 左键是否处于按着状态 */
      isMouseDown: false,

      /* 存储已完成热区的仓库 */
      areaList: [],

      /* 每个热区对应的编号 */
      imgNum: 1,

      /* 每个img的大小 */
      imgSize: {
        width: null,
        height: null
      },

      /* 目前正在操作的热区 */
      activeArea: {},

      /* 热区背景主图 */
      img: '',

      /* 是否正在对完成的热区重新操作 */
      isResizing: false,

      /* 当前热区的初始点 */
      mounseDownPoint: {},

      /* 背景主图对offsetParent的toLeft */
      imgOffsetLeft: 0,

      /* 背景主图对offsetParent的toTop */
      imgOffsetTop: 0,

      /* 鼠标是否移出背景图 */
      isOut: false
    }
  },
  /* watch: {
    bigImg() {
      this.initImg(this.bigImg)
    },
    oldDataList() {
      this.areaList = this.oldDataList.oldDataList
      this.imgNum = this.areaList[this.areaList.length - 1].number + 1
    }
  }, */
  mounted() {
    /* 拿已有的热区数据 */
    if (this.oldDataList && this.oldDataList.oldDataList) {
      this.areaList = this.oldDataList.oldDataList
      this.imgNum = this.areaList[this.areaList.length - 1].number + 1
      this.initImg(this.oldDataList.imgInfo.url)
    } else {
      this.initImg(this.bigImg)
    }
  },
  methods: {
    /* 收集img信息 */
    initImg(src) {
      const img = new Image()
      img.src = src
      img.onload = () => {
        this.img = src

        /* 拿到本热区组件背景图的大小信息 */
        this.imgSize.width = img.width
        this.imgSize.height = img.height

        /* 拿到本热区组件的offset信息 */
        this.imgOffsetTop = this.$refs.img.offsetTop
        this.imgOffsetLeft = this.$refs.img.offsetLeft
      }
    },

    /* 鼠标点击的时候 创建热区事件 */
    mousedown(e) {
      e.preventDefault()
      this.isMouseDown = true

      /* 初始化新建热区信息 */
      const newArea = {
        x1: e.offsetX,
        y1: e.offsetY,
        x2: e.offsetX,
        y2: e.offsetY,
        number: this.imgNum,
        gotolink: '',
        name: '',
        id: new Date().getTime() // 自己定义hash
      }

      /* 拿到本热区组件的offset信息 */
      this.imgOffsetTop = this.$refs.img.offsetTop
      this.imgOffsetLeft = this.$refs.img.offsetLeft

      /* 新建热区和编辑热区 区分 */
      if (e.target.tagName === 'DIV') {
        this.activeArea = this.areaList[e.target.innerText - 1]
        this.isResizing = true
      } else {
        this.imgNum++

        /* 热区可视化数组 */
        this.areaList.push(newArea)

        /* 标记当前操作热区 */
        this.activeArea = newArea

        /* 标记点击坐标 */
        this.mounseDownPoint = {
          x: e.offsetX,
          y: e.offsetY
        }
      }
    },

    /* 鼠标手指抬起 数据初始化迎接下个热区 */
    mouseup() {
      this.isMouseDown = false
      this.isResizing = false
      this.mounseDownPoint = {}
      this.isOut = false
      this.activeArea = {}

      /* 如果点击后没位移 判定为手误 直接删除 */
      this.areaList.forEach((i, index) => {
        if (i.y2 === i.y1 || i.x1 === i.x2) {
          this.delItem(index)
        }
      })
    },

    /* 鼠标点击移动 热区可视化core */
    mousemove(e) {
      if (this.isMouseDown && !this.isResizing) {
        this.isOut = true
        /* 坐标体系为 (x1, y1)左上 (x2, y2)右下 */
        this.activeArea.x1 = Math.min(this.mounseDownPoint.x, e.pageX - this.imgOffsetLeft)
        this.activeArea.y1 = Math.min(this.mounseDownPoint.y, e.pageY - this.imgOffsetTop)
        this.activeArea.x2 = Math.max(this.mounseDownPoint.x, e.pageX - this.imgOffsetLeft)
        this.activeArea.y2 = Math.max(this.mounseDownPoint.y, e.pageY - this.imgOffsetTop)
      }

      /* 编辑热区情况 (目前只兼容向第四象限扩大缩小) */
      if (this.isResizing && this.activeArea) {
        this.isOut = true
        this.activeArea.x2 = e.pageX - this.imgOffsetLeft
        this.activeArea.y2 = e.pageY - this.imgOffsetTop
      }
    },

    /* 处理鼠标移出背景图情况 直接定格 */
    mouseleave() {
      if (this.isMouseDown) {
        this.activeArea = {}
        this.isMouseDown = false
        this.mounseDownPoint = {}
      }
    },

    /* 处理鼠标移出背景图后 上个变为不可点击状态的情况 */
    mouseenter() {
      this.isOut = false
    },

    /* 清除area */
    clearList() {
      this.areaList = []
      this.imgNum = 1
      this.activeArea = {}
    },

    /* 计算每个热区的位置大小信息 */
    styleObject(item) {
      const width = (item.x2 - item.x1)
      const height = (item.y2 - item.y1)
      const top = item.y1
      const left = item.x1
      return {
        'width': width + 'px',
        'height': height + 'px',
        'top': top + 'px',
        'left': left + 'px',
        'pointer-events': this.isOut ? 'none' : 'auto'
      }
    },

    /* 删除某个热区 */
    delItem(index) {
      this.areaList.splice(index, 1)

      /* 删除后 每个热区按顺序重新编号 */
      if (this.areaList) {
        const arr = this.areaList.filter((i) => i.number > index)
        if (!arr) return
        arr.forEach((i) => i.number--)
        if (this.areaList[this.areaList.length - 1]) {
          this.imgNum = this.areaList[this.areaList.length - 1].number + 1
        } else {
          this.imgNum = 1
        }
      }
    },

    getData() {
      console.log(this.returnData())
    },

    /* 返回热区所有信息 imgInfo\oldDataList是记录后台热区数据 positionList是提供前台map数据 */
    returnData() {
      const positionList = this.areaList.map((i) => {
        return {
          number: i.number,
          x: i.x1,
          y: i.y1,
          width: i.x2 - i.x1,
          height: i.y2 - i.y1,
          url: i.gotolink,
          name: i.name,
          id: i.id
        }
      })
      const obj = {
        imgInfo: {
          url: this.bigImg,
          width: this.imgSize.width,
          height: this.imgSize.height
        },
        positionList,
        oldDataList: this.areaList
      }
      return obj
    }
  }
}
</script>

<style lang="scss" scoped>
.flex {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top:10px;
}
.container {
  position:relative;
}
.banner {
  width:100%;
}
.imgArea {
  border:white dashed 1px;
  position:absolute;
  color: white;
  background: rgba($color: pink, $alpha: 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 38px;
  &::after{
      display:block;
      content:'';
      position:absolute;
      width: 0;
      height: 0;
      border-style: solid;
      border-width: 0 0 8px 8px;
      border-color: transparent transparent #f2f6fa transparent;
      right: 0;
      bottom: 0;
      cursor: se-resize;
  }
  .close{
      font-size: 10px;
      position:absolute;
      right:2px;
      top:0;
      width: 8px;
      height:8px;
      color:white;
  }
}
.form {
  font-size:12px;
  width: 100%;
  .form-row {
    display: flex;
    margin: 8px 0;
    align-items: center;
    .form-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      white-space: nowrap;
      margin: 0 5px;
      .label {
          margin-right: 10px;
      }
    }
  }
}
.el-icon-delete{
  cursor: pointer;
  &:hover {
      color:royalblue;
  }
}
</style>