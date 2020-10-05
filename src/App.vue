<template lang="pug">
  #app
    textarea(v-if="renamableItemId", ref="nameTextarea", :style="nameTextareaStyle" @keydown.enter="renameSave")
    v-stage(ref="stage"
      :config="configKonva"
      @dragstart="handleDragstart"
      @dragend="handleDragend")
      v-layer(ref="lines")
        //- v-shape(:config=`{
        //-   sceneFunc: function(context, shape) {
        //-     context.beginPath();
        //-     context.moveTo(0, 0);
        //-     context.lineTo(220, 80);
        //-     context.lineTo(80, 200);
        //-     context.closePath();
        //-
        //-     // special Konva.js method
        //-     context.fillStrokeShape(shape);
        //-   },
        //-   fill: 'black',
        //-   stroke: 'black',
        //-   strokeWidth: 1
        //- }`)
        template(v-for="item in list")
          template(v-if="Array.isArray(item.parent_id)")
            v-line(v-for="(v, k) in item.parent_id", :config="getLineConfig(item, k)", :key="k")
          template(v-else-if="typeof item.parent_id === 'number'")
            v-line(:config="getLineConfig(item)")
      v-layer(ref="layer")
        v-group(v-for="item in list"
          :key="item.id"
          :config=`{
            draggable: true,
            id: item.id,
            width: BLOCK_WIDTH,
            height: BLOCK_HEIGHT,
            x: item.x,
            y: item.y,
          }`
          @mouseover="handleMouseover"
          @mouseout="handleMouseout"
          @mouseup="handleMouseup")
          v-rect(:config=`{
            width: BLOCK_WIDTH,
            height: BLOCK_HEIGHT,
            x: 0,
            y: 0,
            innerRadius: 30,
            outerRadius: 50,
            fill: item.fill,
            /* fillPatternImage: image, // WHY IT NOT WORKS? */
            opacity: 1,
            /* scaleX: dragItemId === item.id ? 1.2 : 1,
            scaleY: dragItemId === item.id ? 1.2 : 1, */
            shadowColor: 'black',
            shadowBlur: 10,
            shadowOffsetX: dragItemId === item.id ? 6 : 3,
            shadowOffsetY: dragItemId === item.id ? 6 : 3,
            shadowOpacity: 0.2345
          }`)
          v-image(:config=`{
            image: item.image,
            width: BLOCK_WIDTH,
            height: BLOCK_HEIGHT,
            x: 0,
            y: 0,
          }`)
          v-text(:config=`{
            align: 'center',
            text: item.name,
            y: BLOCK_HEIGHT + 10,
            width: BLOCK_WIDTH
          }`
          @dblclick="rename(item)")
</template>

