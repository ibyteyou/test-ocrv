<template lang="pug">
  ul.context-menu(v-if="model", :style="pos")
    li(@click="(rename(), close())") Переименовать
    li(v-if="model.parent_id" @click="untie") Отвязать
    li(@click="$emit('tie', model.id)") Привязать
</template>

<script>
  import { ModalRenameBus } from './modal-rename'

  export default {
    data: () => ({
      model: null,
      pos: null
    }),
    methods: {
      close () {
        this.model = null
        this.pos = null
      },
      rename () {
        ModalRenameBus.emit('rename', this.model)
      },
      set (item, pos) {
        this.model = item
        this.pos = pos
      },
      untie () {
        this.model.parent_id = null
        this.$emit('untie')
        this.$nextTick(() => {
          this.close()
        })
      }
    }
  }
</script>

<style lang="sass">
  ul.context-menu
    position: absolute
    width: 200px
    list-style: none
    margin: 0
    padding: 0
    cursor: pointer
    background-color: #fff
    border: solid 1px #dfdfdf
    li
      display: flex
      justify-content: space-between
      padding: 12px 6px
      border-bottom: solid 1px #dfdfdf
      &:hover
        background-color: #f5f5f5
      &:last-child
        border-bottom: none
</style>
