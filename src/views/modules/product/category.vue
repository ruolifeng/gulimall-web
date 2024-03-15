<template>
  <div>
    <el-tree :data="menus" node-key="catId" show-checkbox :props="defaultProps" :expand-on-click-node="false"
             :default-expanded-keys="openKeys">
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
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="活动区域">

        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'category',
  data () {
    return {
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: 0,
        productCount: 0
      },
      dialogVisible: false,
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
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
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
    },
    addCategory () {
      this.$confirm(`此操作将添加记录【${this.category.name}】, 是否继续?', '提示`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const data = this.category
        this.$message({
          type: 'success',
          message: '添加成功!'
        })
        this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(data, false)
        }).then(({data}) => {
          this.dialogVisible = false
          // 刷新出新的菜单
          this.getMenus()
          this.openKeys = [this.category.parentCid]
          this.category = {
            name: '',
            parentCid: 0,
            catLevel: 0,
            showStatus: 1,
            sort: 0,
            icon: '',
            productUnit: 0,
            productCount: 0
          }
        })
      }).catch(() => {
        this.category = {
          name: '',
          parentCid: 0,
          catLevel: 0,
          showStatus: 1,
          sort: 0,
          icon: '',
          productUnit: 0,
          productCount: 0
        }
        this.$message({
          type: 'info',
          message: '已取消添加'
        })
        this.dialogVisible = false
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
