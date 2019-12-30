<template>
  <div ref="img" class="container" :style="{'width':imgSize.width+'px'}">
    <div
      @mousedown="mousedown"
      @mousemove="mousemove"
      @mouseup="mouseup"
      @mouseleave="mouseleave"
      @mouseenter="mouseenter"
      style="font-size:0;">
      <div
        v-for="(item,index) in areaList"
        :key="index"
        class="imgArea"
        :style="styleObject(item)">
          <i class="close el-icon-close" @mousedown.stop="delItem(index)"></i>
          {{ item.number }}
      </div>
      <img class="banner" :src="img"/>
    </div>
    <div class="form">
      <el-row class="flex" v-for="(item,index) in areaList" :key="index">
        <el-col :span="3" class="form-item"><el-tag>热区{{item.number}}</el-tag></el-col>
        <el-col :span="10" class="form-item">
          <label class="label">名称</label><el-input v-model="item.name"></el-input>
        </el-col>
        <el-col :span="10" class="form-item">
          <label class="label">链接</label><el-input v-model="item.gotolink"></el-input>
        </el-col>
        <el-col :span="1"><i class="el-icon-delete" @click="delItem(index)"></i></el-col>
      </el-row>
    </div>
    <div>
      <el-button @click="clearList">重置</el-button>
    </div>
  </div>
</template>

<script>
export default {
  props:{
    // new pic
    bigImg: String,
    // old data : pic, position, size
    oldDataList: Object
  },
  data() {
    return {
      // during mouse-down and mouse-up
      isMouseDown: false,
      // area array
      areaList:[],
      // num on area
      imgNum: 1,
      // img : width, height
      imgSize: {
        width: null,
        height: null
      },
      // area that is using
      activeArea: {},
      // <img> src
      img: '',
      // status : resize
      isResizing: false,
      // create area, the pos of mouse-down
      mounseDownPoint: {},
      // target img to document border left
      imgOffsetLeft: null,
      // target img to document border top
      imgOffsetTop: null,
      // mouse is out of img ?
      isOut: false,
    }
  },
  methods: {
    initImg(src) {
      const self = this
      let img = new Image();
      img.onload = function() {
        self.imgSize.width = img.width
        self.imgSize.height = img.height
      }
      img.src = src;
      this.img = img.src
    },
    mousedown(e) {
      e.preventDefault()
      this.isMouseDown = true;
      this.imgOffsetTop = this.$refs.img.offsetTop;
      this.imgOffsetLeft = this.$refs.img.offsetLeft;
      const hash = new Date().getTime() * Math.random()
      // 点下去的初始坐标
      let newArea = {
        x1 : e.offsetX,
        y1 : e.offsetY,
        x2 : e.offsetX,
        y2 : e.offsetY,
        number: this.imgNum,
        gotolink: '',
        name: '',
        id: Math.floor(hash)
      }
      // start resize
      if(e.target.tagName == 'DIV') {
        this.activeArea = this.areaList[e.target.innerText-1]
        this.isResizing = true
      } else {
        // new area
        this.imgNum++;
        this.areaList.push(newArea);
        this.activeArea = newArea;
        this.mounseDownPoint = {
          x: e.offsetX,
          y: e.offsetY
        }
      }
    },
    mouseup() {
      // 结尾坐标
      this.isMouseDown = false;
      this.isResizing = false
      this.mounseDownPoint = {};
      this.isOut = false
      this.activeArea = {}
      this.areaList.forEach((i, index) => {
        if(i.y2 == i.y1 || i.x1 == i.x2) {
          this.delItem(index)
        }
      })
    },
    mousemove(e) {
      if (this.isMouseDown && !this.isResizing) {
        this.isOut = true;
        this.activeArea.x1 = Math.min(this.mounseDownPoint.x, e.pageX-this.imgOffsetLeft);
        this.activeArea.y1 = Math.min(this.mounseDownPoint.y, e.pageY-this.imgOffsetTop);
        this.activeArea.x2 = Math.max(this.mounseDownPoint.x, e.pageX-this.imgOffsetLeft);
        this.activeArea.y2 = Math.max(this.mounseDownPoint.y, e.pageY-this.imgOffsetTop);
      }
      if (this.isResizing) {
        if (this.activeArea) {
          this.isOut = true;
          this.activeArea.x2 = e.pageX - this.imgOffsetLeft;
          this.activeArea.y2 = e.pageY - this.imgOffsetTop;
        }
      }
    },
    mouseleave() {
      if (this.isMouseDown) {
        this.activeArea = {}
        this.isMouseDown = false;
        this.mounseDownPoint = {};
      }
    },
    mouseenter() {
      this.isOut = false
    },
    clearList() {
      this.areaList = [];
      this.imgNum = 1;
      this.activeArea = {}
    },
    styleObject(item) {
      let width = (item.x2 - item.x1);
      let height = (item.y2 - item.y1);
      let top = item.y1;
      let left = item.x1;
      return {
        'width': width + 'px',
        'height': height + 'px',
        'top': top + 'px',
        'left': left + 'px',
        'pointer-events': this.isOut ? 'none' : 'auto'
      }
    },
    delItem(index) {
      this.areaList.splice(index,1)
      if(this.areaList) {
        const arr = this.areaList.filter(i => i.number > index)
        if(!arr) return;
        arr.forEach(i => i.number--)
        if(this.areaList[this.areaList.length-1]) {
          this.imgNum = this.areaList[this.areaList.length-1].number + 1
        } else {
          this.imgNum = 1
        }
      }
    },
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
      return obj;
    }
  },
  mounted() {
    if(this.oldDataList) {
      this.areaList = this.oldDataList.oldDataList
      this.imgNum = this.areaList[this.areaList.length-1].number+1;
      this.initImg(this.oldDataList.imgInfo.url)
    } else {
      this.initImg(this.bigImg)
    }
  },
  watch: {
    bigImg() {
      this.initImg(this.bigImg)
    },
    oldDataList() {
      this.areaList = this.oldDataList.oldDataList
      this.imgNum = this.areaList[this.areaList.length-1].number+1;
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
    right:4px;
    top:0;
    width: 8px;
    height:8px;
    color:white;
  }
}
.form {
  margin: 20px auto;
  font-size:12px;
  width: 100%;
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
.el-icon-delete{
  cursor: pointer;
  &:hover {
    color:royalblue;
  }
}
</style>