<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
  </div>
</template>
<script>
export default {
  name: 'category',
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
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
      }).then(({data}) => {
        console.log(data.data)
        this.menus = data.data
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
