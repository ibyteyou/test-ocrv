<template lang="pug">
  #app(@keyup.esc="handleEscKeyup")
    modal-rename(ref="modalRename" @input="renamedItemId = $event")
    v-stage(:class="{ 'filter-hidden': renamedItemId }"
      ref="stage"
      :config="configKonva"
      @click="handleClick"
      @dragstart="handleDragstart"
      @dragend="handleDragend")
      v-layer(ref="lines")
        v-line(v-for="line in lines", :config="line", :key="`line--${line.id}_${line.parent_id}`")
        v-shape(v-for="tie in ties", :config="tie", :key="tie.key")
      v-layer(ref="layer")
        el-icon(v-for="item in list"
          :disabled="disabled"
          :key="item.id"
          :item="item"
          @context-menu="$refs.contextMenu.set"
          @mouseup="handleMouseup")
    context-menu(ref="contextMenu" @tie="tie" @untie="untie")
</template>

<script>
  import dataset from './dataset'
  import folderUrl from './assets/images/folder.png'
  import tableUrl from './assets/images/table.png'
  import contextMenu from './components/context-menu'
  import elIcon from './components/el-icon'
  import modalRename from './components/modal-rename'

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

  export {
    BLOCK_HEIGHT,
    BLOCK_WIDTH
  }

  export default {
    components: { contextMenu, elIcon, modalRename },
    provide () {
      return {
        list: this.list,
        operations: this.operations,
        tieAlreadyTied: this.tieAlreadyTied
      }
    },
    data: () => ({
      BLOCK_HEIGHT,
      BLOCK_WIDTH,
      dataset, // debug
      image: null,
      lastId: null,
      list: [],
      configKonva: {
        width,
        height
      },
      lines: [],
      operations: {
        dragItemId: null,
        tieId: null
      },
      renamedItemId: null,
      tieAlreadyTied: [],
      ties: []
    }),
    computed: {
      disabled () {
        return !!this.renamedItemId || !!this.operations.tieId
      }
    },
    methods: {
      calcLines () {
        const lines = []

        for (let item of this.list) {
          if (Array.isArray(item.parent_id)) {
            for (let parIdKey in item.parent_id) {
              lines.push(this.getLineConfig(item, parIdKey))
            }
          } else if (typeof item.parent_id === 'number') {
            lines.push(this.getLineConfig(item))
          }
        }

        this.lines = lines
      },
      calcTies () {
        let ties = []
        for (let line of this.lines) {
          ties = ties.concat(this.lines.filter(l => (line.id !== l.id && line.parent_id === l.parent_id)
            || (line.id === l.id && line.parent_id !== l.parent_id)))
        }
        ties = ties.filter((value, index, self) => self.indexOf(value) === index) // unique filter
        this.ties = ties.map(t => {
          const midX = (t.points[0] + t.points[t.bezier ? 6 : 2]) / 2
          const midY = (t.points[1] + t.points[t.bezier ? 7 : 3]) / 2
          let dx = t.points[t.bezier ? 6 : 2] - t.points[t.bezier ? 2 : 0]
          let dy = t.points[t.bezier ? 7 : 3] - t.points[t.bezier ? 3 : 1]
          let angle = Math.atan2(dy, dx) * 180 / Math.PI

          return {
            sceneFunc (context, shape) {
              context.beginPath()
              context.moveTo(0, 0)
              context.lineTo(12, 4)
              context.lineTo(12, -4)
              context.closePath()
              context.fillStrokeShape(shape)
            },
            fill: 'black',
            stroke: 'black',
            strokeWidth: 1,
            x: midX,
            y: midY,
            rotation: angle,
            key: `tie--${t.id}_${t.parent_id}`
          }
        })
      },
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
          id: item.id,
          parent_id: parentIdKey ? item.parent_id[parentIdKey] : item.parent_id,
          x: 0,
          y: 0,
          points,
          tension: 0,
          bezier,
          closed: !bezier,
          stroke: item.id === this.lastId ? '#64915a' : 'black'
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
        this.operations.dragItemId = e.target.id()
        this.__clonedDragEl = e.target.clone()
        this.__clonedDragEl.setOpacity(.25)
        this.$refs.layer.getNode().add(this.__clonedDragEl)
        // move current element to the top:
        // const item = this.list.find(i => i.id === this.operations.dragItemId)
        // const index = this.list.indexOf(item)
        // this.list.splice(index, 1)
        // this.list.push(item)
      },
      handleDragend () {
        this.operations.dragItemId = null
        this.__clonedDragEl.remove()
        this.__clonedDragEl = null

        this.calcLines()
        this.calcTies()
      },
      handleEscKeyup () {
        if (!this.$refs.contextMenu.model && !this.renamedItemId) return

        if (this.$refs.contextMenu.model) {
          this.$refs.contextMenu.model = null
        } else if (this.renamedItemId) {
          this.renamedItemId = null
        }
      },
      handleClick () {
        if (!this.$refs.contextMenu.model && !this.renamedItemId) return

        if (this.$refs.contextMenu.model) {
          this.$refs.contextMenu.model = null
        } else if (this.renamedItemId) {
          this.renamedItemId = null
        }
      },
      handleMouseup (e) {
        const id = e.currentTarget.attrs.id

        if (this.operations.tieId) {
          const item = this.list.find(l => l.id === id)
          if (Array.isArray(item.parent_id)) {
            if (item.parent_id.includes(this.operations.tieId)) return // already tied?
            item.parent_id.push(this.operations.tieId)
          } else if (typeof item.parent_id === 'number') {
            if (item.parent_id === this.operations.tieId) return // already tied!
            item.parent_id = [item.parent_id, this.operations.tieId]
          } else {
            item.parent_id = this.operations.tieId
          }
          // this.__tieNode.setOpacity(1)
          this.draw()
          this.operations.tieId = null
          this.tieAlreadyTied.splice(0, this.tieAlreadyTied.length)
          // this.__tieNode = null
        } else {
          const _lastPos = e.currentTarget._lastPos

          if (!_lastPos) return

          const listItem = this.list.find(i => i.id === id)
          listItem.x = _lastPos.x
          listItem.y =  _lastPos.y
        }

        this.calcLines()
        this.calcTies()
      },
      tie (tieId) {
        this.operations.tieId = tieId
        this.list
          .filter(l => Array.isArray(l.parent_id) ? l.parent_id.includes(this.operations.tieId) : l.parent_id === this.operations.tieId)
          .map(l => l.id)
          .forEach(item => this.tieAlreadyTied.push(item))

        this.draw()
        this.$refs.contextMenu.close()
      },
      untie () {
        this.calcLines()
        this.calcTies()
      }
    },
    mounted () {
      let lastId = 0
      for (let d of dataset) {
        if (d.id > lastId) {
          lastId = d.id
        }
        this.list.push({
          ...d,
          x: d.x * width - (BLOCK_WIDTH / 2),
          y: d.y * height - (BLOCK_HEIGHT / 2),
          fill: getTypeColor(d.type),
          image: this.getTypeImage(d.type)
        })
      }
      this.lastId = lastId

      this.calcLines()
      this.calcTies()
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
  .filter-hidden
    opacity: 0.3456789
    background-color: rgba(0, 0, 0, 0.3456789)
    filter: brightness(0.654321)
  .modal-rename
    position: absolute
    width: 260px
    height: 64px
    background-color: #fff
    z-index: 1
    left: 0
    right: 0
    margin: auto
    textarea
      font-size: 32px
      border: none
      outline: none
      resize: none
</style>
