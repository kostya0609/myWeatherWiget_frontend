<template>
  <div
      class="mww-gearCard-cityInfo"
  >
    <el-button
        text
        @mouseover="context.emit('enableDrag')"
        @mouseleave="context.emit('disableDrag')"
    >
      <el-icon :size="20"><Expand /></el-icon>
    </el-button>

    <label class="mww-gearCard-cityInfo-label">
      {{city.localName ? city.localName : city.name }}, {{city.country}}
    </label>

    <el-button
        text
        @click="context.emit('deleteCity', city)"
    >
      <el-icon :size="20"><Delete /></el-icon>
    </el-button>
  </div>
</template>

<script>
export default {
  name : "gearCity",
  props : ['city'],
  emits : ['deleteCity', 'enableDrag', 'disableDrag'],
  setup(props, context){
    return{context}
  },
}
</script>

<style scoped>

</style>