<template lang="pug">
  #app
    v-stage(ref="stage"
      :config="configKonva"
      @dragstart="handleDragstart"
      @dragend="handleDragend")
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
          @mouseout="handleMouseout")
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
          }`)
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
  function getTypeImage (type) {
    const imageObj = new Image()
    imageObj.onload = function () {
      console.log(`"${type}" - success`)
    }
    imageObj.src = typeImages[type]
    return imageObj
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
        width: width,
        height: height
      }
    }),
    methods: {
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
      }
    },
    mounted () {
      for (let d of dataset) {
        this.list.push({
          ...d,
          x: d.x * width - (BLOCK_WIDTH / 2),
          y: d.y * height - (BLOCK_HEIGHT / 2),
          fill: getTypeColor(d.type),
          image: getTypeImage(d.type)
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
