<template>
  <div>
    <el-tree :data="menus" node-key="catId" :expand-on-click-node="false" :props="defaultProps" ref="treeMenus"
             @node-click="nodeClick">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ data.name }}</span>
      </span>
    </el-tree>
  </div>
</template>
<script>
export default {
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
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    nodeClick (data, node, co) {
      console.log(data, node, co)
      //   向父组件发送事件
      //   传递的参数：事件名，任意多的东西（当事件触发的时候都会被传递）
      this.$emit('tree-node-event', data, node, co)
    }
  },
  created () {
    this.getMenus()
  }
}
</script>
<style>

</style>
