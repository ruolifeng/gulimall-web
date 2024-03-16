<template>
  <div>
    <el-switch v-model="isDrag" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="isDrag" type="primary" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree :data="menus" node-key="catId" show-checkbox :props="defaultProps" :expand-on-click-node="false"
             :default-expanded-keys="openKeys" :draggable="isDrag" :allow-drop="allowDrop" @node-drop="handleDrop" ref="treeMenus">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ data.name }}</span>
        <span>
          <el-button v-if="node.childNodes.length !== 0" type="text" size="mini" @click="() => append(data)">
            添加
          </el-button>
          <el-button v-if="node.childNodes.length === 0" type="text" size="mini" @click="() => remove(node, data)">
            删除
          </el-button>
          <el-button type="text" size="mini" @click="() => changeMenu(node, data)">
            修改
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="emptyCategoryInfo">取 消</el-button>
    <el-button type="primary" @click="optionUpdate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import {handle} from 'nightwatch/lib/runner/cli/errorhandler'

export default {
  name: 'category',
  data () {
    return {
      pCid: 0,
      isDrag: false,
      updateSortNods: [],
      dialogTitle: '',
      dialogType: '',
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: '',
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
    handle,
    emptyCategoryInfo () {
      this.pCid = 0
      this.dialogVisible = false
      this.updateSortNods = []
      this.category = {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: '',
        productCount: 0
      }
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    append (data) {
      this.dialogTitle = '添加分类'
      this.dialogType = 'append'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
    },
    changeMenu (node, data) {
      this.dialogTitle = '修改分类'
      this.dialogType = 'update'
      this.dialogVisible = true
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        this.category.name = data.data.name
        this.category.parentCid = data.data.parentCid
        this.category.catId = data.data.catId
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
      })
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
    optionUpdate () {
      if (this.dialogType === 'update') {
        this.updateCategory('update')
      } else {
        this.updateCategory('save')
      }
    },
    updateCategory (urlPam) {
      this.$confirm(`此操作将${urlPam === 'update' ? '修改' : '添加'}记录【${this.category.name}】, 是否继续?', '提示`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const data = this.category
        console.log(data)
        this.$message({
          type: 'success',
          message: `${urlPam === 'update' ? '修改' : '添加'}成功!`
        })
        this.$http({
          url: this.$http.adornUrl(`/product/category/${urlPam}`),
          method: 'post',
          data: this.$http.adornData(data, false)
        }).then(({data}) => {
          this.dialogVisible = false
          // 刷新出新的菜单
          this.getMenus()
          this.openKeys = [this.category.parentCid]
          this.emptyCategoryInfo()
        })
      }).catch(() => {
        this.emptyCategoryInfo()
        this.$message({
          type: 'info',
          message: `已取消${urlPam === 'edit' ? '修改' : '添加'}`
        })
        this.dialogVisible = false
      })
    },
    allowDrop (draggingNode, dropNode, type) {
      const countNewLevel = this.getNewLevel(draggingNode, dropNode, type)
      return countNewLevel <= 3
    },
    getNewLevel (draggingNode, dropNode, type) {
      if (type === 'inner') {
        if (draggingNode.parent.data.catId === dropNode.parent.data.catId) return this.getChildLevel(draggingNode) * 1 + 1
        return (this.getChildLevel(draggingNode) + this.getChildLevel(dropNode)) * 1 + 1
      }
      if (type === 'next') {
        if (draggingNode.parent.data.catId === dropNode.parent.data.catId) return this.getChildLevel(draggingNode)
        return this.getChildLevel(draggingNode) + this.getChildLevel(dropNode)
      }
      if (type === 'prev') {
        if (draggingNode.parent.data.catId === dropNode.parent.data.catId) return this.getChildLevel(draggingNode)
        return this.getChildLevel(draggingNode) + this.getChildLevel(dropNode)
      }
    },
    getChildLevel (node) {
      if (node.childNodes.length === 0) {
        return node.level // 如果没有子节点，返回当前节点的层级
      } else {
        return this.getChildLevel(node.childNodes[0]) // 递归调用获取子节点的层级
      }
    },
    handleDrop (draggingNode, dropNode, dropType, ev) {
      console.log(draggingNode, dropNode, dropType, ev)
      // 当前节点的最新父节点的id
      let siblings = []
      if (dropType === 'before' || dropType === 'after') {
        this.pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
      } else {
        this.pCid = dropNode.data.catId
      }
      // 当前拖拽节点的最新顺序
      if (dropType === 'inner') {
        siblings = dropNode.childNodes
      } else {
        siblings = dropNode.parent.childNodes
      }
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          // 如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level
          if (siblings[i].level === draggingNode.level) {
            // 当前节点层级发生变化
            catLevel = siblings[i].level
            // 修改子节点的层级
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateSortNods.push({catId: siblings[i].data.catId, sort: i, parentCid: this.pCid, catLevel: catLevel})
        } else {
          this.updateSortNods.push({catId: siblings[i].data.catId, sort: i})
        }
      }
    },
    batchSave () {
      // 当前拖拽节点的最新层级发送请求给后端更新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/update/sort`),
        method: 'post',
        data: this.$http.adornData(this.updateSortNods, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '菜单顺序修改成功'
        })
        // 刷新出新的菜单
        this.getMenus()
        this.openKeys = [this.pCid]
        this.emptyCategoryInfo()
      })
    },
    // 批量删除
    batchDelete () {
      let catIds = []
      const checkMenus = this.$refs.treeMenus.getCheckedNodes(false, false)
      for (let i = 0; i<checkMenus.length; i++){
        catIds.push(checkMenus[i].catId)
      }
      this.$confirm(`此操作将批量删除【所选数据】是否继续?', '提示`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$message({
          type: 'success',
          message: `批量删除成功`
        })
        this.$http({
          url: this.$http.adornUrl(`/product/category/delete`),
          method: 'post',
          data: this.$http.adornData(catIds, false)
        }).then(({data}) => {
          this.dialogVisible = false
          // 刷新出新的菜单
          this.getMenus()
          this.emptyCategoryInfo()
        })
      }).catch(() => {
        this.emptyCategoryInfo()
        this.$message({
          type: 'info',
          message: `已取消批量删除`
        })
      })
    },
    updateChildNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let cNode = node.childNodes[i].data
          this.updateSortNods.push({catId: cNode.catId, catLevel: node.childNodes[i].level})
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    }
  },
  created () {
    this.getMenus()
  }
}
</script>
<style scoped lang="scss">

</style>
