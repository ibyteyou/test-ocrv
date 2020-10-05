<template lang="pug">
  div.modal-rename(v-if="model")
    textarea(ref="nameTextarea" @keydown.enter="save")
</template>

<script>
  import Emitter from 'component-emitter'
  export const ModalRenameBus = new Emitter()

  export default {
    data: () => ({
      model: null
    }),
    watch: {
      model (model) {
        this.$emit('input', model)
      }
    },
    methods: {
      rename (item) {
        this.model = item
        this.$nextTick(() => {
          this.$refs.nameTextarea.value = item.name
          this.$refs.nameTextarea.focus()
        })
      },
      save (e) {
        this.model.name = e.target.value
        this.model = null
        e.preventDefault()
      }
    },
    mounted () {
      ModalRenameBus.on('rename', this.rename)
    },
    beforeDestroy () {
      ModalRenameBus.off('rename', this.rename)
    }
  }
</script>
