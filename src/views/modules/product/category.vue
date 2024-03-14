<template>
  <div>
    <el-tree :data="data" @node-click="handleNodeClick"></el-tree>
  </div>
</template>
<script>
export default {
  name: 'category',
  data () {
    return {
      data: [],
      defaultProps: {
        children: 'children',
        label: 'label'
      }
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list'),
        method: 'get'
      }).then(data => {
        console.log('成功获取到菜单数据')
      })
    }
  },
  created () {
    this.getMenus()
  }
}
</script>
<style scoped lang="scss">

</style>