<script>
  // import Konva from 'konva'
  import dataset from './dataset'
  import folderUrl from './assets/images/folder.png'
  import tableUrl from './assets/images/table.png'

  const width = window.innerWidth
  const height = window.innerHeight
  const BLOCK_HEIGHT = 64
  const BLOCK_WIDTH = 64
  const GRAY_COLOR = '#d9dbdb'
  const FOLDER_COLOR = '#fbd040'
  const TABLE_COLOR = '#03708a'
  const typeColors = {
    folder: FOLDER_COLOR,
    table: TABLE_COLOR
  }
  const typeImages = {
    folder: folderUrl,
    table: tableUrl
  }
  function getTypeColor (type) {
    return typeColors[type] || GRAY_COLOR
  }

  export default {
    data: () => ({
      BLOCK_HEIGHT,
      BLOCK_WIDTH,
      dataset, // debug
      image: null,
      list: [],
      dragItemId: null,
      configKonva: {
        width,
        height
      },
      renamableItemId: null,
      nameTextareaStyle: {
        position: 'absolute',
        border: 'none',
        padding: '0px',
        margin: '0px',
        overflow: 'hidden',
        background: 'none',
        outline: 'none',
        resize: 'none',
        transformOrigin: 'left top',
        top: null,
        left: null,
        width: null,
        height: null,
        fontSize: null,
        lineHeight: null,
        fontFamily: null,
        textAlign: null,
        color: null
      }
    }),
    methods: {
      draw () {
        this.$refs.stage.getNode().draw()
      },
      getLineConfig (item, parentIdKey) {
        let points = this.getLinePoints(item, parentIdKey)
        const bezier = points[0] !== points[2] && points[1] !== points[3]
        if (bezier) {
          let [xStart, yStart, xEnd, yEnd] = points
          let xMiddle = (xStart + xEnd) / 2

          points = [xStart, yStart, xMiddle, yStart, xMiddle, yEnd, xEnd, yEnd]
        }

        return {
          x: 0,
          y: 0,
          points,
          tension: 0,
          bezier,
          closed: !bezier,
          stroke: item === this.list[this.list.length - 1] ? 'green' : 'black'
        }
      },
      getLinePoints (item, parentIdKey) {
        // console.log(`getLinePoints(${parentIdKey})`, item)
        if (!item.parent_id) return null

        const parent_id = Array.isArray(item.parent_id) ? item.parent_id[parentIdKey] : item.parent_id
        const parent = this.list.find(i => i.id === parent_id)
        return [item.x, item.y + (BLOCK_HEIGHT / 2), parent.x + BLOCK_WIDTH, parent.y + (BLOCK_HEIGHT / 2)]
      },
      getTypeImage (type) {
        const imageObj = new Image()
        imageObj.onload = () => {
          this.draw()
          // console.log(`"${type}" - success`)
        }
        imageObj.src = typeImages[type]
        return imageObj
      },
      handleDragstart (e) {
        // save drag element:
        this.dragItemId = e.target.id()
        // move current element to the top:
        const item = this.list.find(i => i.id === this.dragItemId)
        const index = this.list.indexOf(item)
        this.list.splice(index, 1)
        this.list.push(item)
      },
      handleDragend () {
        this.dragItemId = null
      },
      handleMouseover () {
        document.body.style.cursor = 'pointer'
      },
      handleMouseout () {
        document.body.style.cursor = 'default'
      },
      handleMouseup (e) {
        const id = e.currentTarget.attrs.id
        const _lastPos = e.currentTarget._lastPos

        const listItem = this.list.find(i => i.id === id)
        listItem.x = _lastPos.x
        listItem.y =  _lastPos.y
      },
      rename (item) {
        this.renamableItemId = item.id
        // this.nameTextareaStyle.top = areaPosition.y + 'px';
        // this.nameTextareaStyle.left = areaPosition.x + 'px';
        // const textNode =
        // this.nameTextareaStyle.width = textNode.width() - textNode.padding() * 2 + 'px';
        // this.nameTextareaStyle.height = textNode.height() - textNode.padding() * 2 + 5 + 'px';
        // this.nameTextareaStyle.fontSize = textNode.fontSize() + 'px';
        // this.nameTextareaStyle.lineHeight = textNode.lineHeight();
        // this.nameTextareaStyle.fontFamily = textNode.fontFamily();
        // this.nameTextareaStyle.textAlign = textNode.align();
        // this.nameTextareaStyle.color = textNode.fill();
        this.$nextTick(() => {
          this.$refs.nameTextarea.value = item.name
          this.$refs.nameTextarea.focus()
        })
      },
      renameSave (e) {
        this.list.find(item => item.id === this.renamableItemId).name = e.target.value
        this.renamableItemId = null
        e.preventDefault()
        console.log('@renameSave', e)
      }
    },
    mounted () {
      for (let d of dataset) {
        this.list.push({
          ...d,
          x: d.x * width - (BLOCK_WIDTH / 2),
          y: d.y * height - (BLOCK_HEIGHT / 2),
          fill: getTypeColor(d.type),
          image: this.getTypeImage(d.type)
        })
      }
    }
  }
</script>

<style lang="sass">
  body
    margin: 0
    padding: 0
  #app
    font-family: Avenir, Helvetica, Arial, sans-serif
    -webkit-font-smoothing: antialiased
    -moz-osx-font-smoothing: grayscale
    text-align: center
    color: #2c3e50
</style>
