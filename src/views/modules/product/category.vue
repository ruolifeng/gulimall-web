<template>
  <div>
    <el-tree :data="menus" node-key="catId" show-checkbox :props="defaultProps" :expand-on-click-node="false" :default-expanded-keys="openKeys">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ data.name }}</span>
        <span>
          <el-button v-if="node.childNodes.length !== 0" type="text" size="mini" @click="() => append(data)">
            添加
          </el-button>
          <el-button v-if="node.childNodes.length === 0" type="text" size="mini" @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>
<script>
export default {
  name: 'category',
  data () {
    return {
      openKeys: [],
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
    append (data) {
      console.log(data)
    },
    remove (node, data) {
      this.$confirm(`此操作将永久删除【${node.data.name}】, 是否继续?', '提示`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const ids = [node.data.catId]
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          // 刷新出新的菜单
          this.getMenus()
          this.openKeys = [node.parent.data.catId]
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
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
