<template>
  <div>
    <el-button :type="action.type" v-for="action in actions" @click="action.handler" :icon="action.icon">{{action.name}}</el-button>
  </div>
</template>
<script>
  export default {
    props: {
      actions: [Array]
    }
  }
</script>
