<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" node-key="catId" show-checkbox :default-expanded-keys="expandedKeys"
      @node-expand="onNodeExpand" @node-collapse="onNodeCollapse" :auto-expand-parent=false draggable
      :allow-drop="allowDrop" @node-drop="onNodeDrop">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level < 3" type="text" size="mini" @click.stop="() => append(data)">
            Append
          </el-button>
          <el-button v-if="node.isLeaf" type="text" size="mini" @click.stop="() => remove(node, data)">
            Delete
          </el-button>
          <el-button type="text" size="mini" @click.stop="() => edit(data)">
            Edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="添加分类" :visible.sync="dialogue.visible" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogue.visible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      expandedKeys: [],
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: ''
      },

      dialogue: {
        type: 'add',    // add,edit
        title: '',
        visible: false,
      }
    }
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({ data }) => {
        console.log("Get menu data successfully.", data.data)
        this.menus = data.data
      })
    },

    append(data) {
      console.log("append", data);
      this.dialogue.type = 'add'
      this.dialogue.name = '添加分类'
      this.category.name = "";

      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.icon = ''
      this.category.productUnit = ''
      // this.category.showStatus = 1;
      // this.category.sort = 1;
      this.dialogue.visible = true;
    },
    edit(data) {
      console.log("edit:", data)
      this.dialogue.type = 'edit'
      this.dialogue.name = '编辑分类'

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({ data }) => {
        console.log('获取的分类数据:', data)
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        // this.category.catLevel = data.catLevel;
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
        this.dialogue.visible = true;
      })
    },

    submitData() {
      console.log('submit data')
      if (this.dialogue.type === 'add') {
        this.addCategory()
      } else if (this.dialogue.type === 'edit') {
        this.editCategory()
      }
    },

    editCategory() {
      const { catId, name, icon, productUnit } = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data: retData }) => {
        this.$message({
          type: retData.code === 0 ? "success" : "error",
          message: retData.code === 0 ?
            `修改分类【${this.category.name}】成功` : `修改分类【${this.category.name}】失败`
        });
        this.dialogue.visible = false;
        this.getMenus();
      })
    },
    addCategory() {
      console.log('add category')
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data: retData }) => {
        this.$message({
          type: retData.code === 0 ? "success" : "error",
          message: retData.code === 0 ?
            `添加分类【${this.category.name}】成功` : `添加分类【${this.category.name}】失败`
        });
        this.dialogue.visible = false;
        this.getMenus();
      })
    },

    remove(node, data) {
      console.log("remove", node, data)

      this.$confirm(`删除分类【${data.name}】吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData([data.catId], false)
        }).then(({ data: retData }) => {
          this.$message({
            type: retData.code === 0 ? "success" : "error",
            message: retData.code === 0 ?
              `删除分类【${data.name}】成功` : `删除分类【${data.name}】失败`
          });
          this.getMenus();
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },

    onNodeExpand(data, _node, _component) {
      this.expandedKeys.push(data.catId);
    },

    onNodeCollapse(data, _node, _component) {
      let targetIndex = this.expandedKeys.indexOf(data.catId);
      // console.log(targetIndex);
      if (targetIndex !== -1) {
        this.expandedKeys.splice(this.expandedKeys.indexOf(data.catId), 1);
      }
    },

    pushUpdatedChildLevels(updatedNodes, rootNode) {
      for (let i = 0; i < rootNode.childNodes.length; i++) {
        let childNode = rootNode.childNodes[i]
        updatedNodes.push({ catId: childNode.data.catId, catLevel: childNode.level })
        this.pushUpdatedChildLevels(updatedNodes, childNode)
      }
    },
    onNodeDrop(draggingNode, dropNode, dropType, ev) {
      console.log('on node drop:', draggingNode, dropNode, dropType, ev)

      const updatedNodes = []

      //得到parentCid
      let parentCid = 0
      let siblings = null
      if (dropType === 'before' || dropType === 'after') {
        parentCid = dropNode.data.parentCid
        siblings = dropNode.parent.childNodes
      } else if (dropType === 'inner') {
        parentCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }

      //修改sort、拖拽结点的parentCid及其子结点的catLevel
      for (let i = 0; i < siblings.length; i++) {
        let sibling = siblings[i]
        if (sibling.data.catId === draggingNode.data.catId) {
          // console.log('siblings[i].data.catId === draggingNode.data.catId')
          const level = sibling.level
          updatedNodes.push({ catId: sibling.data.catId, sort: i, parentCid: parentCid, catLevel: level})
          if (draggingNode.level !== level) {
            this.pushUpdatedChildLevels(updatedNodes, sibling)
          }
        } else {
          updatedNodes.push({ catId: sibling.data.catId, sort: i })
        }
      }

      console.log('updatedNodes:', updatedNodes)
      this.$http(
        {
          url: this.$http.adornUrl('/product/category/update/batch'),
          method: 'post',
          data: this.$http.adornData(updatedNodes, false)
        }
      ).then(({ data: retData }) => {
        this.$message({
          type: retData.code === 0 ? "success" : "error",
          message: retData.code === 0 ?
            `修改分类排序成功` : `修改分类排序失败`
        });
        this.getMenus();
      })
    },
    allowDrop(draggingNode, dropNode, type) {
      // console.log('allow drop:', draggingNode, dropNode, type)
      const draggingNodeLevel = this.countNodeLevel(draggingNode)
      let resultLevel = 0
      if (type === 'inner') {
        resultLevel = draggingNodeLevel + dropNode.level
      } else {
        resultLevel = draggingNodeLevel + dropNode.parent.level
      }
      return resultLevel <= 3
    },
    countNodeLevel(node) {
      let childMaxLevel = 0
      for (let i = 0; i < node.childNodes.length; i++) {
        childMaxLevel = Math.max(childMaxLevel, this.countNodeLevel(node.childNodes[i]))
      }
      return childMaxLevel + 1
    }
  },
  created() {
    this.getMenus();
  }
}
</script>