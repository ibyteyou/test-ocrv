<template lang="pug">
  v-group(
    v-on="$listeners"
    :config=`{
      draggable: !disabled,
      id: item.id,
      width: BLOCK_WIDTH,
      height: BLOCK_HEIGHT,
      x: item.x,
      y: item.y,
      opacity: operations.tieId && (operations.tieId === item.id || tieAlreadyTied.includes(item.id)) ? .25 : 1
    }`
    @contextmenu="handleContextMenu"
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
      opacity: 1,
      shadowColor: 'black',
      shadowBlur: 10,
      shadowOffsetX: operations.dragItemId === item.id ? 6 : 3,
      shadowOffsetY: operations.dragItemId === item.id ? 6 : 3,
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
    @dblclick="rename()")
</template>

<script>
  import { BLOCK_HEIGHT, BLOCK_WIDTH } from '../app'
  import { ModalRenameBus } from './modal-rename'

  export default {
    inheritAttrs: false,
    inject: ['list', 'operations', 'tieAlreadyTied'],
    data: () => ({
      BLOCK_HEIGHT,
      BLOCK_WIDTH
    }),
    props: {
      item: Object,
      disabled: Boolean
    },
    methods: {
      handleMouseover () {
        document.body.style.cursor = 'pointer'
      },
      handleMouseout () {
        document.body.style.cursor = 'default'
      },
      handleContextMenu (e) {
        e.evt.preventDefault()
        this.$emit('context-menu', this.list.find(l => l.id === e.currentTarget.VueComponent.config.id), { top: `${e.evt.clientY}px`, left: `${e.evt.clientX}px` })
      },
      rename () {
        ModalRenameBus.emit('rename', this.item)
      }
    }
  }
</script>

<style lang="sass">
</style>
